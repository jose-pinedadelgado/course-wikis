# Tech Stack

## Backend

| Component | Technology | Purpose |
|-----------|-----------|---------|
| Framework | **Django 5.x** | Web framework, ORM, auth, admin |
| Language | **Python 3.12** | Core language |
| Package Manager | **UV** | Fast Python package management |
| Database | **SQLite** (dev) / **PostgreSQL** (prod) | Data persistence |
| AI | **OpenAI GPT-4o-mini** | Chatbot responses |

## Frontend

| Component | Technology | Purpose |
|-----------|-----------|---------|
| CSS Framework | **Bootstrap 5** | Layout, components, responsive design |
| Interactivity | **HTMX** | Partial page updates without full SPA |
| Charts | **Chart.js** | Donut, line, bar, area charts |
| Chart Labels | **chartjs-plugin-datalabels** | Permanent labels on donut charts |
| Sankey | **D3.js + d3-sankey** | Cash flow Sankey diagram |
| Templating | **Django Templates** | Server-side HTML rendering |

## Infrastructure

| Component | Technology | Purpose |
|-----------|-----------|---------|
| Server | **Django dev server / Gunicorn** | HTTP serving |
| Static Files | **WhiteNoise** | Static file serving without Nginx |
| i18n | **Django i18n + polib** | Multi-language support |
| Testing | **Django TestCase + pytest** | 509 tests |

## Why This Stack?

- **Django** — Batteries-included: auth, ORM, admin, i18n, CSRF protection all built in
- **HTMX** — Interactive UX without the complexity of React/Vue for a dashboard app
- **Chart.js** — Simple API for common chart types; D3.js only where needed (Sankey)
- **SQLite** — Zero-config for development; easy migration to PostgreSQL for production
- **UV** — 10-100x faster than pip for dependency resolution
