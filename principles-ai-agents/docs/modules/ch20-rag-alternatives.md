# Chapter 20 — Alternatives to RAG

## Do You Actually Need RAG?

!!! warning "The Most Important Question"
    Before building a RAG pipeline, ask: **can I just put everything in the context window?**

### Full Context Loading

With context windows reaching **2M tokens** (Gemini), many use cases that previously required RAG can now use direct context loading:

| Approach | Tokens | Cost | Complexity |
|----------|--------|------|------------|
| Full context | Up to 2M | Higher per-call | Minimal |
| RAG | Any size | Lower per-call | Significant |

### When RAG Is Still Needed

- Knowledge base is **truly massive** (millions of documents)
- **Real-time updates** — new data needs to be searchable immediately
- **Cost-sensitive** at scale — cheaper to retrieve than to send everything
- **Attribution** — need to cite specific sources

## Other Alternatives

### Fine-Tuning

Bake knowledge directly into model weights. Good for:

- Domain-specific terminology
- Consistent style/tone
- Tasks where retrieval latency matters

### Graph RAG (Knowledge Graphs)

Structure your knowledge as a **graph** (entities + relationships) instead of flat chunks. Better for questions that require **reasoning across multiple documents**.

### Agentic RAG

Instead of a fixed pipeline, let an **agent decide** how to search:

- Reformulate queries
- Search multiple sources
- Verify and cross-reference results

!!! tip "The Trend"
    As context windows grow, the threshold for needing RAG keeps rising. Many teams have ripped out RAG pipelines in favor of direct context loading.

??? question "Discussion: Context Window Economics"
    If context windows keep doubling, does RAG become obsolete? What's the floor — what will always need retrieval?
