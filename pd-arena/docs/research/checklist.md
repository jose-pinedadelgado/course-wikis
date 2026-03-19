# Phase Checklist

Current status of PD Arena against each phase's requirements.

**Last updated:** 2026-03-19

## Phase 1: Baseline Social PD *(12/12 complete)* ✅

| # | Criterion | Status | Notes |
|---|-----------|:---:|-------|
| 1 | Real LLM provider integration | ✅ | CrewAI wired via `RealPDAgent`. Factory routes based on `use_mock` flag |
| 2 | LLM agents with proper prompts | ✅ | `prompts.py` has system + round templates with payoff matrix, history, scores |
| 3 | 5 Persona prompts | ✅ | Seeded as DB agents + YAML configs in `configs/agents/` |
| 4 | LLM vs canonical policy agents | ✅ | 7 policy types. Experiment setup supports LLM vs Policy matchups |
| 5 | Geometric horizon | ✅ | Engine supports fixed + geometric with stop_prob |
| 6 | Replicate Fontana's findings | ✅ | Random(α) agents ready. 3 framings available (named/neutral/situated) |
| 7 | Meta-prompting validation | ✅ | `MetaPromptValidator` with 3 framing-specific questions, scored on 3 criteria |
| 8 | Statistical significance | ✅ | `aggregate_condition_metrics` returns mean/std/ci_low/ci_high (95% CI via scipy) |
| 9 | Baseline metrics | ✅ | Cooperation rate, mutual coop/defect, retaliation, forgiveness |
| 10 | Parseable output | ✅ | DB-backed, JSON chart endpoint, round-by-round drill-down |
| 11 | Retry logic | ✅ | `retry_decision()` re-prompts up to 2x with framing-specific hints. Parse errors flagged. |
| 12 | History window | ✅ | Configurable window (default 10), supports none/window/full/summary modes |

---

## Phase 2: Capability Asymmetry *(9/9 complete)* ✅

| # | Criterion | Status | Notes |
|---|-----------|:---:|-------|
| 1 | Chat phase in game engine | ✅ | `run_game_with_chat()` — alternating first-speaker, CHAT: + ACTION: format |
| 2 | Chat ON vs OFF toggle | ✅ | `chat_enabled` field on Experiment model |
| 3 | Protocol structured vs unstructured | ✅ | 3 levels: none, mcp_basic (schema), mcp_filtered (schema + content blocklist) |
| 4 | Identity persistence | ✅ | fresh vs persistent modes. Cross-game summary stored on ExperimentCondition |
| 5 | Memory regimes | ✅ | 4 modes: none, window (configurable N), full, summary |
| 6 | Context pressure | ✅ | Covered by memory_mode + memory_window variables |
| 7 | Phase 2 metrics | ✅ | deception_success_rate, chat_consistency, protocol_violation_count, exploitation_window, trust_recovery_time |
| 8 | 2×2 design (chat × protocol) | ✅ | chat_enabled × protocol_mode configurable per experiment |
| 9 | Statistical analysis | ✅ | All Phase 2 metrics aggregated with CIs across replicates |

---

## Phase 3: Tools + Ill Intent *(4/4 complete)* ✅

| # | Criterion | Status | Notes |
|---|-----------|:---:|-------|
| 1 | Sandboxed mock tools | ✅ | 3 tools: read_opponent_strategy, send_side_channel, delegate_decision |
| 2 | Violation detection | ✅ | Taxonomy: unauthorized_access, side_channel, work_offloading, prompt_injection |
| 3 | Goal framing variable | ✅ | 3 goals: cooperative, self_maximizing, adversarial — per scenario framing |
| 4 | Tool access variable | ✅ | `tools_enabled` toggle on Experiment |

---

## Phase 4: MCP vs Non-MCP *(built into Phase 2)* ✅

| # | Criterion | Status | Notes |
|---|-----------|:---:|-------|
| 1 | Three protocol levels | ✅ | none / mcp_basic / mcp_filtered |
| 2 | Violation reduction data | ✅ | Protocol violation metrics + manipulation pattern detection |
| 3 | Residual vulnerability analysis | 🟡 | Data will be available from experiments; no automated analyzer yet |

---

## Overall: 321 Tests, All Phases Built

| Phase | Tests | Status |
|-------|:---:|:---:|
| Phase 1 | 120 | ✅ |
| Phase 2 | 269 | ✅ |
| Phase 3 | 321 | ✅ |

**Phase 1 baseline results collected** with gpt-4.1-mini (3 personas × 6 policy agents).

**Next step:** Phase 2 experiments — run chat × protocol conditions with adversarial persona matchups.
