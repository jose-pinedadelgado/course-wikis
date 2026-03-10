# Data Model

## Core Models

```mermaid
erDiagram
    User ||--o{ Transaction : has
    User ||--o{ Budget : has
    User ||--o{ NetWorthEntry : has
    User ||--o{ SavingsGoal : has
    User ||--o{ CategorizationRule : has
    User ||--o{ RecurringTransaction : has
    
    Category ||--o{ Transaction : categorizes
    Category ||--o{ Budget : tracks
    
    NetWorthEntry ||--o{ NetWorthEntryHistory : "snapshots"
    
    Transaction {
        date date
        string merchant
        decimal amount
        string notes
        bool is_recurring
        FK category
        FK user
    }
    
    Budget {
        int month
        int year
        decimal amount
        decimal rollover
        FK category
        FK user
    }
    
    NetWorthEntry {
        string name
        decimal balance
        string account_group
        date date
        FK user
    }
    
    NetWorthEntryHistory {
        decimal amount
        date date
        FK entry
    }
    
    SavingsGoal {
        string name
        decimal target_amount
        decimal current_amount
        date target_date
        FK user
    }
    
    RecurringTransaction {
        string merchant
        decimal expected_amount
        string frequency
        date next_expected_date
        bool is_active
        FK user
    }
    
    CategorizationRule {
        string pattern
        string match_type
        int priority
        FK category
        FK user
    }
    
    Category {
        string name
        string icon
        FK user
    }
```

## Key Relationships

- **User isolation** — Every model has a `user` FK. Users can never see each other's data.
- **Budget rollover** — Calculated at query time from previous month's budget minus actual spending.
- **Net worth history** — `NetWorthEntryHistory` stores periodic snapshots; the entry itself holds the current balance.
- **Account groups** — `NetWorthEntry.account_group` is a CharField with choices: `liquid`, `liability`, `investment`, `retirement`, `other_asset`.
