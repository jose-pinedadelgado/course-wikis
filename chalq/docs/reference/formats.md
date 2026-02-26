# Supported Formats

| Format | Extension | Tier | Parser | Notes |
|---|---|---|---|---|
| Markdown | `.md` | 1 | Built-in | Copy + encoding fix |
| SQL | `.sql` | 1 | Built-in | DDL detection, exercise splitting |
| Word | `.docx` | 1+2 | python-docx | Text + tables + images |
| PowerPoint | `.pptx` | 1+2 | python-pptx | Titles, notes, body text |
| PDF | `.pdf` | 2 | pymupdf | Full text extraction |
| draw.io | `.drawio` | 1 | xml.etree | Entity/relationship labels |
| CSV | `.csv` | 1 | csv | → Markdown tables |

## Input Containers

| Source | Format | Parser |
|---|---|---|
| Canvas | ZIP with `course-data.js` | Custom JS parser |
| Notion | Nested ZIPs with markdown | UUID stripper + hierarchy builder |
| Raw files | Glob pattern | Direct file reader |
