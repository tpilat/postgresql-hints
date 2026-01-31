# POSTGRESQL


## FUNCTIONS
**Generate UUID**
```SQL
uuid_generate_v4()
```


## COLUMNS
**ADD COLUMN**
```SQL
ALTER TABLE schema."Table"
ADD COLUMN "ColumnName" integer NULL;
```


**SET COLUMN AS NOT NULL**
```SQL
ALTER TABLE schema."Table"
ALTER COLUMN "ColumnName" SET NOT NULL;
```


**CHANGE COLUMN TYPE**
```SQL
ALTER TABLE schema."Table"
ALTER COLUMN "ColumnName" TYPE varchar(511);
```


**COLUMN SET DEFAULT VALUE**
```SQL
ALTER TABLE schema."Table"
ALTER COLUMN "ColumnName" SET DEFAULT NEXTVAL(('schema."sequence_name"'::text)::regclass);
```


**DROP COLUMN**
```SQL
ALTER TABLE schema."Table"
DROP COLUMN "ColumnName";
```


**RENAME COLUMN**
```SQL
ALTER TABLE schema."Table"
RENAME COLUMN "OldColumnName" TO "NewColumnName";
```


## SEQUENCES
**CREATE SEQUENCES**
```SQL
CREATE SEQUENCE schema."sequence_name" INCREMENT 1 START 1;
```


## CONSTRAINTS
**DROP CONSTRAINT**
```SQL
ALTER TABLE schema."Table"
DROP CONSTRAINT IF EXISTS "ConstraintName";
```


## INDEXES
**CREATE INDEX**
```SQL
CREATE INDEX IF NOT EXISTS "IX_TableName_ColumnName1" ON schema."Table" ("ColumnName1");
```


**CREATE GIN INDEX**
```SQL
CREATE EXTENSION pg_trgm;
CREATE INDEX "IX_TABLE_COLUMN" ON schema."Table" using gin ("ColumnName" gin_trgm_ops);
```


**READ ALL INDEXES WITH COLUMN TYPES**
```SQL
SELECT
				U.usename AS user_name,
				tnsp.nspname AS table_schema,
				trel.relname AS table_name,
				irel.relname AS index_name,
				a.attname AS column_name,
				col.DATA_TYPE,
				i.indisunique AS is_unique,
				i.indisprimary AS is_primary,  
				(i.indexprs IS NOT NULL) OR(i.indkey::int[] @> array[0]) AS is_functional,
				i.indpred IS NOT NULL AS is_partial				
FROM pg_index AS i
				JOIN pg_class AS trel ON trel.oid = i.indrelid
				JOIN pg_namespace AS tnsp ON trel.relnamespace = tnsp.oid
				JOIN pg_class AS irel ON irel.oid = i.indexrelid
				CROSS JOIN LATERAL unnest(i.indkey) WITH ORDINALITY AS c(colnum, ordinality)
				LEFT JOIN LATERAL unnest(i.indoption) WITH ORDINALITY AS o(option, ordinality) ON c.ordinality = o.ordinality
				JOIN pg_attribute AS a ON trel.oid = a.attrelid AND a.attnum = c.colnum
				JOIN pg_user AS U ON trel.relowner = U.usesysid
				LEFT JOIN information_schema."columns" col ON col.table_schema = tnsp.nspname AND col.table_name = trel.relname AND col.column_name = a.attname
WHERE --i.indisunique <> TRUE AND i.indisprimary <> TRUE AND
				NOT nspname LIKE 'pg%' --Excluding system tables
				AND col.DATA_TYPE <> 'uuid'
				AND col.DATA_TYPE <> 'timestamp without time zone'
				AND col.DATA_TYPE <> 'integer'
				AND col.DATA_TYPE <> 'bigint'
ORDER BY U.usename,
				tnsp.nspname,
				trel.relname,
				irel.relname,
				a.attname

```



## SCHEMAS
**DROP AND CREATE SCHEMA**
```SQL
DROP Schema If Exists schemaname Cascade;
CREATE Schema schemaname;
```


## CREATE
**CREATE TABLE AS SELECT FROM**
```SQL
CREATE TABLE schema."NewTableName" AS
SELECT "ColumnName1", "ColumnName2", "ColumnName3"
FROM schema2."Table"
WHERE ...
```


