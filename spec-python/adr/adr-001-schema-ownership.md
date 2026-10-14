# ADR-001 · Application migrations own the schema

- **Status:** accepted
- **Context:** the `infra` repository starts a database engine. Someone could put DDL in a container startup script and someone else in application
  migrations; with two owners, nobody knows which schema is running in production.
- **Decision:** **all DDL lives in the application's migrations** (tables, indexes, constraints, and category seed data).
  `infra` only creates the empty engine, user, and volume.
- **Rejected alternatives:** SQL scripts in `infra/db/init/` (second source of truth); ORM in "create tables on startup" mode
  (no history or rollback).
- **Consequences:** a Postgres extension or new index is created in the **same** migration that uses it. A schema changed
  manually in the engine is a defect, not a variation.
- **Verification:** `infra` contains no `CREATE TABLE`; the queries in `data-model.md` §10 match the migrations.
