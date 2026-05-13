# DynamoDB

DynamoDB design questions are access-pattern questions. Ask: **what key value does the application know when it asks for the data?**

## Core Terms

| Term | Meaning |
|---|---|
| Table | Collection of items. |
| Item | One record. Similar to a row, but flexible. |
| Attribute | A value inside an item. |
| Partition key | The required key used to locate items. |
| Sort key | Optional second key that orders related items under the same partition key. |
| Query | Efficient read using key values. |
| Scan | Reads the whole table, then filters. |

## Key Design Shortcut

If the app asks "show all songs by one artist sorted by title," a good key design is:

```text
partition key = Artist
sort key = SongTitle
```

!!! tip "Video: Query vs Scan"
    <video controls preload="metadata" width="100%">
      <source src="../../assets/videos/dynamodb_query_vs_scan.mp4" type="video/mp4">
      Your browser does not support the video tag.
    </video>


!!! tip "Video: Partition Key Plus Sort Key"
    <video controls preload="metadata" width="100%">
      <source src="../../assets/videos/dynamodb_partition_sort_key.mp4" type="video/mp4">
      Your browser does not support the video tag.
    </video>


!!! tip "Video: Item Collections"

    This reinforces how related items can be grouped by key and ordered by sort key.
    <video controls preload="metadata" width="100%">
      <source src="../../assets/videos/dynamodb_item_collections.mp4" type="video/mp4">
      Your browser does not support the video tag.
    </video>

## Compact Guide Section

# 5. DynamoDB

## 5.1 What DynamoDB Is

DynamoDB is a fully managed AWS NoSQL database service. It is designed for fast key-based access at scale without the user managing database servers.

Core terms:

| DynamoDB Term | Meaning |
|---|---|
| Table | A collection of items. |
| Item | A single record in a table. Similar to a row, but more flexible. |
| Attribute | A data value inside an item. Similar to a column, but items do not all need the same attributes. |
| Primary key | The value or values that uniquely identify an item. |
| Partition key | The hash key used to organize and locate items. |
| Sort key | Optional second key that stores related items in sorted order under the same partition key. |

## 5.2 Simple Primary Key

A table with only a partition key has a simple primary key.

Example:

```text
Table: People
Primary key: person_id
```

Each `person_id` must be unique.

## 5.3 Composite Primary Key

A table with a partition key and sort key has a composite primary key.

Example:

```text
Table: Music
Partition key: Artist
Sort key: SongTitle
```

This allows multiple songs by the same artist:

| Artist | SongTitle | Year |
|---|---|---|
| Drake | Best I Ever Had | 2010 |
| Drake | God's Plan | 2018 |
| Beyonce | Halo | 2008 |

The pair `(Artist, SongTitle)` must be unique. The partition key alone does not need to be unique when a sort key exists.

## 5.4 Query vs Scan

`Query`:

- Finds items using the primary key.
- Requires a partition key value.
- Can optionally use the sort key.
- Efficient because it uses indexed access.

`Scan`:

- Reads every item in a table.
- Can apply filters after reading.
- Less efficient for large tables.

Rule of thumb: design your table so important application questions can use `Query`, not `Scan`.

## 5.5 Read Consistency

Eventually consistent read:

- Default for many DynamoDB read operations.
- May not immediately reflect a very recent successful write.
- Repeating the read shortly after should return the updated result.
- Uses less read capacity than strongly consistent reads.

Strongly consistent read:

- Returns the most up-to-date committed data for supported reads.
- Useful when the application cannot tolerate reading stale data.

## 5.6 DynamoDB Design Checklist

When designing a DynamoDB table, ask:

1. What questions must the application answer quickly?
2. What exact key values will be known at query time?
3. What should the partition key be?
4. Do related items need a sort key?
5. Can the main access patterns use `Query`?
