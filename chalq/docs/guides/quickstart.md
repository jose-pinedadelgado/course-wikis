# Quick Start

!!! warning "Pre-Release"
    Chalq is currently in the **specification phase**. The CLI doesn't exist yet — the wikis produced so far were built manually using the same approach Chalq will automate. See the [Roadmap](../product/roadmap.md) for build timeline.

## What Chalq Will Do

```bash
# Install
uv add chalq

# Import a Canvas course export
chalq import canvas IS480-export.zip --course adv-db

# Preview locally
chalq serve adv-db --port 8000

# Optional: AI enrichment
chalq enrich adv-db --model sonnet --max-cost 0.50

# Publish to your wiki repo
chalq publish adv-db --repo ../course-wikis
```

## What You Can Do Now

The manual workflow that Chalq will automate:

1. **Export** your Canvas course (Settings → Export Course Content → ZIP)
2. **Extract** the ZIP and find `course-data.js` (contains module structure)
3. **Create** an MkDocs Material project (`mkdocs.yml` + `docs/` folder)
4. **Convert** each file manually:
    - `.sql` → code-fenced markdown
    - `.docx` → cleaned markdown
    - `.pptx` → slide titles + notes
    - `.md` → copy with encoding fixes
5. **Build** with `uv run mkdocs build`
6. **Serve** with `uv run mkdocs serve`

Chalq automates steps 2–5.

## Project Location

- **Spec:** `6_Chalq/docs/SPEC.md`
- **Course wikis (source):** `6_Chalq/<course-name>/`
- **Course wikis (published):** `course-wikis/<course-name>/`
- **GitHub (private):** `jose-pinedadelgado/chalq`
- **GitHub (public):** `jose-pinedadelgado/course-wikis`
