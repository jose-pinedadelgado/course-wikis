# Chapter 3 — Writing Great Prompts

## Prompting Techniques

### Shot Approaches

| Technique | Description | Quality | Effort |
|-----------|-------------|---------|--------|
| **Zero-shot** | Ask directly, no examples | Low-medium | Minimal |
| **Single-shot** | One example (input + output) | Medium | Low |
| **Few-shot** | Multiple examples for precise control | High | Medium |

!!! note "More examples = more guidance, but also more tokens (time + cost)"

### The "Seed Crystal" Approach

Not sure where to start? Ask the model to generate a prompt for you:

> "Generate a prompt for requesting a picture of a dog playing with a whale."

This gives you a solid v1 to refine. You can then ask the model to suggest improvements.

!!! tip "Model-Specific Prompts"
    Ask the **same model** you'll be prompting to generate the prompt. Claude is best at generating prompts for Claude, GPT-4o for GPT-4o, etc.

### System Prompts

Set a **system prompt** to define agent characteristics (tone, persona, role). This shapes how the agent responds but usually doesn't improve accuracy.

**Fun experiment:** Ask the same question with different personas — Steve Jobs vs. Bill Gates, Harry Potter vs. Draco Malfoy.

### Formatting Tricks

| Technique | Effect |
|-----------|--------|
| **CAPITALIZATION** | Adds weight to key words |
| **XML-like structure** | Helps models follow instructions precisely |
| **Structured prompts** (task, context, constraints) | Better adherence, especially with Claude & GPT-4 |

!!! warning "Small Changes, Big Differences"
    Formatting tweaks can dramatically affect output quality. Measure with evals (Chapter 27+).

### Production Prompts Are Detailed

Real production prompts are **extremely detailed** — the book shows ~1/3 of a live code-generation prompt from bolt.new. Don't be afraid to write long, specific prompts.

??? question "Discussion: Prompt Engineering as a Skill"
    Is prompt engineering a lasting skill, or will it become obsolete as models improve? What's the equivalent in traditional software engineering?
