# POSTGRESQL


## FUNCTIONS
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
