# Configuration

## Agent Configuration

`src/pineapple/config/agents.yaml`:

```yaml
researcher:
  role: "Senior Data Researcher"
  goal: "Uncover cutting-edge developments in {topic}"
  backstory: >
    You're a seasoned researcher with a knack for uncovering
    the latest developments in {topic}. Known for your ability
    to find the most relevant information and present it clearly.

reporting_analyst:
  role: "Reporting Analyst"
  goal: "Create detailed reports based on {topic} data analysis and research findings"
  backstory: >
    You're a meticulous analyst with a keen eye for detail.
    Known for turning complex data into clear and concise reports.
```

## Task Configuration

`src/pineapple/config/tasks.yaml`:

```yaml
research_task:
  description: >
    Conduct thorough research about {topic}.
    Make sure you find any interesting and relevant information
    given the current year is {current_year}.
  expected_output: >
    A list with 10 bullet points of the most relevant
    information about {topic}.
  agent: researcher

reporting_task:
  description: >
    Review the context you got and expand each topic into
    a full section for a report. Make sure the report is
    detailed and contains any and all relevant information.
  expected_output: >
    A fully fledged report with the main topics, each with
    a full section of information. Formatted as markdown.
  agent: reporting_analyst
  output_file: report.md
```

## Knowledge Base

Place reference files in `knowledge/`:

```
knowledge/
└── user_preference.txt  # User profile and preferences
```

crewAI agents can reference knowledge base files during execution.

## Environment Variables

| Variable | Required | Description |
|---|---|---|
| `OPENAI_API_KEY` | Yes | OpenAI API key |
| `OPENAI_MODEL_NAME` | No | Model name (default: gpt-4o) |
