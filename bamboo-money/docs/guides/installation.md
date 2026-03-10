# Installation

## Prerequisites

- Python 3.12+
- [UV](https://docs.astral.sh/uv/) package manager
- Git

## Setup

```bash
# Clone the repository
git clone https://github.com/jose-pinedadelgado/bamboo-money.git
cd bamboo-money

# Install dependencies
uv sync

# Set up environment variables
cp .env.example .env
# Edit .env with your OpenAI API key (for chatbot)

# Run migrations
uv run python manage.py migrate

# Create a superuser
uv run python manage.py createsuperuser

# Seed demo data (optional)
uv run python manage.py seed_demo2

# Run the server
uv run python manage.py runserver 8400
```

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `OPENAI_API_KEY` | For chatbot | GPT-4o-mini API key |
| `SECRET_KEY` | Yes | Django secret key |
| `DEBUG` | No | `True` for development |

## Demo Accounts

After seeding:

- **demo/demo** — Sample data for testing
- **demo2/demo2** — Real financial data (34 accounts, 204 history records)
