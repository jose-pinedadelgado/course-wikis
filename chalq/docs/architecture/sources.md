# Input Sources

## Canvas Export (Priority 1)

Canvas exports are ZIP files containing the full course structure.

### Structure

```
Course-Name.zip
└── Course-Name/
    ├── index.html          # Viewer app (ignore)
    └── viewer/
        ├── course-data.js  # ← Full course structure as JSON
        └── files/
            ├── *.pptx      # Lecture slides
            ├── *.docx      # Activities, labs, syllabi
            ├── *.sql       # Code files
            └── ...
```

### The Goldmine: course-data.js

`course-data.js` contains a `window.COURSE_DATA` object with:

- Module names and ordering
- Item types (Attachment, ExternalUrl, Assignment, SubHeader)
- File paths
- Assignment details (due dates, points, submission types)
- Indent levels (for sub-items)

This structured data means **Tier 1 can build the entire wiki skeleton** from it.

### Parsed Output

```python
@dataclass
class CanvasModule:
    name: str           # "Week 4: Using SQL within PL/SQL"
    items: list[CanvasItem]

@dataclass
class CanvasItem:
    title: str          # "SQL in PLSQL.pptx"
    type: str           # "Attachment" | "ExternalUrl" | "Assignment"
    content: str        # File path or URL
    indent: int         # Nesting level
```

---

## Notion Export (Priority 2)

Notion exports are **nested ZIPs** with a page hierarchy.

### Structure

```
export.zip
└── ExportBlock-*.zip
    └── Content/
        ├── Course Title <uuid>.md
        └── Course Title/
            ├── Chapter 1 <uuid>.md
            ├── Chapter 1/
            │   ├── Topic A <uuid>.md
            │   ├── Topic A/
            │   │   └── image.png
            │   └── Topic B <uuid>.md
            └── Chapter 2 <uuid>.md
```

### Challenges

- **UUIDs in filenames** — must be stripped
- **Encoding issues** — UTF-8 mojibake common (e.g., `â€™` → `'`)
- **Content varies wildly** — from rich (code, formulas) to empty stubs
- **Images** — stored alongside parent page, need path adjustment
- **"Owner: José Pineda"** lines — need removal

---

## Raw Files (Priority 3)

Direct file import for courses without an LMS export:

```bash
chalq import files "lectures/*.pptx" --course my-course --section "Lectures"
```

---

## Future Sources

| Source | Status | Notes |
|---|---|---|
| Canvas API | Planned (v1.0) | Live sync without manual export |
| Google Drive | Planned | Folder structure → wiki structure |
| Blackboard | Future | Similar to Canvas export format |
| Moodle | Future | XML-based export |
