# Cost Model

## Per-Course Cost Breakdown

| Tier | Work Share | Cost | Speed | Model |
|---|---|---|---|---|
| Tier 1 (Deterministic) | ~60% | $0.00 | Seconds | None |
| Tier 2 (Cheap AI) | ~25% | ~$0.01 | Seconds | Haiku / regex |
| Tier 3 (Smart AI) | ~15% | ~$0.15 | 30-60s | Sonnet |
| **Total** | **100%** | **~$0.16** | **< 2 min** | |

## Cost Comparison

| Approach | Cost/Course | Speed | Quality |
|---|---|---|---|
| Manual (human + AI pair) | $5-10 | 2-4 hours | High |
| Send everything to Opus | $2-4 | 5-10 min | High |
| Send everything to Sonnet | $1-2 | 3-5 min | Good |
| **Chalq three-tier** | **$0.16** | **< 2 min** | **Good (90%)** |
| Tier 1 only (no AI) | $0.00 | Seconds | Scaffolding only |

## Cost Controls

```bash
# No AI at all
chalq import canvas export.zip --course my-course --no-ai

# Cap at Tier 2
chalq enrich my-course --tier 2

# Budget cap
chalq enrich my-course --max-cost 0.50

# Dry run (show what would be called)
chalq enrich my-course --dry-run
```

## At Scale

| Courses | Tier 1 | Tier 2 | Tier 3 | Total |
|---|---|---|---|---|
| 1 course | $0 | $0.01 | $0.15 | **$0.16** |
| 10 courses | $0 | $0.10 | $1.50 | **$1.60** |
| 100 courses | $0 | $1.00 | $15.00 | **$16.00** |
| 1,000 courses | $0 | $10.00 | $150.00 | **$160.00** |

Even at 1,000 courses, the total cost is under $200 — comparable to a single month of ChatGPT Plus.
