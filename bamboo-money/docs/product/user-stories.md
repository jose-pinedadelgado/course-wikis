# User Stories

Organized by persona and epic. Each story follows: *"As a [persona], I want to [action], so that [value]."*

## Personas

| Persona | Name | Description | Primary Need |
|---|---|---|---|
| 🎓 Student | **Sofia** | 22, grad student, tight budget | Simple spending visibility on a free tool |
| 💼 Professional | **Marcus** | 35, dual income, multiple cards | Privacy-first budgeting without Plaid |
| 👫 Couple | **Ana & David** | 30s, married, planning for house | Shared financial visibility |
| 📊 Optimizer | **Priya** | 40, high earner | Comprehensive dashboard + AI insights |

---

## Epic 1: First-Time Experience

### US-1.1: Quick Signup ✅
**As** any new user, **I want to** create an account in under 60 seconds, **so that** I can start using the app without friction.

- [x] Register with username + password
- [x] Default budget categories auto-created (12 categories)
- [x] Redirected to dashboard after registration

### US-1.2: First Data Import ✅
**As** Sofia, **I want to** upload my bank's CSV export, **so that** I can immediately see where my money went.

- [x] Upload CSV via drag-and-drop or file picker
- [x] Bank auto-detected from CSV headers
- [x] Transactions extracted and displayed for review
- [x] Duplicate detection prevents double-importing

### US-1.3: Guided Budget Setup
**As** Sofia, **I want** the app to suggest budget limits based on my actual spending, **so that** I don't have to guess what's realistic.

- [ ] After first import, suggest limits per category based on spending
- [ ] User can accept, modify, or skip suggestions
- [ ] Can be completed in under 3 minutes

---

## Epic 2: Core Budgeting Loop

### US-2.1: Budget Dashboard ✅
**As** Marcus, **I want to** see my budget status at a glance, **so that** I know if I'm on track without digging.

- [x] Total spent vs. total budget this month
- [x] Per-category progress bars with color coding
- [x] Days remaining in month
- [x] Recent transactions visible
- [x] Loads in under 2 seconds

### US-2.2: Transaction Review ✅
**As** Marcus, **I want to** quickly review and fix transaction categories, **so that** my budget is accurate.

- [x] Transaction list with search, date filter, category filter
- [x] Category changeable inline via HTMX (no page reload)
- [x] "Uncategorized" filterable
- [ ] Bulk select + reassign (not yet implemented)

### US-2.3: Manual Transaction Entry ✅
**As** Sofia, **I want to** add cash purchases manually, **so that** my budget includes all spending.

- [x] "Add transaction" form
- [x] Fields: date, description, amount, category, account
- [x] Amount reflected in budget tracking immediately

### US-2.4: Budget Alerts ✅
**As** Sofia, **I want to** get warned when I'm close to my budget limit, **so that** I can slow down.

- [x] Alert at 80% of category limit
- [x] Alert when limit exceeded
- [x] Unusual spending detection (>1.5× average)
- [x] Dismissible alert banners on dashboard

---

## Epic 3: Analytics & Insights

### US-3.1: Spending Breakdown ✅
**As** Priya, **I want to** see where my money goes visually, **so that** I can identify optimization areas.

- [x] Pie chart: spending by category
- [ ] Bar chart: budget vs. actual per category
- [ ] Click category to see its transactions

### US-3.2: Spending Trends ✅
**As** Marcus, **I want to** see month-over-month spending changes, **so that** I can spot patterns.

- [x] 6-month trend chart (income vs. expenses)
- [ ] Per-category trend lines
- [ ] Highlight changes ("15% less on dining this month")

### US-3.3: Cash Flow Forecast ✅
**As** Priya, **I want to** see if I'm on pace with my budget, **so that** I can adjust mid-month.

- [x] Daily spending pace chart (actual vs. linear budget)
- [x] End-of-month projection
- [x] Upcoming recurring bills listed

---

## Epic 4: Smart Features

### US-4.1: Auto-Categorization ✅
**As** Marcus, **I want** the app to automatically categorize my transactions, **so that** I don't fix every one manually.

- [x] Keyword-based rule matching
- [x] Create rules from transaction (one-click)
- [x] Bulk apply rules to uncategorized
- [ ] ML-based categorization (future)

### US-4.2: Recurring Detection ✅
**As** Marcus, **I want** the app to detect my subscriptions, **so that** I know what I'm paying for.

- [x] Auto-detect from transaction patterns
- [x] Shows next expected date and frequency
- [x] Total monthly recurring cost
- [x] Can confirm, pause, or delete

### US-4.3: AI Insights
**As** Priya, **I want** AI-generated insights about my spending, **so that** I discover patterns I wouldn't notice.

- [ ] Monthly auto-generated summary
- [ ] Identifies unusual spikes and savings opportunities
- [ ] Natural language queries

---

## Epic 5: Financial Tracking

### US-5.1: Net Worth ✅
**As** Priya, **I want to** track my net worth over time, **so that** I see the big picture.

- [x] Manual asset/liability entries
- [x] Net worth = assets − liabilities
- [x] Historical chart from monthly snapshots

### US-5.2: Savings Goals ✅
**As** Ana & David, **I want to** set savings goals with deadlines, **so that** we track our house down payment.

- [x] Create goals with target amount and deadline
- [x] Add contributions with dates and notes
- [x] Progress bar and monthly projection
- [x] Days remaining countdown

---

## Epic 6: Household (Future)

### US-6.1: Invite Household Member
**As** Ana, **I want to** invite David to share our budget, **so that** we both see the same financial picture.

- [ ] Invite by email
- [ ] Shared transactions, budgets, and goals
- [ ] Both can upload and categorize

---

## Epic 7: Data Portability

### US-7.1: Export Data ✅
**As** Marcus, **I want to** export my data, **so that** I'm never locked in.

- [x] Export transactions to CSV
- [ ] Export full data as JSON backup

### US-7.2: Import Data ✅
**As** Marcus, **I want to** import from CSV, **so that** I can bring in bank data.

- [x] Upload CSV with auto-detection
- [x] Preview before import
- [x] Duplicate detection
- [ ] Auto-apply categorization rules on import

---

## Story Completion Summary

| Status | Count |
|---|---|
| ✅ Complete | 15 stories |
| 🔶 Partial | 3 stories (some criteria met) |
| 🔴 Not started | 4 stories |
| **Total** | **22 stories** |
