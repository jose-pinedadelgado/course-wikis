# 1. LLM Post-Training

## What Is Post-Training?

Pre-training gives an LLM broad linguistic ability and general knowledge by training on massive text corpora. **Post-training** is everything that happens *after* — the phase that transforms a general model into one that follows instructions, aligns with human preferences, and meets production reliability requirements.

```
Pre-Training          Post-Training              Production
(Broad knowledge) →   (Aligned behavior) →       (Deployed model)

Trillions of tokens   SFT → RLHF/DPO/GRPO →     Serving via vLLM
on web text           Domain adaptation            A/B tested
```

## Why It Matters

- **Pre-training** costs $10M–$100M+ and takes months → you don't do this yourself
- **Post-training** costs $1K–$100K and takes days → this is where you add value
- Every production LLM (ChatGPT, Claude, Gemini) goes through extensive post-training
- Netflix's entire ML research team is focused on post-training open-weight models for their specific use cases

## The Post-Training Landscape (2025-2026)

| Method | Type | Signal | Key Idea |
|--------|------|--------|----------|
| [**SFT**](sft.md) | Supervised | Dense (per-token) | Train on curated (prompt, response) pairs |
| [**RLHF**](rlhf.md) | RL-based | Sparse (reward) | Train reward model on preferences, then optimize policy with PPO |
| [**DPO**](dpo.md) | Offline | Preference pairs | Skip the reward model — optimize directly on preferred vs rejected |
| [**GRPO**](grpo.md) | On-policy RL | Sparse (reward) | Group-relative scoring, no critic network needed |
| [**LoRA**](lora.md) | Parameter-efficient | Any of above | Train only low-rank adapter weights, not full model |
| [**Distillation**](distillation.md) | Supervised | Teacher logits | Train small model to mimic large model's outputs |

## The Evolution

```
2022: SFT → RLHF (InstructGPT)
      "Fine-tune, then align with human feedback"

2023: SFT → DPO (Zephyr, Neural Chat)
      "Skip the reward model, optimize preferences directly"

2025: SFT → GRPO (DeepSeek-R1)
      "On-policy RL without a critic — group-relative scoring"

2026: SFT → GRPO + Domain Adaptation (Netflix, Google)
      "Post-training as a product differentiator"
```

## Netflix's Approach

Netflix built an [internal post-training framework](netflix-framework.md) that supports SFT, DPO, RL, and distillation. Key insight from their blog:

> *"SFT became table stakes rather than the finish line. Staying close to the frontier required infrastructure that could move from 'offline training loop' to 'multi-stage, on-policy orchestration.'"*

Their framework handles:

- Standard chat/instruction fine-tuning
- Custom transformers trained on **non-NLP sequences** (member interaction events)
- RL loops with business-defined reward metrics
- Models with expanded vocabularies (semantic IDs for catalog items)

## What To Learn (Priority Order)

1. **SFT** — the foundation; understand loss masking, chat templates, data curation
2. **LoRA** — parameter-efficient fine-tuning; you'll use this constantly
3. **DPO** — the simplest preference optimization method
4. **GRPO** — the current frontier (DeepSeek-R1)
5. **RLHF/PPO** — the classic approach; important historically and conceptually
6. **Distillation** — increasingly important for deployment (large → small model transfer)

---

*Next: [Supervised Fine-Tuning (SFT)](sft.md)*
