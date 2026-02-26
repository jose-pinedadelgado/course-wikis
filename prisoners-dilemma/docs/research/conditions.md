# Condition Matrix

Detailed experiment sizing, runtime estimates, and cost analysis.

## Grand Total

| Phase | Conditions | Runs (×15 reps) |
|---|---|---|
| Phase 1 (reduced) | 45 | 675 |
| Phase 2 | 54 | 810 |
| Phase 3 | 24 | 360 |
| Phase 4 | 24 | 360 |
| **Total** | **147** | **~2,200** |

## Runtime Estimation

Assuming ~100 rounds/game × ~2–3 sec per API call (both agents) ≈ 4–5 min per game.

| Phase | Runs | Sequential | Parallel (10×) |
|---|---|---|---|
| Phase 1 | 675 | ~56 hours | ~6 hours |
| Phase 2 | 810 | ~68 hours | ~7 hours |
| Phase 3 | 360 | ~30 hours | ~3 hours |
| Phase 4 | 360 | ~30 hours | ~3 hours |
| **Total** | **2,200** | **~184 hours** | **~19 hours** |

!!! info "Bottleneck: Rate Limits"
    At 10 concurrent games (~600K tokens/min), we may hit TPM caps depending on API tier. The bottleneck is rate limits, not computation time.

**Practical plan:**

- Phase 1 on Haiku → fast + cheap, generous rate limits
- Phases 2–4 on Sonnet → slower but where the novelty lives
- Run overnight, batch by phase
- Realistic wall-clock: **2–3 days** with smart parallelism

## Cost Estimation

Assuming ~500 tokens per round × 100 rounds per game × 2 agents = ~100K tokens per run.

2,200 runs × 100K = **~220M tokens total**

At Claude Sonnet pricing (~$3/M input, $15/M output, ~50/50 split):

| Phase | Model | Estimated Cost |
|---|---|---|
| Phase 1 | Haiku | ~$100 |
| Phase 2 | Sonnet | ~$900 |
| Phase 3 | Sonnet | ~$400 |
| Phase 4 | Sonnet | ~$400 |
| **Total** | | **$2,000–$3,000** |

## Cost Reduction Strategies

1. **Use Haiku for Phase 1** — drops cost from ~$750 to ~$100
2. **Shorter games** — 50 rounds instead of 100 (halves token cost)
3. **Fewer replicates** — 10 instead of 15 for low-variance conditions
4. **Progressive design** — only expand conditions that show signal
5. **Mock agents for rig validation** — run 100+ games with zero API cost using personality agents

## Priority Order (Budget-Constrained)

If budget is limited, phases should be prioritized:

| Priority | Phase | Why |
|---|---|---|
| 1 | Phase 1 | Validates the rig, cheap on Haiku |
| 2 | Phase 2: Memory | Most novel — memory + identity is the paper's contribution |
| 3 | Phase 3: Tools | Safety implications — high impact |
| 4 | Phase 2: Context | Interesting but deferrable |
| 5 | Phase 4: MCP | Important for policy but depends on Phase 3 |

## Statistical Design

- **Replicates:** 10–20 runs per condition (for temperature > 0)
- **Analysis:** Offline log analysis on JSONL + Parquet outputs
- All phases produce structured logs for cross-phase comparison
- Key statistical tests: Mann-Whitney U for cooperation rates, chi-square for violation frequencies
