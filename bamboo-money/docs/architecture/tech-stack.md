# Tech Stack

## Core Technologies

| Layer | Technology | Version | Purpose |
|---|---|---|---|
| **Language** | Python | 3.12 | Backend runtime |
| **Framework** | Django | 6.0 | Web framework, ORM, auth, admin |
| **Database** | SQLite | Built-in | Development database (PostgreSQL for production) |
| **Frontend** | Bootstrap | 5.3 | Responsive UI, components, dark mode |
| **Interactivity** | HTMX | Latest | Partial page updates without JavaScript |
| **Charts** | Chart.js | Latest | Interactive data visualizations |
| **Icons** | Bootstrap Icons | Latest | UI iconography |
| **Static Files** | WhiteNoise | Latest | Static file serving without Nginx |
| **Package Manager** | UV | Latest | Fast Python dependency management |

## Why This Stack?

### Django + Server-Rendered HTML

Bamboo Money deliberately avoids the SPA (Single Page Application) pattern:

- **No React/Vue/Angular** — Django templates render full HTML server-side
- **No REST API** — views return HTML directly, not JSON
- **HTMX for dynamic parts** — inline edits and partial updates without a JS framework

This results in:

- Faster initial page loads (no JS bundle to download)
- Simpler deployment (one process, one codebase)
- Full-stack Python (no TypeScript/JavaScript build pipeline)
- SEO-friendly by default

### Bootstrap 5

- Built-in dark mode via `data-bs-theme` attribute
- Comprehensive component library (navbars, cards, progress bars, modals)
- Responsive grid system
- No jQuery dependency (v5+)

### HTMX

Two specific uses in Bamboo Money:

1. **Inline category editing** on the Transactions page — changing a dropdown updates just that table row
2. **Rule creation from transaction** — click a button, get instant feedback without page reload

```html
<!-- Example: inline category update -->
<select hx-post="/transactions/{{ txn.id }}/category/"
        hx-target="#row-{{ txn.id }}"
        hx-swap="outerHTML">
```

### Chart.js

Four chart types used:

| Chart | Page | Type |
|---|---|---|
| Category spending | Dashboard | Doughnut/Pie |
| Monthly trend | Dashboard | Bar (grouped) |
| Spending pace | Dashboard | Line (dual series) |
| Net worth history | Net Worth | Line |

### UV Package Manager

- **10-100x faster** than pip for dependency resolution
- Deterministic lockfile (`uv.lock`)
- Built-in virtual environment management
- Compatible with `pyproject.toml`

## Dependencies

### Runtime

```toml
[project]
dependencies = [
    "django>=6.0",
    "django-htmx",
    "whitenoise",
]
```

### Development Only

The development environment includes additional tools installed via UV:

```bash
uv sync  # Installs all dependencies from uv.lock
```

## Production Recommendations

For production deployment, the stack would expand:

| Component | Development | Production |
|---|---|---|
| Database | SQLite | PostgreSQL |
| Web Server | `manage.py runserver` | Gunicorn |
| Reverse Proxy | None | Nginx |
| Static Files | WhiteNoise (still works) | Nginx or CDN |
| Secret Management | Hardcoded in settings | Environment variables |
| Container | None | Docker + docker-compose |

See [Security](security.md) for production hardening requirements.
