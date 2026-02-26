# Tech Stack

## Core Dependencies

### AI/ML

| Technology | Version | Purpose |
|---|---|---|
| crewAI | ≥0.118, <1.0 | Multi-agent orchestration framework |
| OpenAI | ≥1.77 | LLM integration (GPT-4o) |
| LiteLLM | — | LLM abstraction layer |
| LangChain Core | — | LLM framework utilities |
| Instructor | — | Structured output extraction |

### Data & Storage

| Technology | Purpose |
|---|---|
| ChromaDB | Vector database for semantic search |
| Neo4j | Graph database for knowledge relationships |
| SQLAlchemy | ORM for relational databases |
| Oracle DB | Enterprise data source integration |
| AWS S3 (boto3) | Document storage |
| Pandas | Data manipulation |

### Web & UI

| Technology | Purpose |
|---|---|
| Django (≥5.2) | Web backend framework |
| Streamlit (≥1.45) | Interactive dashboard UI |
| streamlit-chat | Chat component for Q&A interface |

### Utilities

| Technology | Purpose |
|---|---|
| BeautifulSoup4 | Web scraping |
| Pydantic | Data validation |
| PyMuPDF | PDF processing |
| python-docx | Word document processing |

## Why crewAI?

crewAI provides:

- **Role-based agent design** — each agent has a defined role, goal, and backstory
- **Sequential and hierarchical execution** — tasks can run in order or be delegated
- **Tool integration** — agents can use custom tools (web search, file access, APIs)
- **Memory and knowledge** — agents can access shared knowledge bases
- **Training mode** — iterative improvement with performance tracking
- **Replay capability** — re-execute previous runs for debugging

## Environment Configuration

```bash
# .env file
OPENAI_API_KEY=<your-api-key>
OPENAI_MODEL_NAME=gpt-4o
```

All AI calls go through OpenAI. Future versions may support Anthropic Claude via LiteLLM.
