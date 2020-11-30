# DB

## [PG] Differencial sync using only SQL

Requirements:
- **Identifiable rows**: used to determine an item exists or not
- **Data rows**: used to determine an item has changed or not

Setup:
- **REF_TABLE**: the table holding your data at time T-1
- **SYNC_TABLE** (can be temporary): the table holding the data you want to compare with (data at time T)

### Deletion
```sql
WITH to_delete AS (
    SELECT [id(s)]
    FROM REF_TABLE
        EXCEPT
    SELECT [id(s)]
    FROM SYNC_TABLE
)
UPDATE REF_TABLE -- or DELETE/whatever
SET deleted = true
FROM to_delete
WHERE REF_TABLE.[id] = to_delete.[id]
```

*JOIN accordingly if requiering more data to evaluate the rows to delete*

### Addition
```sql
WITH to_add AS (
    SELECT [id(s)]
    FROM SYNC_TABLE
        EXCEPT
    SELECT [id(s)]
    FROM REF_TABLE
),
     -- Find the insertable data back from the ids to add
     insertable AS (
         SELECT SYNC_TABLE.*
         FROM SYNC_TABLE
                  JOIN to_add
                       ON to_add.[id] = SYNC_TABLE.[id]
     )
INSERT
INTO REF_TABLE ([cols])
SELECT [cols]
FROM insertable
```

*JOIN and add CTE accordingly if requiering more data or steps to make the final INSERT*

### Changes
```sql
WITH both_existing AS (
    SELECT [id(s)]
    FROM SYNC_TABLE
    INTERSECT
    SELECT [id(s)]
    FROM REF_TABLE
),
     to_update AS (
         SELECT [data_cols]
         FROM both_existing
                  JOIN SYNC_TABLE
                       ON both_existing.[ids] = SYNC_TABLE.[ids]
    EXCEPT
         SELECT [data_cols]
         FROM both_existing
                  JOIN REF_TABLE
                       ON both_existing.[ids] = REF_TABLE.[ids]
     ),
     updateable AS (
         -- Add the id from the REF_TABLE in the previously found changed stuff
         SELECT REF_TABLE.id, to_update.*
         FROM REF_TABLE
                  JOIN to_update
                       ON to_update.[ids] = REF_TABLE.[ids]
     )
UPDATE REF_TABLE
SET
    [data_cols] = updateable.[data_cols]
FROM updateable
WHERE REF_TABLE.[id] = updateable.[id]
```

*JOIN and add CTE accordingly if requiering more data or steps to make the final UPDATE*



## [PG] Assign each line of a query a random entry of a list

The list is smaller than the query.

```sql
WITH list AS (
   SELECT *, row_number() over () as rn
   FROM (VALUES ('item1'), ('item2'), ('item3')) as cr (id)
), list_count AS (
   SELECT count(*) AS 
   FROM list
),
random_base AS (
   SELECT base.*, (row_number() OVER () % list_count.c) + 1 AS rn
   FROM base, list_count
)
SELECT *
FROM random_base JOIN list USING (rn)
```

## [PG] Deduplication

The list is smaller than the query.

```sql
DELETE FROM data USING (
    SELECT MIN(ctid) as ctid, key
    FROM data
    GROUP BY key HAVING COUNT(*) > 1
) dupes
WHERE data.key = dupes.key
AND data.ctid != dupes.ctid
```

## Polymorphism

```
   +------------------------+      +----------------------+
   | stuff                  |      | stuff_one            |
   +------------------------+      +----------------------+
   | id                     <---+  | id                   |
   | [generic stuff fields] |   +--+ #stuff_id            |
+--+ #stuff_type_id         |   |  | [stuff_one fields]   |
|  +------------------------+   |  +----------------------+
|                               |
|  +------------+               |  +----------------------+
|  | stuff_type |               |  | stuff_two            |
|  +------------+               |  +----------------------+
+--> id         |               |  | id                   |
   | name       |               +--+ #stuff_id            |
   +------------+                  | [stuff_two fields]   |
                                   +----------------------+
```
