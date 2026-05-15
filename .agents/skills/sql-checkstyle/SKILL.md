# SQL Checkstyle Skill

This skill reviews SQL migration files and query definitions in the OpenMetadata project for style, correctness, and best practices.

## Trigger

Use this skill when:
- Reviewing `.sql` files under `bootstrap/sql/migrations/`
- Reviewing Flyway migration scripts
- Reviewing any raw SQL queries embedded in Java or Python source files
- A PR modifies database schema or data migration logic

## Checklist

### Naming Conventions
- [ ] Table names use `snake_case` and are plural (e.g., `entity_relationships`, not `EntityRelationship`)
- [ ] Column names use `snake_case`
- [ ] Index names follow the pattern `idx_<table>_<column(s)>` (e.g., `idx_entity_relationships_from_id`)
- [ ] Foreign key constraint names follow `fk_<table>_<referenced_table>`
- [ ] Migration file names follow Flyway versioning: `v<version>__<description>.sql` (e.g., `v1.3.0__add_domain_entity.sql`)

### SQL Style
- [ ] Keywords are UPPERCASE (`SELECT`, `FROM`, `WHERE`, `JOIN`, `INSERT`, `UPDATE`, `DELETE`, `CREATE`, `ALTER`, `DROP`)
- [ ] Each clause starts on a new line for multi-line queries
- [ ] Indentation uses 4 spaces (no tabs)
- [ ] Trailing whitespace is absent
- [ ] Statements end with a semicolon (`;`)
- [ ] No `SELECT *` in production queries — columns must be explicitly named

### Schema Design
- [ ] Every table has a primary key defined
- [ ] `id` columns use `VARCHAR(36)` for UUIDs or `BIGINT AUTO_INCREMENT` for surrogate keys — be consistent with existing tables
- [ ] Timestamp columns (`created_at`, `updated_at`) use `DATETIME(6)` for MySQL or `TIMESTAMP WITH TIME ZONE` for Postgres
- [ ] JSON/JSONB columns are used only when the schema is genuinely variable; prefer normalized columns otherwise
- [ ] `NOT NULL` constraints are explicit where nullability is not intended
- [ ] Default values are set for boolean and timestamp columns where applicable

### Migration Safety
- [ ] Migrations are **additive-only** where possible (add columns/tables, avoid drops in the same release)
- [ ] Column drops or renames are preceded by a deprecation migration in a prior release
- [ ] `ALTER TABLE ... ADD COLUMN` statements include a `DEFAULT` value if the column is `NOT NULL` (required for zero-downtime deployments)
- [ ] No DML (`INSERT`/`UPDATE`/`DELETE`) that could cause long table locks on large datasets without a batching strategy
- [ ] Migrations are idempotent where feasible (use `IF NOT EXISTS`, `IF EXISTS` guards)
- [ ] Both MySQL and PostgreSQL dialects are handled if the migration targets both engines (separate files under `mysql/` and `postgresql/` subdirectories)

### Indexes & Performance
- [ ] Foreign key columns have a corresponding index
- [ ] Columns used in frequent `WHERE` or `JOIN` clauses are indexed
- [ ] Composite indexes list the highest-cardinality column first
- [ ] No redundant indexes (e.g., an index on `(a)` when `(a, b)` already exists and covers it)

### Security
- [ ] No hardcoded credentials, tokens, or secrets in SQL files
- [ ] User-facing string inputs referenced in inline queries are parameterized (no string concatenation)

### Comments & Documentation
- [ ] Complex migrations include a comment block at the top explaining the purpose and any manual steps required
- [ ] Non-obvious column definitions have an inline comment
- [ ] Example comment block:
  ```sql
  -- Migration: v1.4.0__add_data_product_entity.sql
  -- Purpose: Introduces the data_product table to support the Data Product domain feature.
  -- Affects: data_product, entity_relationship
  -- Manual steps: None — fully automated.
  ```

## Common Issues to Flag

| Issue | Severity | Example |
|---|---|---|
| Missing semicolon | Error | `ALTER TABLE foo ADD COLUMN bar VARCHAR(256)` |
| `SELECT *` in view or stored query | Warning | `SELECT * FROM entity_extension` |
| Drop column without prior deprecation | Error | `ALTER TABLE thread_entity DROP COLUMN reactions` |
| Non-nullable column added without DEFAULT | Error | `ADD COLUMN owner_id VARCHAR(36) NOT NULL` |
| Mixed-case keywords | Warning | `select id from entity_relationship` |
| Missing index on FK column | Warning | FK `user_id` added but no `CREATE INDEX` |
| Hardcoded schema name | Warning | `SELECT * FROM openmetadata_db.entity_relationship` |

## References

- [OpenMetadata Bootstrap SQL directory](../../bootstrap/sql/migrations/)
- [Flyway versioned migrations docs](https://documentation.red-gate.com/fd/versioned-migrations-184127470.html)
- [MySQL ALTER TABLE online DDL](https://dev.mysql.com/doc/refman/8.0/en/innodb-online-ddl-operations.html)
- [PostgreSQL zero-downtime migrations](https://gocardless.com/blog/zero-downtime-postgres-migrations-the-hard-parts/)
