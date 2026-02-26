# Contributing

Bamboo Money is a research project exploring AI-assisted software development and privacy-first personal finance. Contributions are welcome!

## Development Setup

```bash
git clone https://github.com/jose-pinedadelgado/bamboo-money-combined.git
cd bamboo-money-combined
uv sync
uv run python manage.py migrate
uv run python manage.py seed_demo_data
uv run python manage.py runserver 8400
```

## Project Conventions

### Code Style
- **Python:** Follow Django conventions and PEP 8
- **Templates:** Django template language with Bootstrap 5 classes
- **JavaScript:** Minimal — prefer HTMX for interactivity

### Git Workflow
- Work on `main` branch (prototype phase)
- Commit frequently with descriptive messages
- Run smoke tests before pushing: `uv run python smoke_test.py`

### Testing Policy
- All bug fixes require a **failing test first** (see `docs/TEST_FIRST_BUGFIX_POLICY.md`)
- Security bugs require negative + positive regression tests
- Run: `uv run python manage.py test`

### File Organization
- **Models, views, forms** — in `interface/` app
- **Templates** — in project-level `templates/interface/`
- **Management commands** — in `interface/management/commands/`
- **Documentation** — in `docs/`

## Adding a Feature

1. Check the [Roadmap](product/roadmap.md) for planned features
2. Create the model(s) in `interface/models.py`
3. Create migration: `uv run python manage.py makemigrations`
4. Add form(s) in `interface/forms.py`
5. Add view(s) in `interface/views.py`
6. Add URL pattern(s) in `interface/urls.py`
7. Create template(s) in `templates/interface/`
8. Add sidebar link in `base.html` if it's a new page
9. Update smoke test in `smoke_test.py`
10. Run smoke test: `uv run python smoke_test.py`
11. Commit and push

## Reporting Issues

When reporting bugs, include:

- Steps to reproduce
- Expected behavior
- Actual behavior
- Browser and OS
- Screenshots if applicable

## Project Links

- **Repository:** [github.com/jose-pinedadelgado/bamboo-money-combined](https://github.com/jose-pinedadelgado/bamboo-money-combined)
- **Security Policy:** `docs/SECURITY_PRODUCTION_READINESS.md`
- **Testing Policy:** `docs/TEST_FIRST_BUGFIX_POLICY.md`
- **Feature Plan:** `docs/IMPROVEMENT_PLAN.md`

---

*Built by [Jose Pineda](https://github.com/jose-pinedadelgado) — Assistant Professor, California State University, Long Beach*
