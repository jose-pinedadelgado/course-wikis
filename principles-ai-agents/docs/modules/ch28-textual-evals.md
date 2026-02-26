# Chapter 28 — Textual Evals

## Evaluating Text Quality

For text generation tasks, you need metrics beyond exact match:

### Common Textual Metrics

| Metric | What It Measures |
|--------|-----------------|
| **Relevance** | Does the answer address the question? |
| **Faithfulness** | Is the answer grounded in provided context? (No hallucination) |
| **Completeness** | Does it cover all important points? |
| **Coherence** | Is it well-organized and logical? |
| **Conciseness** | Is it appropriately brief? |
| **Toxicity** | Does it contain harmful content? |

## LLM-as-Judge Setup

Use a structured prompt that asks the judge model to:

1. Evaluate the response on specific criteria
2. Provide a score (1-5 or pass/fail)
3. Explain the reasoning

!!! tip "Use a Different Model as Judge"
    Don't use the same model to both generate and judge. Cross-model evaluation reduces bias.

## Eval Frameworks

| Framework | Notes |
|-----------|-------|
| **Braintrust** | Production-grade, integrates with CI |
| **Promptfoo** | Open-source, CLI-based |
| **Ragas** | Focused on RAG evaluation |

??? question "Discussion: Subjectivity in Evals"
    Two humans might disagree on whether a response is "complete" or "concise." How do you handle this inherent subjectivity in automated evals?
