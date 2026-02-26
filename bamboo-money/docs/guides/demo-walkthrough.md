# Demo Walkthrough

A guided tour of every feature in Bamboo Money using the demo account.

## Login

Navigate to [http://localhost:8400](http://localhost:8400) and log in with `demo` / `demo`.

## 1. Dashboard

The dashboard is your financial command center. It shows:

### Summary Cards
- **Total Spent** this month
- **Total Income** this month
- **Total Budget** across all categories
- **Budget Progress** — overall percentage with color coding

### Budget Progress by Category
Each category shows a progress bar:

- :material-check-circle:{ .text-success } **Green** — under 80% spent
- :material-alert:{ .text-warning } **Yellow** — 80–99% spent
- :material-close-circle:{ .text-danger } **Red** — over budget

### Spending Alerts
Yellow and red banners appear when:

- A category reaches **80%** of its limit (approaching)
- A category **exceeds** its limit
- Spending is **1.5x higher** than your 3-month average (unusual)

Click the ✕ to dismiss individual alerts, or "Dismiss All."

### Cash Flow Forecast
The **Spending Pace** chart shows two lines:

- **Solid line** — your actual cumulative spending day by day
- **Dashed line** — where you'd be if spending evenly across the month

A badge tells you: *"Projected $X under/over budget by month end."*

### Upcoming Bills
If recurring transactions are detected, upcoming charges appear here with dates and amounts.

### Charts
- **Category Pie Chart** — where your money goes
- **6-Month Trend** — income vs. expenses over time

## 2. Transactions

Click **Transactions** in the sidebar.

### Filtering
Use the top bar to filter by:

- **Search** — matches description and merchant
- **Category** — dropdown filter
- **Date Range** — from/to date pickers
- **Type** — All, Income, or Expense

### Inline Category Editing
Each transaction row has a category dropdown. Change it and it saves instantly via HTMX — no page reload.

### Create Rule from Transaction
Click the :material-tag-plus: button on any transaction to create an auto-categorization rule from its merchant and current category.

### Export
Click **Export CSV** to download all filtered transactions as a CSV file.

### Pagination
Transactions display 25 per page to keep page sizes small (~62KB vs ~626KB for all at once).

## 3. CSV Import

Click **Import CSV** in the sidebar.

### Step 1: Upload
Drag and drop a bank CSV file or use the file picker. Supports `.csv` files up to 500 rows per import.

### Step 2: Preview
The parser auto-detects your bank:

| Bank | Detection Method |
|---|---|
| Chase | Header contains "Transaction Date" and "Post Date" |
| American Express | Header contains "Date" and "Reference" |
| Wells Fargo | Header pattern with specific column order |
| Bank of America | Header contains "Date" and "Payee" |

For unrecognized formats, you'll see a manual column mapping interface.

The preview shows:

- Parsed transactions with dates, descriptions, and amounts
- :material-alert: **Duplicate warnings** for transactions matching existing records (±1 day, same amount and description)
- Error count for rows that couldn't be parsed

### Step 3: Confirm
Select which transactions to import, then click **Import**. A success message shows how many were imported and how many duplicates were skipped.

## 4. Budgets

Click **Budgets** in the sidebar.

Each category shows:

- Icon, name, and monthly limit
- Progress bar with color coding
- Amount spent vs. limit
- **Effective budget** (base + rollover) if rollover is enabled

### Editing a Budget
Click **Edit** on any category to change:

- Name, icon, color, monthly limit
- **Rollover enabled** — carry unused budget forward
- **Rollover cap** — maximum amount that can roll over (0 = unlimited)

## 5. Categorization Rules

Click **Rules** in the sidebar.

### Creating Rules
Click **New Rule** and set:

- **Keyword** — text to match (e.g., "STARBUCKS")
- **Match Type** — Contains, Starts With, or Exact Match
- **Category** — which category to assign
- **Priority** — higher priority rules are checked first

### Applying Rules
Click **Apply All Rules** to run all rules against uncategorized transactions. A message shows how many transactions were categorized.

## 6. Recurring Transactions

Click **Recurring** in the sidebar.

### Detection
Click **Detect Recurring** to scan your transaction history. The algorithm:

1. Groups transactions by normalized merchant name
2. Calculates intervals between charges
3. Classifies frequency (weekly, biweekly, monthly, quarterly, yearly)
4. Requires 3+ occurrences with consistent intervals

### Management
- **Confirm** — mark a detected recurring as verified
- **Pause** — temporarily hide without deleting
- **Delete** — remove the recurring entry

### Dashboard Integration
Active recurring transactions appear as "Upcoming Bills" on the dashboard, showing charges expected in the current month.

## 7. Savings Goals

Click **Goals** in the sidebar.

### Creating a Goal
Set a name, icon, target amount, and optional deadline.

### Contributing
Click into a goal to see its detail page. Add contributions with an amount, date, and optional note. The goal's progress bar and "monthly needed" calculation update automatically.

### Projections
If a deadline is set, the goal shows:

- Days remaining
- Monthly amount needed to hit the target
- Progress percentage

## 8. Net Worth

Click **Net Worth** in the sidebar.

### Entries
Add assets (savings, investments, property) and liabilities (loans, credit card debt). Each entry has a name, type, amount, and optional category label.

### Summary
The page shows:

- Total assets
- Total liabilities
- **Net worth** (assets − liabilities)

### Historical Chart
Monthly snapshots create a line chart showing net worth over time.

## 9. Dark Mode

Click the :material-weather-night: **moon icon** at the bottom of the sidebar.

Dark mode:

- Uses Bootstrap 5's native `data-bs-theme="dark"` attribute
- Persists via `localStorage` — survives page refreshes and browser restarts
- Adapts all Chart.js charts with dark-friendly colors
- Includes CSS overrides for table headers, card borders, and sidebar contrast
