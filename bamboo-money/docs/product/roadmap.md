# Roadmap

## ✅ Completed

### v1.0 — Core Finance Dashboard
- Dashboard with summary cards, donut chart, spending trends
- Transaction CRUD with search and filtering
- Monthly budgets with category tracking
- CSV import with auto-bank detection (10+ formats)
- Savings goals with progress tracking
- Recurring transaction detection
- Categorization rules engine
- Dark mode + Spanish i18n

### v1.1 — AI & Visualizations
- GPT-4o-mini chatbot with inline charts and tables
- Sprouting seed thinking animation
- Suggested follow-up questions
- 4 chat resize modes (small, half-vertical, half-horizontal, fullscreen)
- Collapsible sidebar with localStorage persistence
- Sankey cash flow diagram (D3.js + d3-sankey)

### v1.2 — Net Worth Tracker
- `NetWorthEntry` and `NetWorthEntryHistory` models
- 5 account groups: liquid, liability, investment, retirement, other asset
- 8 key financial metrics on the net worth page
- Trend chart (assets, liabilities, net worth lines)
- Asset allocation donut with permanent data labels
- Stacked area composition chart
- Collapsible group breakdown cards
- Formatted Excel export (3 tabs: Dashboard, Tracker, Charts)
- `demo2` account with real financial data (34 accounts, 204 history records)

### v1.3 — Quality & Security
- 509 tests across 8 test files
- Security tests (CSRF, XSS, SQLi, data isolation, API key protection)
- Performance tests at 0/300/2K/10K transaction volumes
- Prompt injection protection for chatbot (7 payloads tested)

## 🔮 Future

### v2.0 — Multi-User & Cloud
- PostgreSQL migration for production
- User registration and authentication
- Household/family shared budgets
- Google Sheets API integration for net worth sync

### v2.1 — Advanced Analytics
- Spending predictions and anomaly detection
- Budget recommendations based on history
- Investment performance tracking
- Tax category tagging
