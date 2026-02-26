# Wikis Built

These wikis were built manually using Chalq's approach (before the CLI exists), validating the three-tier model.

## Course Wikis

| Wiki | Pages | Source | Theme | Build Time |
|---|---|---|---|---|
| IS480 Advanced Database Mgmt | 13 | Canvas export | Indigo/Pink | 0.53s |
| IS480 (Zensical build) | 13 | Same source | Indigo/Pink | 0.50s |
| IS671 Responsible AI | 8 | Notion export | Teal/Orange | ~0.4s |
| Principles of Building AI Agents | 34 | PDF (149 pages) | Deep Purple/Amber | 0.42s |

## Project Wikis

| Wiki | Pages | Source | Theme |
|---|---|---|---|
| Bamboo Money | 30 | Project docs + code | Green/Amber |
| Prisoner's Dilemma | 19 | Research docs + code | Deep Orange/Teal |
| Chalq (this wiki) | 14 | Product spec | Indigo/Pink |
| Pineapple | ~10 | README + review | Orange/Blue |
| PacoDogShop | ~8 | Schema + README | Brown/Amber |

## Lessons Learned

### What Tier 1 Can Handle

- SQL → markdown conversion (DDL detection, exercise splitting) ✅
- Module structure from Canvas `course-data.js` ✅
- Page hierarchy from Notion folder structure ✅
- PDF text extraction (pymupdf) ✅
- `mkdocs.yml` generation from page tree ✅

### What Needs Tier 2

- Encoding fixes from Notion exports (`â€™` → `'`)
- Identifying exercise vs. solution boundaries in SQL
- Cleaning `.docx` extraction artifacts

### What Needs Tier 3

- Writing course overview pages
- Enriching stub pages (Notion stubs with just "Owner: José Pineda")
- Cross-referencing between topics
- Writing "why this matters" introductions

### Key Insight

> The manual wiki-building sessions proved that **60-70% of the work is mechanical** — parsing, structuring, converting. AI is only needed for the remaining 30-40% that requires judgment. This validates Chalq's three-tier cost model.
