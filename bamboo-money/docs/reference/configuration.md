# Configuration

Bamboo Money's configuration lives in `bamboo_site/settings.py`. This page documents all configurable settings.

## Django Settings

### Core Settings

| Setting | Default | Description |
|---|---|---|
| `SECRET_KEY` | Hardcoded | ⚠️ Must be set via environment variable in production |
| `DEBUG` | `True` | ⚠️ Must be `False` in production |
| `ALLOWED_HOSTS` | `["*"]` | ⚠️ Must be restricted in production |

### Installed Apps

```python
INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
    "django.contrib.humanize",  # For |intcomma template filter
    "django_htmx",              # HTMX middleware
    "interface",                # Main application
]
```

### Middleware Stack

```python
MIDDLEWARE = [
    "django.middleware.security.SecurityMiddleware",
    "whitenoise.middleware.WhiteNoiseMiddleware",  # Static files
    "django.contrib.sessions.middleware.SessionMiddleware",
    "django.middleware.common.CommonMiddleware",
    "django.middleware.csrf.CsrfViewMiddleware",
    "django.contrib.auth.middleware.AuthenticationMiddleware",
    "django_htmx.middleware.HtmxMiddleware",  # Adds request.htmx
    "django.contrib.messages.middleware.MessageMiddleware",
    "django.middleware.clickjacking.XFrameOptionsMiddleware",
]
```

### Database

```python
# Development (default)
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.sqlite3",
        "NAME": BASE_DIR / "db.sqlite3",
    }
}

# Production (recommended)
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.postgresql",
        "NAME": "bamboo_money",
        "USER": os.environ["DB_USER"],
        "PASSWORD": os.environ["DB_PASSWORD"],
        "HOST": os.environ.get("DB_HOST", "localhost"),
        "PORT": os.environ.get("DB_PORT", "5432"),
    }
}
```

### Static Files

```python
STATIC_URL = "static/"
STATICFILES_STORAGE = "whitenoise.storage.CompressedManifestStaticFilesStorage"
```

WhiteNoise serves static files directly from Django — no Nginx required for development.

### Templates

```python
TEMPLATES = [{
    "DIRS": [BASE_DIR / "templates"],  # Project-level templates
    # ...
}]
```

!!! info "Template Location"
    Templates live at the **project level** (`templates/interface/`), not inside the `interface` app. This was a deliberate choice for the combined prototype.

### Authentication

```python
LOGIN_URL = "/login/"
LOGIN_REDIRECT_URL = "/"
LOGOUT_REDIRECT_URL = "/login/"
```

### Password Validators

```python
AUTH_PASSWORD_VALIDATORS = []  # ⚠️ Disabled for development
```

For production, re-enable Django's default validators:

```python
AUTH_PASSWORD_VALIDATORS = [
    {"NAME": "django.contrib.auth.password_validation.UserAttributeSimilarityValidator"},
    {"NAME": "django.contrib.auth.password_validation.MinimumLengthValidator"},
    {"NAME": "django.contrib.auth.password_validation.CommonPasswordValidator"},
    {"NAME": "django.contrib.auth.password_validation.NumericPasswordValidator"},
]
```

## Application Constants

These values are hardcoded in views and could be made configurable:

| Constant | Location | Value | Purpose |
|---|---|---|---|
| Transactions per page | `transaction_list` | 25 | Pagination size |
| Dashboard recent transactions | `dashboard` | 10 | Recent items shown |
| Alert threshold (approaching) | `_check_budget_alerts` | 80% | When to warn |
| Alert threshold (exceeded) | `_check_budget_alerts` | 100% | When to alert |
| Unusual spending multiplier | `_check_budget_alerts` | 1.5× | vs 3-month average |
| Recurring min occurrences | `detect_recurrings` | 3 | Min charges to flag |
| CSV max rows per import | `csv_upload` | 500 | Session storage limit |
| Upcoming recurring window | `dashboard` | Current month | Bills shown |
| Dashboard alerts shown | `dashboard` | 5 | Max visible alerts |

## Port Assignments

For local development with multiple projects:

| Project | Port |
|---|---|
| MkDocs DB wiki | 8000 |
| Zensical DB wiki | 8100 |
| Responsible AI wiki | 8200 |
| Principles AI Agents wiki | 8300 |
| **Bamboo Money** | **8400** |
