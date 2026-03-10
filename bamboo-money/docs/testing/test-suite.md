# Test Suite

Bamboo Money has **509 tests** across 8 test files, covering security, functionality, performance, and edge cases.

## Test Files

| File | Tests | Coverage |
|------|------:|----------|
| `test_security.py` | 63 | Auth, data isolation, CSRF, XSS, SQLi, chatbot isolation, API key protection |
| `test_financial_calculations.py` | 52 | Budgets, rollover, savings, net worth, sankey, recurring, categorization rules |
| `test_usability_bugs.py` | 56 | Alert dismiss, all 13 pages load, CSRF, transaction/budget CRUD, chatbot scenarios, language switching, mobile, edge cases |
| `test_crud_workflows.py` | 115 | All model lifecycle operations (create, read, update, delete) |
| `test_csv_import.py` | 78 | Bank detection, CSV parsing, deduplication, auto-categorization |
| `test_e2e_journeys.py` | 71 | 10 multi-step user journeys end-to-end |
| `test_chatbot.py` | 39 | Accuracy, safety, 7 prompt injection payloads |
| `test_performance.py` | 33 | 0 / 300 / 2K / 10K transaction volume tiers |

## Running Tests

```bash
# Run all tests
uv run python manage.py test

# Run a specific test file
uv run python manage.py test tests.test_security

# Run with verbose output
uv run python manage.py test -v 2
```

## Performance Testing

The performance suite tests four volume tiers:

| Tier | Transactions | Expected Response |
|------|-------------|-------------------|
| Empty | 0 | < 100ms |
| Small | 300 | < 200ms |
| Medium | 2,000 | < 500ms |
| Large | 10,000 | < 2s |

## Security Testing Highlights

- **7 prompt injection payloads** tested against the chatbot (all blocked)
- **Cross-user data access** verified impossible across all models
- **CSRF enforcement** on all POST/PUT/DELETE endpoints
- **XSS** via merchant names, notes, and chatbot inputs
