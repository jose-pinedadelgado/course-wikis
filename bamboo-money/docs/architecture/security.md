# Security

This page documents the current security posture, known issues, and remediation plan for Bamboo Money.

!!! danger "Development Only"
    Bamboo Money is currently configured for **development use only**. Do not deploy to production without addressing the findings below.

## Current Security Posture

### What's In Place

- ✅ Django CSRF middleware (enabled by default)
- ✅ Django template auto-escaping (XSS protection)
- ✅ `@login_required` on all views
- ✅ User-scoped querysets (`filter(user=request.user)`)
- ✅ Django ORM (parameterized queries, SQL injection protection)
- ✅ Password hashing (Django's default PBKDF2)

### What's Missing

- ❌ Production-grade `SECRET_KEY` (currently hardcoded)
- ❌ `DEBUG = False` for production
- ❌ Strict `ALLOWED_HOSTS`
- ❌ Password validators (disabled)
- ❌ HTTPS enforcement (HSTS, secure cookies)
- ❌ Login rate limiting
- ❌ Comprehensive test coverage
- ❌ Cross-user FK validation on category assignment

## Security Findings

### Critical

#### 1. Insecure Production Settings

The `settings.py` file has development defaults that are dangerous in production:

```python
SECRET_KEY = "hardcoded-dev-key"  # ⚠️ Must be env variable
DEBUG = True                       # ⚠️ Must be False
ALLOWED_HOSTS = ["*"]             # ⚠️ Must be restricted
AUTH_PASSWORD_VALIDATORS = []     # ⚠️ Must be re-enabled
```

**Fix:** Split settings into `base/dev/prod` and load secrets from environment variables.

#### 2. Cross-User Category Assignment

The `transaction_update_category` view accepts a category ID from POST data without verifying the category belongs to the current user:

```python
# Current (vulnerable):
cat_id = request.POST.get("category")
txn.category_id = int(cat_id)  # No ownership check!

# Fixed:
cat = get_object_or_404(BudgetCategory, pk=cat_id, user=request.user)
txn.category = cat
```

**Risk:** A user could bind their transaction to another user's category by guessing IDs.

### High

#### 3. GET-Based Logout

Logout is accessible via GET, making it CSRF-vulnerable:

```python
# Current:
def logout_view(request):
    auth_logout(request)  # Any GET request logs user out

# Fixed:
def logout_view(request):
    if request.method == "POST":
        auth_logout(request)
```

#### 4. No Brute-Force Protection

Login has no rate limiting. An attacker could attempt unlimited password guesses.

**Fix:** Add `django-axes` or custom throttling middleware.

#### 5. Non-Atomic Goal Updates

Savings goal contributions use a read-modify-write pattern:

```python
# Current (race condition):
goal.current_amount += contrib.amount
goal.save()

# Fixed:
from django.db.models import F
goal.current_amount = F('current_amount') + contrib.amount
goal.save(update_fields=['current_amount'])
```

## Remediation Plan

### Phase 0 — Blockers Before Production

1. Split settings by environment
2. Fix cross-user category assignment
3. Convert logout to POST-only
4. Remove/sanitize test fixture files

### Phase 1 — First Security Sprint

1. Add login rate limiting
2. Add authorization regression tests
3. Add secure headers (CSP, Referrer-Policy)
4. Add structured audit logging

### Phase 2 — Hardening

1. Migrate to PostgreSQL
2. Set up dependency vulnerability scanning
3. Add incident response runbook
4. Optional MFA support

## Test-First Bugfix Policy

All security bugs follow a strict policy:

1. Reproduce the bug
2. Write a **failing test** that captures expected correct behavior
3. Confirm the test fails
4. Implement the minimal fix
5. Confirm the test passes
6. Keep the regression test permanently

See `docs/TEST_FIRST_BUGFIX_POLICY.md` for the full policy.

## Deployment Checklist

Before any production deployment:

- [ ] `python manage.py check --deploy` passes with no security warnings
- [ ] `SECRET_KEY` loaded from environment/secret manager
- [ ] `DEBUG = False`
- [ ] `ALLOWED_HOSTS` restricted to actual domain
- [ ] HTTPS enforced (`SECURE_SSL_REDIRECT = True`)
- [ ] Secure cookies (`SESSION_COOKIE_SECURE`, `CSRF_COOKIE_SECURE`)
- [ ] HSTS enabled (`SECURE_HSTS_SECONDS`)
- [ ] Password validators re-enabled
- [ ] No sensitive data in repository
- [ ] Authorization test suite passing in CI
