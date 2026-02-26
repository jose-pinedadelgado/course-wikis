# File Converters

## Supported Formats

| Format | Tier | Library | Notes |
|---|---|---|---|
| `.md` | 1 | Built-in | Copy + clean encoding |
| `.sql` | 1 | Built-in | Wrap in code fences, detect DDL vs. DML |
| `.docx` | 1+2 | `python-docx` | Extract text, tables, images; Tier 2 for formatting |
| `.pptx` | 1+2 | `python-pptx` | Extract slide titles + notes; Tier 2 for structure |
| `.pdf` | 2 | `pymupdf` | Extract text; Tier 2 for markdown formatting |
| `.drawio` | 1 | `xml.etree` | Parse XML for entity/relationship labels |
| `.csv` | 1 | `csv` | Convert to markdown tables |

## SQL Converter (Tier 1)

SQL files get special treatment for database courses:

```python
def convert_sql(path: Path) -> MarkdownPage:
    content = path.read_text()
    
    if "CREATE TABLE" in content.upper():
        # DDL → schema reference page with markdown tables
        tables = parse_create_tables(content)
        return generate_schema_page(tables, original_sql=content)
    
    elif re.search(r'--\s*(Problem|Q\d|Exercise)\s', content):
        # Exercise file → split into problems with collapsible solutions
        problems = split_by_comments(content)
        return generate_exercise_page(problems)
    
    else:
        # Generic SQL → code-fenced markdown
        return wrap_in_code_fence(content, language="sql")
```

### Schema Page Output

```markdown
## Customer Table

| Column | Type | Constraints |
|---|---|---|
| customer_id | INT | PRIMARY KEY, AUTO_INCREMENT |
| first_name | VARCHAR(50) | NOT NULL |
| email | VARCHAR(100) | UNIQUE |

??? example "Original DDL"
    ```sql
    CREATE TABLE Customer (
        customer_id INT PRIMARY KEY AUTO_INCREMENT,
        ...
    );
    ```
```

### Exercise Page Output

```markdown
## Problem 1: Find all orders over $100

??? success "Solution"
    ```sql
    SELECT * FROM Orders WHERE total > 100;
    ```
```

## PPTX Converter (Tier 1+2)

The hardest format — content is visual.

**Tier 1 extracts:**

- Slide titles (from title placeholders)
- Slide notes (often contain the "script")
- Text content from body placeholders
- Embedded images

**Tier 2 organizes:**

- Group slides into logical sections
- Classify slides (concept vs. example vs. transition)
- Convert bullet-heavy slides into prose

**Tier 3 enriches:**

- Write connecting narrative between topics
- Add context delivered verbally but missing from slides

## PDF Converter (Tier 2)

Used successfully for the Principles of AI Agents wiki (149-page book → 34 chapters):

1. **Extract** full text via `pymupdf` (fitz)
2. **Split** by chapter headings (regex on "Chapter N:" patterns)
3. **Format** each chapter as markdown with key concepts tables and admonition boxes
4. **Group** chapters by book parts for navigation

## Encoding Fixer (Tier 2)

Common Notion export artifacts:

| Artifact | Fix |
|---|---|
| `â€™` | `'` |
| `â€œ` | `"` |
| `â€"` | `—` |
| `Ã©` | `é` |

Handled by regex, not AI — pattern matching is sufficient.
