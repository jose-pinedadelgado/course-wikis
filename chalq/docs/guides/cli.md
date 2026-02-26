# CLI Reference (Planned)

## Commands

### chalq import canvas

Import from a Canvas LMS course export.

```bash
chalq import canvas <zip-path> --course <slug>
```

Extracts the ZIP, parses `course-data.js`, converts all files, and builds the wiki structure.

### chalq import notion

Import from a Notion export.

```bash
chalq import notion <zip-path> --course <slug>
```

Handles nested ZIPs, UUID stripping, encoding fixes, and page hierarchy.

### chalq import files

Import raw files without an LMS export.

```bash
chalq import files <glob-pattern> --course <slug> --section <name>
```

### chalq enrich

AI-enrich stub pages and write overviews.

```bash
chalq enrich <course-slug> [--model sonnet] [--dry-run] [--max-cost 1.00]
```

| Flag | Default | Description |
|---|---|---|
| `--model` | `sonnet` | AI model for enrichment |
| `--dry-run` | off | Show what would be done without doing it |
| `--max-cost` | unlimited | Budget cap in dollars |
| `--tier` | 3 | Maximum processing tier (1, 2, or 3) |

### chalq build

Build the static wiki site.

```bash
chalq build <course-slug>
```

Wrapper around `mkdocs build`.

### chalq serve

Serve locally for preview.

```bash
chalq serve <course-slug> [--port 8000]
```

### chalq publish

Copy wiki to the course-wikis repository.

```bash
chalq publish <course-slug> --repo <path-to-course-wikis>
```

Copies, commits, and pushes.

### chalq list

List all known courses.

```bash
chalq list
```

### chalq status

Show course status.

```bash
chalq status <course-slug>
```

Shows page count, stub count, last build time, source info.
