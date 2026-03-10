# Excel Export

Export your net worth data as a professionally formatted `.xlsx` file with three tabs.

## Endpoint

`GET /networth/export/` → Downloads `bamboo_money_networth.xlsx`

## Tabs

### Dashboard Tab

- Branded header with app name
- Summary cards: Total Assets, Total Liabilities, Net Worth
- 6-metric table (Liquid Months, Debt-to-Asset, FI Ratio, etc.)
- Account breakdown by group
- Embedded trend line chart (assets/liabilities/net worth)
- Embedded pie chart (asset allocation)

### Net Worth Tracker Tab

- Accounts as columns, dates as rows
- Color-coded group headers (blue=liquid, green=investment, purple=retirement, red=liability)
- Subtotals per group
- Month-over-month change column
- Frozen panes (headers stay visible while scrolling)
- Alternating row stripes

### Charts Tab

- Stacked area chart showing composition over time
- Bar chart breaking down by group

## Technology

Built with **openpyxl** for full control over formatting, charts, and styles. The export module lives at `interface/excel_export.py`.

## Future Enhancements

- Date range picker (export specific periods)
- Google Sheets API integration (live sync)
