# ADR-001 · The application migrations own the schema

- **Status:** accepted
- **Context:** the `infra` repository starts a database engine. Someone could put DDL in a container startup script and someone else in the
  application migrations; with two owners, no one knows which schema runs in production.
- **Decision:** **all DDL lives in the application migrations** (tables, indexes, constraints, and category seed data).
  `infra` creates only the empty engine, user, and volume.
- **Rejected alternatives:** SQL scripts in `infra/db/init/` (a second source of truth); ORM in “create tables on startup” mode
  (leaves no history or rollback).
- **Consequences:** a Postgres extension or new index is created in the **same** migration that uses it. A schema changed manually in
  the engine is a defect, not a variant.
- **Verification:** `infra` contains no `CREATE TABLE`; the queries in `data-model.md` §10 match the migrations.
