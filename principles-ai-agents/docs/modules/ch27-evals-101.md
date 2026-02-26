# Chapter 27 — Evals 101

## Why Evals Matter

!!! quote "The Testing Problem"
    Traditional software has unit tests — deterministic inputs produce deterministic outputs. AI agents are **non-deterministic**. How do you test something that gives different answers each time?

## Eval Basics

An eval is a **systematic test** of an AI system's output quality:

1. Define a set of **test cases** (input + expected characteristics)
2. Run the agent on each case
3. **Score** each output (manually, heuristically, or with another LLM)
4. Track scores over time

## Types of Scoring

| Method | How It Works | Best For |
|--------|-------------|----------|
| **Exact match** | Output == expected | Classification, structured output |
| **Contains/regex** | Output includes key terms | Simple fact checking |
| **Heuristic** | Custom scoring function | Domain-specific quality |
| **LLM-as-judge** | Another model rates the output | Open-ended generation |

!!! warning "LLM-as-Judge Pitfalls"
    Models tend to prefer verbose responses and their own outputs. Calibrate your judge model carefully.

## When to Run Evals

- **Development** — after every prompt change
- **CI/CD** — automated regression testing
- **Production** — sample monitoring

??? question "Discussion: Eval Design"
    You're building an eval suite for a medical Q&A agent. What test cases do you include? How do you handle edge cases where the "right" answer is debatable?
