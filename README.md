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
