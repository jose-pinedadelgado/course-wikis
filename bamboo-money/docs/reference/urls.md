# URL Routes

Complete list of all URL patterns in Bamboo Money.

## Authentication

| Method | URL | View | Description |
|---|---|---|---|
| GET/POST | `/login/` | `login_view` | Login page |
| GET/POST | `/register/` | `register_view` | Registration page |
| GET | `/logout/` | `logout_view` | Logout (⚠️ should be POST) |

## Dashboard

| Method | URL | View | Description |
|---|---|---|---|
| GET | `/` | `dashboard` | Main dashboard |

## Transactions

| Method | URL | View | Description |
|---|---|---|---|
| GET | `/transactions/` | `transaction_list` | List with filters + pagination |
| GET | `/transactions/export/` | `transaction_export` | CSV download |
| GET/POST | `/transactions/add/` | `transaction_add` | Create transaction |
| GET/POST | `/transactions/<pk>/edit/` | `transaction_edit` | Edit transaction |
| POST | `/transactions/<pk>/delete/` | `transaction_delete` | Delete transaction |
| POST | `/transactions/<pk>/category/` | `transaction_update_category` | HTMX inline category update |
| POST | `/transactions/<pk>/create-rule/` | `create_rule_from_transaction` | HTMX create categorization rule |

## CSV Import

| Method | URL | View | Description |
|---|---|---|---|
| GET/POST | `/import/` | `csv_upload` | Upload CSV file |
| GET/POST | `/import/preview/` | `csv_preview` | Preview parsed transactions |
| POST | `/import/confirm/` | `csv_confirm` | Confirm and import |

## Budgets

| Method | URL | View | Description |
|---|---|---|---|
| GET | `/budgets/` | `budget_list` | List budget categories |
| GET/POST | `/budgets/add/` | `budget_add` | Create category |
| GET/POST | `/budgets/<pk>/edit/` | `budget_edit` | Edit category |
| POST | `/budgets/<pk>/delete/` | `budget_delete` | Delete category |

## Categorization Rules

| Method | URL | View | Description |
|---|---|---|---|
| GET | `/rules/` | `rules_list` | List all rules |
| GET/POST | `/rules/add/` | `rule_add` | Create rule |
| GET/POST | `/rules/<pk>/edit/` | `rule_edit` | Edit rule |
| POST | `/rules/<pk>/delete/` | `rule_delete` | Delete rule |
| POST | `/rules/apply-all/` | `rules_apply_all` | Apply rules to uncategorized |

## Recurring Transactions

| Method | URL | View | Description |
|---|---|---|---|
| GET | `/recurring/` | `recurring_list` | List recurring transactions |
| POST | `/recurring/<pk>/toggle/` | `recurring_toggle` | Pause/resume |
| POST | `/recurring/<pk>/confirm/` | `recurring_confirm` | Confirm detection |
| POST | `/recurring/<pk>/delete/` | `recurring_delete` | Delete recurring |
| GET | `/recurring/detect/` | `recurring_detect` | Run detection |

## Spending Alerts

| Method | URL | View | Description |
|---|---|---|---|
| POST | `/alerts/<pk>/dismiss/` | `alert_dismiss` | HTMX dismiss single alert |
| GET | `/alerts/dismiss-all/` | `alert_dismiss_all` | Dismiss all alerts |

## Savings Goals

| Method | URL | View | Description |
|---|---|---|---|
| GET | `/goals/` | `goals_list` | List all goals |
| GET/POST | `/goals/add/` | `goal_add` | Create goal |
| GET/POST | `/goals/<pk>/` | `goal_detail` | View + add contribution |
| POST | `/goals/<pk>/delete/` | `goal_delete` | Delete goal |

## Net Worth

| Method | URL | View | Description |
|---|---|---|---|
| GET | `/networth/` | `networth` | Net worth overview |
| GET/POST | `/networth/add/` | `networth_add` | Add asset/liability |
| GET/POST | `/networth/<pk>/edit/` | `networth_edit` | Edit entry |
| POST | `/networth/<pk>/delete/` | `networth_delete` | Delete entry |
