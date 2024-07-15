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


## SCHEMAS
**DROP AND CREATE SCHEMA**
```SQL
DROP Schema If Exists schemaname Cascade;
CREATE Schema schemaname;
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
