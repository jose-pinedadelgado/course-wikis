# Integrations

## Current

### OpenAI

Primary LLM provider. Both agents use GPT-4o.

```python
# Configured via .env
OPENAI_API_KEY=sk-your-key
OPENAI_MODEL_NAME=gpt-4o
```

## Available (Dependencies Installed)

These integrations have dependencies installed but aren't fully wired up yet:

### ChromaDB (Vector Database)

For semantic search over course content:

```python
# Future: embed course wiki pages for RAG
import chromadb
client = chromadb.Client()
collection = client.create_collection("course-content")
```

### Neo4j (Graph Database)

For knowledge graph relationships:

```python
# Future: model course prerequisites, topic connections
from neo4j import GraphDatabase
driver = GraphDatabase.driver(uri, auth=(user, password))
```

### AWS S3

For document storage and retrieval:

```python
# Future: store and retrieve course materials
import boto3
s3 = boto3.client('s3')
```

### Oracle Database

For enterprise data source integration:

```python
# Future: connect to institutional databases
import oracledb
```

## Planned

| Integration | Purpose | Priority |
|---|---|---|
| Chalq wiki output | RAG over course wiki markdown | High |
| Canvas API | Direct course content access | Medium |
| Streamlit chat | Student Q&A interface | Medium |
| LiteLLM | Multi-provider LLM support (Claude, etc.) | Low |
