# Chapter 18 — Choosing a Vector Database

## What Is a Vector Database?

A database optimized for storing and searching **high-dimensional vectors** (embeddings). When a user asks a question, you embed the query and find the most similar stored vectors.

## Options

### Managed Cloud

| Service | Notes |
|---------|-------|
| **Pinecone** | Purpose-built, popular, serverless option |
| **Weaviate** | Open-source, cloud offering |
| **Qdrant** | Rust-based, fast |

### Vector Extensions (Add to Existing DB)

| Extension | Database | Notes |
|-----------|----------|-------|
| **pgvector** | PostgreSQL | Most popular; add vectors to your existing Postgres |
| **SQLite VSS** | SQLite | Lightweight, local-first |

### Open-Source / Self-Hosted

| Option | Notes |
|--------|-------|
| **Chroma** | Python-first, easy to start |
| **Milvus** | Large-scale, enterprise |

!!! tip "Start with pgvector"
    If you already use PostgreSQL, pgvector lets you add vector search without a new service. One less thing to manage.

## Embedding Models

You need an **embedding model** to convert text → vectors:

| Provider | Model | Dimensions |
|----------|-------|------------|
| OpenAI | text-embedding-3-small | 1,536 |
| Cohere | embed-english-v3 | 1,024 |
| Open-source | all-MiniLM-L6-v2 | 384 |

!!! warning "Lock In"
    Once you choose an embedding model, **you can't easily switch** — all stored vectors would need re-embedding. Choose carefully.

??? question "Discussion: Build vs. Buy"
    When is a purpose-built vector database worth it vs. pgvector? What scale triggers the switch?
