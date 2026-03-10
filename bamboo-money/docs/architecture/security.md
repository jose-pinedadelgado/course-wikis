# Security

## Authentication & Authorization

- Django's built-in auth system
- Login required on all financial views
- Session-based authentication with CSRF protection
- `CSRF_COOKIE_HTTPONLY = False` to allow HTMX/fetch JS access to the token

## Data Isolation

Every query is scoped to the authenticated user:

```python
transactions = Transaction.objects.filter(user=request.user)
```

**63 security tests** verify:

- Users cannot access other users' transactions, budgets, or net worth data
- Cross-user URL manipulation returns 403/404
- API endpoints enforce user scoping

## CSRF Protection

- Django CSRF middleware enabled globally
- HTMX requests inject CSRF token via `htmx:configRequest` event listener
- Language switch form refreshes CSRF token on submit (fixes BUG-004)

## XSS Prevention

- Django auto-escapes template variables
- Chart data sanitized before rendering
- User input in chatbot responses escaped

## SQL Injection

- Django ORM parameterizes all queries
- No raw SQL used anywhere in the application

## Chatbot Security

- API key stored in environment variable, never exposed to frontend
- 7 prompt injection payloads tested and blocked
- User data scoped — chatbot context builder only includes authenticated user's data
- Rate limiting per session

## Tested Vulnerabilities

All verified by `test_security.py` (63 tests):

- ✅ Authentication enforcement
- ✅ Cross-user data isolation
- ✅ CSRF token validation
- ✅ XSS in user inputs
- ✅ SQL injection attempts
- ✅ Chatbot prompt injection
- ✅ API key exposure prevention
