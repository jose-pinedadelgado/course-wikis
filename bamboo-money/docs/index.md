# 🎋 Bamboo Money

**A personal finance dashboard built with Django, Chart.js, and HTMX.**

Bamboo Money helps you track spending, manage budgets, monitor net worth, and visualize cash flow — all in one place. It features an AI-powered chatbot, automatic bank CSV import, and a Sankey diagram for cash flow analysis.

---

## Key Features

| Feature | Description |
|---------|-------------|
| 📊 **Dashboard** | Summary cards, donut charts, spending trends, budget progress |
| 💰 **Budgets** | Zero-based budgeting with automatic monthly rollover |
| 📈 **Net Worth** | Track 34+ accounts across 5 groups with 8 financial metrics |
| 🔀 **Sankey Cash Flow** | D3.js-powered income → category → account flow diagrams |
| 🤖 **AI Chatbot** | GPT-4o-mini with inline charts, tables, and follow-up suggestions |
| 📥 **CSV Import** | Auto-detect 10+ bank formats, smart deduplication |
| 📤 **Excel Export** | Formatted .xlsx with Dashboard, Tracker, and Charts tabs |
| 🎯 **Savings Goals** | Visual progress bars with target dates |
| 🔄 **Recurring Detection** | Auto-detect subscriptions and recurring charges |
| 🏷️ **Categorization Rules** | Auto-assign categories based on merchant patterns |
| 🌙 **Dark Mode** | Full dark theme with toggle |
| 🌐 **i18n** | English + Spanish (Django translations) |

---

## Tech Stack

- **Backend:** Django 5.x, Python 3.12
- **Frontend:** Bootstrap 5, HTMX, Chart.js, D3.js
- **Database:** SQLite (dev), PostgreSQL (prod-ready)
- **AI:** OpenAI GPT-4o-mini for chatbot
- **Testing:** 509 tests across 8 test files
- **Package Manager:** UV

---

## Quick Links

- [Installation Guide](guides/installation.md)
- [Quick Start](guides/quickstart.md)
- [Feature Overview](features/index.md)
- [Architecture](architecture/tech-stack.md)

---

*Built by Jose Pineda · Private repo: `jose-pinedadelgado/bamboo-money`*
