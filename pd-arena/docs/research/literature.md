# Literature Review

## Key Papers

### Fontana et al. (2024/2025) — "Nicer Than Humans"

- **Scenario:** 100-round IPD, LLM agents vs. random opponents that cooperate with probability α (0.0 to 1.0)
- **Communication:** None
- **Agent introduction:** System prompt with PD rules, payoff matrix, 10-round history window
- **Motivation:** "Maximize your score"
- **Key finding:** LLMs cooperate ~79% vs. humans ~37%. Llama3 was exploitative.
- **Validation:** Meta-prompting — asked LLMs to explain the game before playing

### Akata et al. (2025) — "Playing Repeated Games" (Nature)

- **Scenario:** Only 10 rounds, 144 different 2×2 games. LLM-vs-LLM, LLM-vs-policy, LLM-vs-humans (195 people)
- **Communication:** None. Used neutral labels (J/F, not Cooperate/Defect) to avoid bias
- **Agent introduction:** Cover stories, abstract framing. Temperature 0.0
- **Motivation:** Score-maximizing. Added Social Chain-of-Thought (SCoT) — predict opponent's action before deciding
- **Key finding:** GPT-4 permanently defects after a single betrayal. "The other player sometimes makes mistakes" framing restored cooperation. SCoT improved coordination.
- **Limitation:** 10 rounds too short for strategies to evolve

### Pal et al. (2026) — "Strategies of Cooperation/Defection in Five LLMs"

- **Scenario:** 50 API calls per scenario (hypothetical), then 10-round verification games
- **Communication:** None. Neutral labels (L/R)
- **Agent introduction:** 9 framings (neutral, competitive, cooperative, etc.)
- **Key finding:** Each model has a distinct "strategic personality" — Claude = Forgiver, GPT-4o = GRIM, Llama = Generous TFT. Claude wins 12 of 15 tournament disciplines.

### Guo (2023) & Phelps (2023) — Persona Studies

- Tested personality prompts ("fair," "selfish," "altruistic," "competitive")
- Personas translate to behavior, but agents struggle with conditional reciprocity
- Cooperation only stable when both agents are cooperative

### Piatti et al. (2024) — "Cooperate or Collapse" (NeurIPS)

- **Scenario:** 5 LLM agents share a renewable resource pool (commons dilemma)
- **Communication:** Yes — free-form natural language between agents
- **Key finding:** Communication reduced overuse by 21%. "Universalization" reasoning improved sustainability.
- **Only the strongest models (GPT-4o) could sustain cooperation**

### Poje et al. (2024) — Private Deliberation & Deception

- **Scenario:** Repeated games with "private agent" (hidden chain-of-thought)
- **Communication:** Public messages, but private reasoning could differ
- **Key finding:** Private agents with deception capability achieved higher long-term payoffs. Deployed deception only when strategically optimal.

## Summary: Communication Effects

| Study | Communication? | Effect |
|-------|:---:|--------|
| Fontana, Akata, Pal | ❌ | Baseline — cooperation driven by prompt + model personality |
| Piatti (resource game) | ✅ Free-form | 21% reduction in overuse |
| Poje (repeated games) | ✅ With hidden reasoning | Deception → higher payoffs when optimal |
| **PD Arena (ours)** | ✅ Controlled variable | **First to test in standard IPD with LLMs** |

## Key Gaps We Address

1. **No one tested communication as a controlled variable** in standard IPD with LLMs
2. **No one tested protocol safeguards** (MCP-like) in game-theoretic settings
3. **No one used geometric/unknown horizon** (changes optimal strategy fundamentally)
4. **No one mapped personas to classical strategies** using canonical baselines
5. **No one published in IS venues** — all prior work is CS/AI/psychology
6. **No one built a reproducible web-based benchmark** with UI for experiment design
