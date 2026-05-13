# Big Data and NoSQL

Big Data and NoSQL questions are usually conceptual. The goal is to decide what kind of data problem you are facing and which model fits it.

## The Five V's

| V | Question To Ask |
|---|---|
| Volume | Is there too much data for ordinary tools? |
| Velocity | Is data arriving too quickly? |
| Variety | Are there many formats, such as tables, JSON, logs, text, or media? |
| Veracity | Is the data incomplete, inconsistent, noisy, or uncertain? |
| Value | Can the organization turn this data into useful decisions? |

## NoSQL Category Selection

| Need | Best Fit |
|---|---|
| Fast lookup by ID | Key-value store |
| Flexible JSON-like records | Document database |
| Large distributed event-style data | Wide-column store |
| Relationship traversal | Graph database |

!!! tip "Video: NoSQL Category Comparison"
    <video controls preload="metadata" width="100%">
      <source src="../../assets/videos/nosql_category_comparison.mp4" type="video/mp4">
      Your browser does not support the video tag.
    </video>


!!! tip "Video: Relational Rows vs Document Model"
    <video controls preload="metadata" width="100%">
      <source src="../../assets/videos/nosql_relational_vs_document.mp4" type="video/mp4">
      Your browser does not support the video tag.
    </video>

## Compact Guide Section

# 4. Big Data and NoSQL

## 4.1 Big Data

Big Data refers to data that is too large, fast, varied, or messy for traditional tools to handle easily.

The Five V's:

| V | Meaning | Example |
|---|---|---|
| Volume | Very large amount of data | Billions of clickstream records |
| Velocity | Data arrives quickly | Sensor readings every second |
| Variety | Many formats | Tables, JSON, images, logs |
| Veracity | Data quality uncertainty | Duplicate, incomplete, or noisy data |
| Value | Data must support a useful goal | Predicting demand or detecting fraud |

## 4.2 Schema-on-Write vs Schema-on-Read

Schema-on-write:

- Structure is defined before data is stored.
- Typical relational database approach.
- Good when data is well understood and consistency is important.

Schema-on-read:

- Data is stored first.
- Structure is interpreted later when reading or analyzing it.
- Common with semi-structured formats such as JSON or XML.

## 4.3 NoSQL Overview

NoSQL means "Not Only SQL." It refers to database systems that are not based mainly on the relational model.

Common NoSQL motivations:

- Scale out across many machines.
- Handle flexible or changing data structures.
- Work naturally with cloud environments.
- Support schema-on-read or semi-structured data.

Important tradeoff:

- Traditional relational systems emphasize ACID consistency.
- Many NoSQL systems emphasize BASE: basically available, soft state, eventually consistent.

## 4.4 NoSQL Categories

| Type | Data Model | Good For | Example |
|---|---|---|---|
| Key-value | Key points to a value | Fast lookup by key | Redis, DynamoDB style access |
| Document | Key points to structured document | JSON-like records with flexible fields | MongoDB |
| Wide-column | Rows with column families | Large distributed datasets | Cassandra, HBase |
| Graph | Nodes and relationships | Relationship-heavy questions | Neo4j |

## 4.5 Relational vs NoSQL Decision Examples

Choose relational when:

- Data is highly structured.
- Relationships and constraints are stable.
- Transactions and consistency are central.
- SQL reporting is important.

Choose NoSQL when:

- The data structure changes often.
- The system needs massive horizontal scale.
- The main access pattern is key lookup or graph traversal.
- Semi-structured documents are natural for the problem.

Examples:

| Scenario | Better Fit | Why |
|---|---|---|
| Payroll system | Relational | Strong consistency and structured records matter. |
| Product catalog with flexible attributes | Document database | Products may have different fields. |
| User session cache | Key-value database | Fast lookup by session ID. |
| Friend-of-friend recommendations | Graph database | Connections are the main question. |
| IoT sensor stream | Big data / NoSQL tools | Velocity and volume may overwhelm one relational server. |
