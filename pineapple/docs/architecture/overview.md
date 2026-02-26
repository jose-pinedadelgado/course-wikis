# System Overview

## Architecture

```mermaid
graph TB
    subgraph Input["Input Layer"]
        CLI["CLI Commands"]
        Config["YAML Configs"]
        Knowledge["Knowledge Base"]
    end

    subgraph Crew["crewAI Orchestration"]
        Researcher["🔍 Researcher Agent"]
        Analyst["📊 Reporting Analyst"]
    end

    subgraph Providers["AI & Data"]
        OpenAI["OpenAI GPT-4o"]
        Chroma["ChromaDB"]
        Neo4j["Neo4j"]
        S3["AWS S3"]
    end

    subgraph Output["Output"]
        Report["report.md"]
        UI["Streamlit Dashboard"]
    end

    CLI --> Crew
    Config --> Crew
    Knowledge --> Researcher
    Researcher --> OpenAI
    Researcher --> Analyst
    Analyst --> OpenAI
    Analyst --> Report
    Chroma --> Researcher
    Neo4j --> Researcher
    S3 --> Researcher
    Report --> UI
```

## Data Flow

```
User Input (topic, year)
        │
        ▼
┌───────────────────┐
│  crewAI Crew      │
│  Orchestration    │
└───────────────────┘
        │
        ▼
┌───────────────────┐
│  Researcher Agent │──► Web Search & Information Gathering
└───────────────────┘
        │
        ▼ (10 bullet points)
┌───────────────────┐
│ Reporting Analyst │──► Format & Expand Content
└───────────────────┘
        │
        ▼
┌───────────────────┐
│  report.md        │──► Markdown Output File
└───────────────────┘
```

## Project Structure

```
3_Pineapple_Course_Assistant/
├── src/pineapple/
│   ├── main.py          # Entry point and CLI handlers
│   ├── crew.py          # Crew orchestration and agent definitions
│   ├── config/
│   │   ├── agents.yaml  # Agent configurations
│   │   └── tasks.yaml   # Task definitions
│   └── tools/
│       └── custom_tool.py  # Template for custom tools
├── knowledge/
│   └── user_preference.txt  # User profile and preferences
├── pyproject.toml
└── .env                 # API keys
```

## Execution Model

crewAI runs agents **sequentially** by default:

1. Researcher completes its task → produces 10 bullet points
2. Reporting Analyst receives the bullets → expands into full report
3. Output saved to `report.md`

The execution model can be configured to **hierarchical** mode where a manager agent delegates and coordinates.
