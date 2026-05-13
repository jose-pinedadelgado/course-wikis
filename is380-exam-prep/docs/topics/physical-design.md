# Physical Design

Physical design asks how the database can store and retrieve data efficiently. For this exam-prep guide, the focus is intentionally narrow: denormalization and basic access methods.

## Definitions

| Term | Meaning |
|---|---|
| Sequential access | The DBMS reads rows one by one. |
| Indexed access | The DBMS uses a lookup structure to jump closer to matching rows. |
| Hashed access | A hash function maps a key value to a storage location. |
| Denormalization | Duplicating or combining data to make common reads faster. |

## Tradeoff To Remember

Indexes and denormalization can make reads faster, but they introduce costs:

- Indexes must be maintained during inserts, updates, and deletes.
- Denormalized data can become inconsistent if duplicated values are not updated everywhere.

!!! tip "Video: Sequential Scan vs Index Lookup"
    <video controls preload="metadata" width="100%">
      <source src="../../assets/videos/physical_scan_vs_index.mp4" type="video/mp4">
      Your browser does not support the video tag.
    </video>


!!! tip "Video: Denormalization Tradeoff"
    <video controls preload="metadata" width="100%">
      <source src="../../assets/videos/physical_denormalization_tradeoff.mp4" type="video/mp4">
      Your browser does not support the video tag.
    </video>


## Compact Guide Section

# 3. Physical Design

Physical design is about how the database is actually stored and accessed. Logical design asks, "What tables and relationships should exist?" Physical design asks, "How can the DBMS store and retrieve this efficiently?"

## 3.1 Denormalization

Denormalization means intentionally combining or duplicating data from normalized tables to improve performance for common reads.

Normalized design example:

- `devices(device_id, device_label, room_id)`
- `rooms(room_id, room_name, building)`

Denormalized reporting design example:

- `device_room_report(device_id, device_label, room_name, building)`

Benefit:

- Reports may run faster because fewer joins are needed.

Risk:

- Room data is duplicated across many rows.
- If room name or building changes, every duplicated copy must be updated correctly.

Use denormalization carefully. First normalize to get a sound structure, then denormalize only when there is a clear performance reason.

## 3.2 Sequential Access

Sequential access means the DBMS reads through rows one by one.

This can be acceptable when:

- The table is small.
- Most rows are needed.
- There is no useful index.

It can be expensive when:

- The table is large.
- Only a few rows are needed.
- The query is frequent.

## 3.3 Indexed Access

An index is like a lookup structure that helps the DBMS find rows faster.

Indexes are often useful on:

- Primary keys.
- Foreign keys used in joins.
- Columns often used in `WHERE`.
- Columns often used in `ORDER BY`.
- Columns often used in `GROUP BY`.

Tradeoff:

- Reads can be faster.
- Inserts, updates, and deletes can be slower because the index must also be maintained.
- Indexes take extra storage.

Do not index every column. Indexes are most useful when the column has enough variety and the query returns a relatively small portion of the table.

## 3.4 Hashed Access

Hashed access uses a hashing algorithm to map a key value to a storage location.

Example idea:

```text
device_id = D100 -> hash function -> storage location
```

This can make exact-key lookup very fast. It is less useful for range questions such as "all devices between ID D100 and D200" unless the DBMS has additional structures.