## INSERT
**INSERT INTO ... SELECT FROM**
```SQL
INSERT INTO mbs."RolePermission"
  ("...", "...", "...")
SELECT "ColumnName1", "ColumnName2", "ColumnName3"
FROM schema."Table"
WHERE ...
```


**INSERT INTO ... ON CONFLICT**
```SQL
INSERT INTO mbs."RolePermission"
  ("...", "...", "...")
VALUES
  ('...', '...', '...')
ON CONFLICT ("ColumnName1", "ColumnName2") DO NOTHING;
```


**INSERT INTO ... SELECT FROM ... ON CONFLICT**
```SQL
INSERT INTO mbs."RolePermission"
  ("...", "...", "...")
SELECT "ColumnName1", "ColumnName2", "ColumnName3"
FROM schema."Table"
WHERE ...
ON CONFLICT ("ColumnName1", "ColumnName2") DO NOTHING;
```


## UPDATE
**UPDATE ... FROM, JOIN**
```SQL
UPDATE schema."Table" u
SET u."ColumnName1" = t2."ColumnName2"
FROM schema."Table" t1
JOIN schema."Table2" t2
WHERE ...
```


## USERS, PERMISSIONS
**CREATE USER ... GRANT SELECT**
```SQL
DROP USER IF EXISTS myuser;
CREATE USER myuser WITH PASSWORD 'myPassword' NoCreateDB; 

GRANT USAGE ON SCHEMA doc TO myuser;
GRANT SELECT ON ALL TABLES IN SCHEMA doc TO myuser;

...

GRANT USAGE ON SCHEMA des To myuser;
GRANT select, insert, update, delete On All Tables In Schema des To myuser;
GRANT usage On All Sequences In Schema des To myuser;
```

**GET ALL GRANTED PERMISSIONS**
```SQL
SELECT 
	*
FROM 
    information_schema.role_table_grants;
```


## QUERY PARAMETERS
**WITH, TABLE**
```SQL
WITH
    constant_1_str AS (VALUES ('Hello World')),
    constant_2_int AS (VALUES (100))
SELECT *
FROM some_table
WHERE column8 = (table constant_1_str)
LIMIT (table constant_2_int)
```

**ONE ROW WITH**
```SQL
WITH DATUMY AS (
SELECT
    '2023-01-01 00:00:00'::timestamp AS "DATUM_OD",
    '2023-12-31 23:59:59'::timestamp AS "DATUM_DO"
)
SELECT s."BusinessName" AS "SPV", s."BusinessIdentificationNumber" AS "ICO", COUNT(id."IdIncomingDocument") AS "PocetDorucenychZasielok"
FROM sub."Subject" s
LEFT JOIN reg."IncomingDocument" id ON s."IdSubject" = id."IdOwnerLegalSubject", DATUMY
WHERE s."IdTenant" IS NOT NULL AND s."IdParent" IS NOT NULL
	AND (id."AuditCreatedUtc" IS NULL
		OR DATUMY."DATUM_OD" <= (id."AuditCreatedUtc" at time zone 'utc' at time zone 'Europe/Bratislava') AND (id."AuditCreatedUtc" at time zone 'utc' at time zone 'Europe/Bratislava') <= DATUMY."DATUM_DO")
GROUP BY s."BusinessName", s."BusinessIdentificationNumber";
```


## REPEAT
**FOR i**
```SQL
do
$$
declare 
  i record;
begin
  for i in 1..3 loop
    Insert into employee values(1,'Mike');
  end loop;
end;
$$
;
```


## PostgreSQL CONNECTIONS
**SELECT ALL CONNECTIONS**
```
SELECT 
    pid, 
    usename AS username, 
    datname AS database_name, 
    client_addr AS client_address, 
    client_port, 
    application_name, 
    state, 
    backend_start AS connection_start_time, 
    query AS current_query,
    backend_type,
    wait_event
FROM 
    pg_stat_activity
ORDER BY 
    backend_start;
```


