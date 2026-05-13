# IS380 Exam Prep

This wiki is a web version of the IS380 study guide. It keeps the full guide intact, then expands each topic into separate pages with definitions, mental models, examples, and short visual explainers.

!!! note "How to use this site"
    Start with the **Full Guide** if you want the compact version. Use the topic pages when a concept still feels fuzzy or when you want more examples.

## Exam Prep Map

| Area | Start Here | What to Master |
|---|---|---|
| SQL foundations | [Foundations](topics/sql-foundations.md) | `SELECT`, `WHERE`, calculations, aggregates, grouping, `HAVING`, sorting |
| Advanced SQL | [Advanced SQL](topics/advanced-sql.md) | joins, self joins, subqueries, `CASE`, views |
| Query writing | [Query Recognition](topics/query-patterns.md) | recognize which SQL technique a request calls for |
| Physical design | [Physical Design](topics/physical-design.md) | denormalization, sequential access, indexes, hashed access |
| Big Data and NoSQL | [Big Data and NoSQL](topics/big-data-nosql.md) | Five V's, schema-on-read, NoSQL categories |
| DynamoDB | [DynamoDB](topics/dynamodb.md) | keys, Query vs Scan, consistency |
| Neo4j/Cypher | [Neo4j and Cypher](topics/neo4j-cypher.md) | nodes, relationships, `MATCH`, graph patterns |

## Videos Are Embedded

The topic pages now include the rendered Manim videos. The full library is listed on the [Videos](videos.md) page, and the MP4 files live in:

```text
docs/assets/videos/
```

## Recommended Study Order

1. Read the compact [Full Guide](full-guide.md).
2. Work through [SQL Foundations](topics/sql-foundations.md) and [Advanced SQL](topics/advanced-sql.md).
3. Review the query-recognition page to connect wording with SQL techniques.
4. Review the conceptual sections: physical design, NoSQL, DynamoDB, and Neo4j.
5. Rewatch the embedded videos for any concept that still feels fuzzy.
