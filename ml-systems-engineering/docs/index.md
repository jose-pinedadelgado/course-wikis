# ML Systems Engineering — Field Guide

**A self-study curriculum for staff-level ML engineering roles ($200K–$750K+) at Netflix, Google, Meta, Anthropic, and beyond.**

---

## Why This Exists

The gap between "I understand ML concepts" and "I can build production ML systems at scale" is where the highest-paying roles in tech live. This wiki bridges that gap — covering the full stack from distributed training to serving, from data pipelines to recommendation systems.

Every section is built around **what top companies actually ask for** in their job postings and what their engineering blogs reveal about how they work.

## The Target

!!! info "Netflix Research Scientist 5/6 — $466,000–$750,000/year"

    **Required:** Python, TensorFlow, PyTorch

    **Nice to have:** Java, Scala, Spark, Hive, Jax, Flink, Hadoop

    Deep expertise in LLM development, post-training (fine-tuning, distillation), distributed training, reinforcement learning, personalization, recommender systems.

This isn't just Netflix. The same skillset maps to:

- **Google** — ML Infrastructure, Search, Ads
- **Meta** — AI Research, Recommendation Systems
- **Anthropic / OpenAI** — Post-training, RLHF, Safety
- **Spotify** — Personalization, ML Platform
- **Apple** — On-device ML, Foundation Models

## Curriculum Map

```
┌─────────────────────────────────────────────────────────────┐
│                    ML SYSTEMS ENGINEERING                     │
├──────────────┬──────────────┬──────────────┬────────────────┤
│  1. POST-    │ 2. DISTRIB-  │ 3. INFERENCE │ 4. DATA AT     │
│  TRAINING    │ UTED         │ & SERVING    │ SCALE          │
│              │ TRAINING     │              │                │
│ • SFT       │ • FSDP       │ • vLLM       │ • Spark SQL    │
│ • RLHF      │ • Tensor     │ • KV Cache   │ • Hive         │
│ • DPO       │   Parallel   │ • Quantize   │ • Feature      │
│ • GRPO      │ • Ray        │ • Batching   │   Stores       │
│ • LoRA      │ • Checkpoint │              │ • Training     │
│ • Distill   │ • MFU        │              │   Data         │
├──────────────┼──────────────┼──────────────┼────────────────┤
│ 5. RECSYS   │ 6. TRANS-    │ 7. RL FOR    │ 8. MLOPS       │
│              │ FORMERS      │ LLMs         │                │
│ • Collab    │ • Attention  │ • PPO        │ • Experiment   │
│   Filtering │ • Position   │ • GRPO       │   Tracking     │
│ • Two-Tower │   Encoding   │ • Reward     │ • CI/CD        │
│ • Semantic  │ • MoE        │   Modeling   │ • A/B Testing  │
│   IDs       │ • Modern     │ • On/Off     │                │
│ • Netflix   │   Archs      │   Policy     │                │
│   Personal. │              │              │                │
└──────────────┴──────────────┴──────────────┴────────────────┘
```

## How to Use This Wiki

1. **Read section overviews first** — each section starts with "what this is and why it matters"
2. **Depth pages** — detailed explanations with code examples, math where needed
3. **Case studies** — real implementations from Netflix, Google, Meta tech blogs
4. **Reading list** — papers, blogs, and courses ranked by priority
5. **Target roles** — specific job postings mapped to wiki sections

## Progress Tracker

| Section | Status | Pages |
|---------|--------|-------|
| 1. LLM Post-Training | 🟢 Started | 7 |
| 2. Distributed Training | 🟡 Scaffolded | 5 |
| 3. ML Inference & Serving | 🟡 Scaffolded | 4 |
| 4. Data at Scale | 🟡 Scaffolded | 4 |
| 5. Recommender Systems | 🟡 Scaffolded | 4 |
| 6. Transformers Deep Dive | 🟡 Scaffolded | 4 |
| 7. RL for LLMs | 🟡 Scaffolded | 4 |
| 8. MLOps & Infrastructure | 🟡 Scaffolded | 3 |

---

*Built by Jose Pineda — preparing for what's next.*
