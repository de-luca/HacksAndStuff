# DB

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
