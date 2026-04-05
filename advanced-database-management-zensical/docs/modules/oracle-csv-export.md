# How to Export Oracle Tables as CSV

> Three methods to get your star schema data out of Oracle and into Databricks.  
> Pick whichever works best for you.

---

## Method 1: SQL Developer (GUI)

The easiest method — no code required.

### Steps

1. **Open SQL Developer** and connect to your Oracle RDS instance
2. **Navigate** to your table in the Connections panel (expand your connection → Tables)
3. **Right-click** the table name (e.g., `FACT_ENROLLMENTS`)
4. Select **Export Data** → **Export Wizard** (or in newer versions: **Export...**)
5. Configure the export:
    - **Format:** CSV
    - **Header:** Yes (check the box — Spark needs column names)
    - **File:** Choose a location and name (e.g., `C:\IS480\fact_enrollments.csv`)
    - **Encoding:** UTF-8
    - **Delimiter:** Comma (default)
    - **Left/Right Enclosure:** Double quote `"` (handles commas in text)
6. Click **Next** → **Finish**
7. Repeat for each table

!!! tip "Export query results instead"
    You can also run a SELECT query first, then right-click the results grid → **Export**. This lets you export a filtered subset or joined result.

### What You'll See

The export wizard shows a preview of the first few rows. Verify that:

- Column headers appear in the first row
- Numbers aren't quoted unnecessarily
- Date columns exported in a readable format (YYYY-MM-DD)

---

## Method 2: Python Script

Best for exporting multiple tables at once or automating the process.

### Prerequisites

```bash
pip install oracledb
```

### Connection String Format

For Oracle RDS, your connection string looks like:

```
host:port/service_name
```

Example:

```
your-rds-instance.abc123.us-west-2.rds.amazonaws.com:1521/ORCL
```

!!! info "Find your connection details"
    Check the Oracle RDS connection info your instructor provided. You need: host, port (usually 1521), and service name (usually ORCL).

### Full Export Script

```python
import oracledb
import csv
import os

# ============================================
# Configuration — UPDATE THESE VALUES
# ============================================
DB_USER = "your_username"
DB_PASSWORD = "your_password"
DB_HOST = "your-rds-instance.abc123.us-west-2.rds.amazonaws.com"
DB_PORT = 1521
DB_SERVICE = "ORCL"

# Tables to export
TABLES = [
    "dim_student",
    "dim_course",
    "dim_date",
    "dim_class",
    "fact_enrollments"
]

# Output directory
OUTPUT_DIR = "csv_exports"

# ============================================
# Export Logic
# ============================================
def export_table(cursor, table_name, output_dir):
    """Export a single table to CSV."""
    output_file = os.path.join(output_dir, f"{table_name}.csv")

    # Get data
    cursor.execute(f"SELECT * FROM {table_name}")

    # Get column names from cursor description
    columns = [col[0] for col in cursor.description]

    # Write CSV
    with open(output_file, "w", newline="", encoding="utf-8") as f:
        writer = csv.writer(f)
        writer.writerow(columns)        # Header row
        writer.writerows(cursor)        # Data rows

    row_count = cursor.rowcount
    print(f"  Exported {table_name}: {row_count} rows -> {output_file}")


def main():
    # Create output directory
    os.makedirs(OUTPUT_DIR, exist_ok=True)

    # Connect to Oracle
    dsn = f"{DB_HOST}:{DB_PORT}/{DB_SERVICE}"
    print(f"Connecting to {dsn}...")

    connection = oracledb.connect(
        user=DB_USER,
        password=DB_PASSWORD,
        dsn=dsn
    )
    print("Connected!")

    cursor = connection.cursor()

    # Export each table
    for table in TABLES:
        try:
            export_table(cursor, table, OUTPUT_DIR)
        except Exception as e:
            print(f"  ERROR exporting {table}: {e}")

    # Cleanup
    cursor.close()
    connection.close()
    print(f"\nDone! Files saved to: {os.path.abspath(OUTPUT_DIR)}")


if __name__ == "__main__":
    main()
```

### Running the Script

```bash
python export_oracle.py
```

Expected output:

```
Connecting to your-rds-instance.abc123.us-west-2.rds.amazonaws.com:1521/ORCL...
Connected!
  Exported dim_student: 150 rows -> csv_exports/dim_student.csv
  Exported dim_course: 45 rows -> csv_exports/dim_course.csv
  Exported dim_date: 8 rows -> csv_exports/dim_date.csv
  Exported dim_class: 120 rows -> csv_exports/dim_class.csv
  Exported fact_enrollments: 890 rows -> csv_exports/fact_enrollments.csv

Done! Files saved to: C:\IS480\csv_exports
```

---

## Method 3: SQL*Plus SPOOL

Command-line method — useful if you only have terminal access.

### Basic Export

```sql
-- Connect to Oracle
-- sqlplus your_username/your_password@your-rds-host:1521/ORCL

-- Configure CSV output
SET MARKUP CSV ON
SET PAGESIZE 0
SET FEEDBACK OFF
SET TRIMSPOOL ON
SET LINESIZE 32767

-- Export dim_student
SPOOL dim_student.csv
SELECT * FROM dim_student;
SPOOL OFF

-- Export dim_course
SPOOL dim_course.csv
SELECT * FROM dim_course;
SPOOL OFF

-- Export dim_date
SPOOL dim_date.csv
SELECT * FROM dim_date;
SPOOL OFF

-- Export dim_class
SPOOL dim_class.csv
SELECT * FROM dim_class;
SPOOL OFF

-- Export fact_enrollments
SPOOL fact_enrollments.csv
SELECT * FROM fact_enrollments;
SPOOL OFF

-- Reset settings
SET MARKUP CSV OFF
SET PAGESIZE 14
SET FEEDBACK ON
```

!!! warning "SET MARKUP CSV ON"
    This requires Oracle 12.2 or later. If you get an error, use the manual approach below.

### Manual CSV (Older Oracle Versions)

If `SET MARKUP CSV ON` isn't available:

```sql
SET PAGESIZE 0
SET COLSEP ','
SET TRIMSPOOL ON
SET LINESIZE 32767
SET FEEDBACK OFF
SET HEADING ON

SPOOL dim_student.csv
SELECT snum || ',' || sname || ',' || standing || ',' || majorid || ',' || gpa
FROM dim_student;
SPOOL OFF
```

---

## Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| `ORA-12541: TNS:no listener` | Wrong host or port | Double-check your RDS endpoint and port |
| `ORA-01017: invalid username/password` | Wrong credentials | Verify username and password (case-sensitive) |
| `DPI-1047: Cannot locate a 64-bit Oracle Client library` | Oracle Instant Client not installed | Install Oracle Instant Client or use `oracledb` thin mode (default) |
| CSV has extra blank lines | SQL*Plus adds linefeeds | Add `SET TRIMSPOOL ON` and `SET PAGESIZE 0` |
| Numbers in quotes | CSV writer quoting | In Python: `csv.writer(f, quoting=csv.QUOTE_MINIMAL)` |
| Date format wrong | Oracle default date format | Use `TO_CHAR(load_date, 'YYYY-MM-DD')` in your SELECT |
| Special characters garbled | Encoding mismatch | Export with `encoding="utf-8"` and upload to Databricks as UTF-8 |

!!! tip "Verify your CSV"
    Before uploading to Databricks, open the CSV in a text editor (not Excel — it can mangle data). Check that:

    - First row has column names
    - No extra header/footer lines from SQL*Plus
    - Numbers aren't quoted
    - No trailing commas
