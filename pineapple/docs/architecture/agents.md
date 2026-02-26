# Agents & Tasks

## Agents

Agents are defined in `src/pineapple/config/agents.yaml`.

### Researcher Agent

| Property | Value |
|---|---|
| **Role** | Senior Data Researcher |
| **Goal** | Uncover cutting-edge developments on specified topics |
| **Capabilities** | Web search, information gathering, fact extraction |
| **Model** | GPT-4o |

The Researcher performs web searches and knowledge base queries to gather relevant information. It produces a structured list of 10 bullet points summarizing findings.

### Reporting Analyst

| Property | Value |
|---|---|
| **Role** | Reporting Analyst |
| **Goal** | Create detailed reports from research data |
| **Capabilities** | Report generation, data formatting, content expansion |
| **Model** | GPT-4o |
| **Output** | `report.md` |

The Analyst takes the Researcher's findings and expands them into a comprehensive markdown report with sections, analysis, and recommendations.

## Tasks

Tasks are defined in `src/pineapple/config/tasks.yaml`.

### Research Task

| Property | Value |
|---|---|
| **Description** | Conduct thorough research on specified topic |
| **Expected Output** | 10 bullet points of relevant information |
| **Assigned To** | Researcher Agent |

### Reporting Task

| Property | Value |
|---|---|
| **Description** | Review research and expand into full report |
| **Expected Output** | Complete markdown report with detailed sections |
| **Assigned To** | Reporting Analyst |
| **File Generated** | `report.md` |

## Future Agents (Planned)

| Agent | Role | Purpose |
|---|---|---|
| **Course Expert** | Domain specialist | Answers student questions using wiki content |
| **Quiz Generator** | Assessment creator | Generates practice questions from course material |
| **Study Guide Builder** | Content summarizer | Creates condensed study guides from full content |
| **Office Hours Bot** | Student support | Real-time Q&A during office hours |
