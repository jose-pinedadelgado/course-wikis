# Roadmap

## Current State: Spec Complete, Pre-Build

The full product spec is written (`docs/SPEC.md`, ~20KB). Five wikis have been built manually, validating the approach. The CLI doesn't exist yet.

## Milestones

### v0.1 — Canvas Importer (MVP)

- [ ] Canvas ZIP extractor + `course-data.js` parser
- [ ] Folder structure generator from module tree
- [ ] `mkdocs.yml` generator
- [ ] `.sql` converter (DDL → schema pages, exercises → collapsible)
- [ ] `.docx` text extractor
- [ ] `.md` pass-through with encoding fix
- [ ] `chalq import canvas` command
- [ ] `chalq serve` and `chalq build` (wrapper around mkdocs)

### v0.2 — Notion Importer + AI

- [ ] Notion ZIP extractor (nested zips, UUID stripping)
- [ ] Page hierarchy parser
- [ ] Tier 2: Haiku-based cleanup
- [ ] Tier 3: Sonnet-based enrichment
- [ ] `chalq import notion` and `chalq enrich` commands
- [ ] Cost tracking and budget caps

### v0.3 — PPTX + Publishing

- [ ] `.pptx` slide extractor and converter
- [ ] `chalq publish` command
- [ ] `chalq.yml` course config with module overrides
- [ ] `chalq list` and `chalq status` commands

### v0.4 — Polish

- [ ] `.pdf` text extractor
- [ ] `.drawio` entity/relationship parser
- [ ] `.csv` → markdown table converter
- [ ] Raw file import
- [ ] Content validation (broken links, missing images)

### v1.0 — Product

- [ ] Canvas API integration (live sync)
- [ ] Multi-institution config
- [ ] GitHub Pages deployment automation
- [ ] Web UI for non-technical instructors

## Success Metrics

| Metric | Manual (today) | v0.1 Target | v1.0 Target |
|---|---|---|---|
| Time per course | 2-4 hours | 5 minutes | 30 seconds |
| Cost per course | $5-10 | $0.16 | $0.05 |
| Setup effort | High | `chalq import` | Canvas webhook |
| Quality | High (human) | Good (90%) | High (AI enrichment) |
