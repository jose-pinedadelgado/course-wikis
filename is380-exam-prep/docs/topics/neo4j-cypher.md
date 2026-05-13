# Neo4j and Cypher

Neo4j stores data as a graph: nodes connected by relationships. Cypher queries look like the graph pattern you want to find.

## Read A Pattern

```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
RETURN p.name, m.title;
```

Read this as: find `Person` nodes connected by an outgoing `ACTED_IN` relationship to `Movie` nodes.

## Pattern Symbols

| Symbol | Meaning |
|---|---|
| `(p:Person)` | A node with variable `p` and label `Person`. |
| `[:ACTED_IN]` | A relationship with type `ACTED_IN`. |
| `->` | Direction of the relationship. |
| `WHERE` | Filters matched nodes or relationships. |
| `RETURN` | Chooses what appears in the output. |

!!! tip "Video: Cypher Pattern Matching"
    <video controls preload="metadata" width="100%">
      <source src="../../assets/videos/neo4j_cypher_pattern_matching.mp4" type="video/mp4">
      Your browser does not support the video tag.
    </video>


!!! tip "Video: Graph Traversal vs SQL Joins"
    <video controls preload="metadata" width="100%">
      <source src="../../assets/videos/neo4j_traversal_vs_sql_join.mp4" type="video/mp4">
      Your browser does not support the video tag.
    </video>

## Compact Guide Section

# 6. Neo4j and Cypher

## 6.1 What Neo4j Is

Neo4j is a graph database. Instead of storing data primarily as tables, it stores data as nodes and relationships.

Graph databases are useful when the relationships between things are as important as the things themselves.

Examples:

- People connected to friends.
- Actors connected to movies.
- Bank accounts connected to transactions.
- Cities connected by roads.

## 6.2 Property Graph Concepts

| Concept | Meaning | Example |
|---|---|---|
| Node | An entity or object | A person, movie, company |
| Label | A category for a node | `Person`, `Movie` |
| Relationship | A directed connection between nodes | `ACTED_IN`, `DIRECTED`, `KNOWS` |
| Property | A key-value pair on a node or relationship | `name: 'Tom Hanks'`, `released: 1994` |

Example graph idea:

```text
(Person: Tom Hanks)-[:ACTED_IN]->(Movie: Forrest Gump)
(Person: Robert Zemeckis)-[:DIRECTED]->(Movie: Forrest Gump)
```

## 6.3 Reading Data With MATCH

`MATCH` describes the graph pattern Neo4j should find.

Find all people:

```cypher
MATCH (p:Person)
RETURN p;
```

Find movie titles:

```cypher
MATCH (m:Movie)
RETURN m.title;
```

Find actors and movies:

```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
RETURN p.name, m.title;
```

Find directors of movies after 2000:

```cypher
MATCH (p:Person)-[:DIRECTED]->(m:Movie)
WHERE m.released > 2000
RETURN p.name, m.title, m.released;
```

## 6.4 Understanding Cypher Patterns

Pattern:

```cypher
(actor:Person)-[:ACTED_IN]->(movie:Movie)<-[:DIRECTED]-(director:Person)
```

Read it as:

Find a `Person` node called `actor` who has an outgoing `ACTED_IN` relationship to a `Movie` node called `movie`, and that same movie has an incoming `DIRECTED` relationship from another `Person` node called `director`.

This is why graph databases are useful: the query is shaped like the relationship pattern you want to find.

## 6.5 Neo4j vs SQL Joins

In SQL, related data is reconnected with joins:

```sql
SELECT p.person_name, m.movie_title
FROM people p
INNER JOIN acted_in ai
  ON p.person_id = ai.person_id
INNER JOIN movies m
  ON ai.movie_id = m.movie_id;
```

In Cypher, the relationship is part of the pattern:

```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
RETURN p.name, m.title;
```

Both can represent relationships. The difference is that graph databases make relationship traversal the center of the model.
