# Query Recognition

This page connects common request wording to the SQL technique it usually calls for. It is a recognition aid, not a solution page.

## Recognition Table

| Request Wording | Likely SQL Move |
|---|---|
| "Show rows from two related tables" | Join the tables using their key relationship. |
| "Keep rows even when there is no match" | Use an outer join. |
| "Compare a row to another row in the same table" | Use a self join with two aliases. |
| "Create a readable label or status" | Use `CASE`. |
| "Compare to one overall value" | Use a noncorrelated subquery. |
| "Compare each row to a value for its own group" | Use a correlated subquery. |
| "One row per category" | Use `GROUP BY`. |
| "Only keep groups that meet a condition" | Use `HAVING`. |
| "Reuse the same query later" | Use `CREATE VIEW`. |

## Query Writing Checklist

1. Circle the required output columns.
2. Identify the table that owns each column.
3. Draw the join path.
4. Decide whether unmatched rows must stay.
5. Decide the grain: one row per base row, one row per group, or one row per relationship.
6. Add calculations, aliases, and `CASE` labels only when requested.
7. Add sorting last.

## Partial Credit Strategy

Even when syntax is imperfect, a strong written answer should show:

- Correct tables.
- Correct join direction.
- Correct join condition.
- Correct aggregate level.
- Clear aliases.
- Output columns that match the request.
