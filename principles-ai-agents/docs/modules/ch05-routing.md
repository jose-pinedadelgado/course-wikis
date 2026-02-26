# Chapter 5 — Model Routing & Structured Output

## Model Routing

Model routing lets you **switch between providers** without rewriting code. Use a routing library (like Vercel's AI SDK) to abstract away provider-specific APIs.

!!! tip "Why This Matters"
    Models improve constantly. You want to swap GPT-4o for Claude or Gemini with a one-line change, not a rewrite.

## Structured Output

When LLMs are part of an application pipeline, you often need **JSON responses** instead of free text. Most modern models support structured output via schemas.

### Use Cases

| Input | Structured Output |
|-------|-------------------|
| Resume text | List of jobs, employers, date ranges |
| Medical record | List of symptoms, diagnoses |
| Customer email | Intent classification, sentiment, entities |

!!! note "Key Insight"
    LLMs are exceptionally powerful at processing **unstructured or semi-structured text** and extracting structured data. This is one of their highest-ROI applications.

??? question "Discussion: Schema Design"
    How does designing output schemas for LLMs differ from designing database schemas? What happens when the model encounters data that doesn't fit your schema?