**BLOCKED COMMAND vs BLOCKING COMMANDS**
```
SELECT
  blocked.pid     AS blocked_pid,
  blocked.query   AS blocked_query,
  blocking.pid    AS blocking_pid,
  blocking.query  AS blocking_query
FROM pg_stat_activity blocked
JOIN pg_locks blocked_locks ON blocked_locks.pid = blocked.pid AND NOT blocked_locks.granted
JOIN pg_locks blocking_locks ON blocking_locks.locktype = blocked_locks.locktype
  AND blocking_locks.database IS NOT DISTINCT FROM blocked_locks.database
  AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
  AND blocking_locks.page IS NOT DISTINCT FROM blocked_locks.page
  AND blocking_locks.tuple IS NOT DISTINCT FROM blocked_locks.tuple
  AND blocking_locks.virtualxid IS NOT DISTINCT FROM blocked_locks.virtualxid
  AND blocking_locks.transactionid IS NOT DISTINCT FROM blocked_locks.transactionid
  AND blocking_locks.classid IS NOT DISTINCT FROM blocked_locks.classid
  AND blocking_locks.objid IS NOT DISTINCT FROM blocked_locks.objid
  AND blocking_locks.objsubid IS NOT DISTINCT FROM blocked_locks.objsubid
  AND blocking_locks.pid != blocked_locks.pid
JOIN pg_stat_activity blocking ON blocking.pid = blocking_locks.pid;
```


## PostgreSQL Database SIZEs
**SELECT ALL DATABASE SIZEs**
```
SELECT
    d.datname                                  AS database,
    pg_catalog.pg_get_userbyid(d.datdba)       AS owner,
    t.spcname                                  AS tablespace,
    pg_database_size(d.datname)                AS size_bytes,
    pg_size_pretty(pg_database_size(d.datname)) AS size_pretty
FROM pg_database d
LEFT JOIN pg_tablespace t ON t.oid = d.dattablespace
ORDER BY pg_database_size(d.datname) DESC;
```


**SELECT ALL TABLE SIZEs**
```
SELECT
    n.nspname                                AS schema,
    c.relname                                AS table_name,
    pg_size_pretty(pg_total_relation_size(c.oid)
        - pg_relation_size(c.oid)
        - pg_indexes_size(c.oid))            AS toast_other_pretty,
    pg_size_pretty(pg_total_relation_size(c.oid)) AS total_pretty,
    pg_size_pretty(pg_relation_size(c.oid))  AS heap_pretty,
    pg_size_pretty(pg_indexes_size(c.oid))   AS indexes_pretty,
    c.reltuples::bigint                      AS approx_rows,                -- približný počet riadkov (po ANALYZE)
    pg_relation_size(c.oid)                  AS heap_bytes,                 -- samotná tabuľka (bez indexov)
    pg_indexes_size(c.oid)                   AS indexes_bytes,              -- všetky indexy tabuľky
    (pg_total_relation_size(c.oid)
        - pg_relation_size(c.oid)
        - pg_indexes_size(c.oid))            AS toast_other_bytes,          -- TOAST + VM/FSM
    pg_total_relation_size(c.oid)            AS total_bytes                -- spolu
FROM pg_class c
JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE
    c.relkind = 'r'                                  -- bežné (fyzické) tabuľky; zahŕňa aj partície
    AND n.nspname NOT IN ('pg_catalog', 'information_schema')
    AND n.nspname NOT LIKE 'pg_toast%'
    AND n.nspname NOT LIKE 'pg_temp_%'
ORDER BY pg_total_relation_size(c.oid) DESC;
```



**SELECT ALL INDEX SIZEs**
```
SELECT
  n.nspname                                AS schema,
  t.relname                                AS table_name,
  i.relname                                AS index_name,
  pg_am.amname                             AS index_type,        -- btree, gin, gist, ...
  pg_relation_size(i.oid)                  AS size_bytes,
  pg_size_pretty(pg_relation_size(i.oid))  AS size_pretty,
  s.idx_scan                               AS scans,             -- koľko-krát sa index použil (od posledného resetu štatistík)
  s.idx_tup_read, s.idx_tup_fetch
FROM pg_class       AS i                   -- index
JOIN pg_namespace   AS n ON n.oid = i.relnamespace
JOIN pg_index       AS x ON x.indexrelid = i.oid
JOIN pg_class       AS t ON t.oid = x.indrelid
LEFT JOIN pg_stat_all_indexes AS s ON s.indexrelid = i.oid
LEFT JOIN pg_am ON pg_am.oid = i.relam
WHERE i.relkind IN ('i','I')               -- 'i' = bežný index, 'I' = parent partic. index (zvyčajne 0B)
	AND n.nspname NOT IN ('pg_catalog', 'information_schema')
    AND n.nspname NOT LIKE 'pg_toast%'
    AND n.nspname NOT LIKE 'pg_temp_%'
ORDER BY pg_relation_size(i.oid) DESC;
```
