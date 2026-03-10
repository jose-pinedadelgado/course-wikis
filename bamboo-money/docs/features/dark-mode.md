# Dark Mode & Internationalization

## Dark Mode

Full dark theme toggle available on every page.

- CSS custom properties for theme colors
- Toggle persisted via `localStorage`
- Charts and visualizations adapt to dark/light theme
- Bootstrap 5 dark mode utilities

## Internationalization (i18n)

Bamboo Money supports multiple languages via Django's translation framework.

### Supported Languages

- 🇺🇸 **English** (default)
- 🇪🇸 **Spanish**

### Implementation

- Django `{% trans %}` template tags throughout all templates
- Chart.js labels included in translation scope
- `.po` files compiled with `polib` (replaced custom compiler that had UTF-8 charset bugs)
- Language switcher in the navigation bar
- User language preference persisted in session

### Adding New Languages

1. Create `.po` file: `django-admin makemessages -l <locale>`
2. Translate strings in `locale/<locale>/LC_MESSAGES/django.po`
3. Compile: `python manage.py compilemessages` (uses polib)
