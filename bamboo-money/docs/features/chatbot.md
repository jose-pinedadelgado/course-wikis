# AI Chatbot

An AI-powered financial assistant that lets you explore your data through natural language conversation.

## Model

- **GPT-4o-mini** via OpenAI API
- Context builder assembles relevant financial data (recent transactions, budgets, net worth) into the system prompt
- Conversation history maintained per session

## Capabilities

- Answer questions about spending, budgets, and trends
- Generate inline visualizations (bar charts, pie charts, tables)
- Suggest follow-up questions (clickable green pills)
- Link to relevant pages (e.g., "View your Sankey diagram →")

### Example Queries

> "How much did I spend on food last month?"

> "What's my savings rate this year?"

> "Show me my top 5 expense categories as a chart"

> "Am I on track with my grocery budget?"

## Response Format

The chatbot returns structured JSON:

```json
{
  "message": "You spent $1,234 on food last month...",
  "visualization": {
    "type": "bar",
    "data": { ... }
  },
  "suggested_questions": [
    "How does that compare to the month before?",
    "What merchants do I spend the most at?"
  ]
}
```

## UI

- **Chat widget** — Floating bottom-right, CSS-based positioning (refactored from inline styles)
- **4 resize modes**: Small ▫ | Half-vertical ◧ | Half-horizontal ⬓ | Fullscreen ⬜
- **Thinking animation** — Sprouting seed animation while waiting for response
- **Inline charts** — Chart.js rendered directly in chat bubbles

## Security

- 7 prompt injection payloads tested and blocked
- User data isolation — chatbot only sees the authenticated user's data
- API key stored server-side, never exposed to frontend
- Rate limiting per user session
