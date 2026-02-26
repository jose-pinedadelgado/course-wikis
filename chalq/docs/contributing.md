# Contributing

## Repository Structure

Chalq uses a dual-repo model:

| Repo | Visibility | Content |
|---|---|---|
| `jose-pinedadelgado/chalq` | Private | CLI tool + source wikis (may contain student data) |
| `jose-pinedadelgado/course-wikis` | Public | Published wikis (sanitized) |

## Development Setup

```bash
cd 6_Chalq
uv sync
```

## Building a Wiki Locally

Each wiki is an independent MkDocs project:

```bash
cd course-wikis/bamboo-money  # or any wiki
uv sync
uv run mkdocs serve -a 127.0.0.1:8400
```

## Adding a New Wiki

1. Create a folder in `course-wikis/<wiki-name>/`
2. Add `pyproject.toml` with `mkdocs-material` dependency
3. Add `mkdocs.yml` with theme and navigation
4. Add `docs/` with markdown pages
5. Run `uv sync && uv run mkdocs build` to verify
6. Copy to `6_Chalq/<wiki-name>/` (source of truth)
7. Commit and push both repos

---

*Jose Pineda — CSULB*
