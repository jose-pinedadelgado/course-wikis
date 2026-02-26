# Chapter 19 — Setting Up Your RAG Pipeline

## End-to-End Setup

### 1. Document Ingestion

- Parse source documents (PDF, HTML, markdown, etc.)
- Clean and normalize text
- Split into chunks

### 2. Embedding & Storage

- Generate embeddings for each chunk
- Store in your vector database with metadata

### 3. Query Pipeline

```
User question → Embed query → Search → Re-rank → Format context → Generate answer
```

## Advanced Techniques

### Hybrid Search

Combine **vector similarity** with **keyword search** (BM25). This catches cases where semantic similarity misses exact terms.

### Re-ranking

After initial retrieval, use a **cross-encoder** or second LLM call to re-rank results by relevance. Slower but more accurate.

### Metadata Filtering

Filter results by metadata **before** vector search:

- Document type
- Date range
- Department/category
- Access permissions

!!! tip "Iterate, Don't Perfect"
    Get a basic pipeline working, then improve based on actual failure cases. Over-engineering RAG upfront is a common trap.

??? question "Discussion: RAG Failure Modes"
    What are the most common ways a RAG pipeline fails? (Think: bad chunks, embedding mismatches, retrieval misses, hallucination despite context.)
