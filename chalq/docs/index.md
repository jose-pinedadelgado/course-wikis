# 🩵 Chalq

> *"Write it on the Chalk board, publish it to the world."*

Chalq is a **CLI tool** that converts course materials from various sources (Canvas LMS, Notion, raw files) into beautiful, searchable MkDocs Material wikis — with optional AI enrichment for content that needs judgment.

## The Problem

University professors maintain course content across multiple platforms:

- **Canvas LMS** — modules, assignments, files
- **Notion** — lecture notes, study guides
- **Google Docs** — syllabi, activities
- **PowerPoint** — slide decks
- **SQL files** — database exercises
- **draw.io** — diagrams

Students access this through a clunky LMS interface. This content *could* be published as beautiful, searchable, public-facing wikis — but the manual conversion is time-consuming.

## The Solution

```bash
# Convert a Canvas export to a wiki in one command
chalq import canvas IS480-export.zip --course adv-db

# Serve it locally
chalq serve adv-db --port 8000

# Enrich with AI (optional)
chalq enrich adv-db --model sonnet

# Publish to course-wikis repo
chalq publish adv-db --repo ../course-wikis
```

**Cost:** ~$0.16 per course (vs. $2–4 sending everything to a large LLM)
**Speed:** Under 2 minutes per course

## Design Principles

1. **Deterministic first** — Parse, structure, and convert without AI when possible
2. **AI for judgment only** — Use LLMs only where human-like decisions are needed
3. **Incremental** — Process one file, one module, or one course at a time
4. **Idempotent** — Running Chalq twice produces the same result
5. **Source-agnostic** — Support Canvas, Notion, raw files, and future LMS exports
6. **Framework-agnostic** — Output works with MkDocs Material today, Zensical tomorrow

## Wikis Built So Far

Chalq's approach has been validated by building 5 wikis (manually, pre-CLI):

| Wiki | Pages | Source | Status |
|---|---|---|---|
| [IS480 Advanced Database Mgmt](../advanced-database-management/) | 13 | Canvas export | ✅ Complete |
| [IS671 Responsible AI](../responsible-ai/) | 8 | Notion export | ✅ Partial |
| [Principles of Building AI Agents](../principles-ai-agents/) | 34 | PDF extraction | ✅ Complete |
| [Bamboo Money](../bamboo-money/) | 30 | Project docs | ✅ Complete |
| [IS480 Zensical Build](../advanced-database-management-zensical/) | 13 | Same as MkDocs | ✅ Complete |

## Tech Stack

| Component | Technology |
|---|---|
| Language | Python 3.12 |
| CLI Framework | Click or Typer |
| Template Engine | Jinja2 |
| Document Parsers | python-docx, python-pptx, pymupdf |
| AI Client | Anthropic SDK |
| Wiki Engine | MkDocs Material (→ Zensical) |
| Package Manager | UV |

---

*Jose Pineda — Assistant Professor, California State University, Long Beach*
