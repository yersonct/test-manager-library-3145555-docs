# Constitution — Manager Library (Python)

Non-negotiable project principles. These are not style or preferences: each article states **how it is verified**, because a principle
nobody can check is only an intention. A change that violates an article is rejected without considering the rest.

---

## Article I — The domain depends on nothing

`src/library/domain/` imports no other project module or third-party library: only the standard library (`dataclass(frozen=True)`, `datetime`, `typing`); no SQLAlchemy, FastAPI, or pydantic.

**Why:** this is the only way to test invariants without a database, server, or container, and to change the ORM without touching rules.
**Verification:** an `import-linter` contract and a test in `tests/architecture/` prohibit the domain from depending on `application`, `adapters`, or `bootstrap`.

## Article II — The core declares its ports; the outside implements them

The application layer knows only the domain. Everything it needs from the outside (repositories, clock, password hashing, cover storage,
token generation) is expressed as a **port that it defines itself**. No framework or ORM type appears in a port signature.

**Verification:** an `import-linter` contract and a test in `tests/architecture/` prevent `application` from importing `adapters`. Application tests run with fakes and without referencing `adapters`.

## Article III — A single composition point

A single module (`src/library/bootstrap/composition/port_bindings.py`) is where a port meets its adapter. Infrastructure is not instantiated anywhere else.
**Verification:** no instance of an `adapters` class exists outside that module; checked by an architecture test.

## Article IV — SOLID is not decorative

| Principle | Concrete application | Verification |
|---|---|---|
| **SRP** | One application service = one use case. `LendBooksService` lends and does nothing else | A service with two reasons to change is split |
| **OCP** | Changing cover storage (disk → S3) does not touch the domain or application | The new adapter only adds one line to composition |
| **LSP** | Test fakes replace the real adapter without changing expectations | The same test passes against the fake and the real adapter |
| **ISP** | Small ports: `Clock`, `PasswordHasher`, `TokenIssuer`, `CoverStorage` | No implementer leaves methods unused |
| **DIP** | The consumer defines the port | Ports live in the application package |

## Article V — Migrations own the schema

The `infra` repository starts an **empty** database engine. All DDL (tables, indexes, constraints, extensions, and seed data) lives in the application's migrations (Alembic revisions). See [ADR-001](adr/adr-001-schema-ownership.md).
**Verification:** `infra` contains no `CREATE TABLE`.

## Article VI — The database enforces the invariants it can

A rule that exists only in code protects the application, not the data. Every invariant expressible in the engine (`CHECK`, `UNIQUE`, FK)
is declared there. Anything not enforced is marked **pending** in [`data-model.md`](data-model.md); it is not promised.
**Verification:** the queries in `data-model.md` §10 match what the document declares.

## Article VII — Derived values are calculated, not stored

Loan status (`open`, `overdue`, `closed`) and days late are calculated. There are no columns for them.
**Verification:** `data-model.md` lists no `status` or `days_late` column.

## Article VIII — Snapshotted values do not follow the catalog

The loan item stores the title, ISBN, and category from the moment of the loan. No report reads them from the live catalog.
**Verification:** a test loans a book, renames it afterward, and confirms the report retains the old title ([ADR-004](adr/adr-004-aggregated-snapshotted-report.md)).

## Article IX — No secret enters the repository or a log

No passwords, precomputed hashes, or token keys. All secrets come from the environment. The real `.env` is in `.gitignore`; only `.env.example` is versioned.
**Verification:** `gitleaks` or a pattern `grep` in CI; no HTTP response includes `password` or `password_hash`.

## Article X — The database engine wins

If the data document and database disagree, the document is broken and must be corrected.

## Article XI — Language

Code, columns, and routes are in English. Documentation and user-facing messages are in Spanish.

## Article XII — Nothing is complete without evidence

A task in [`tasks.md`](tasks.md) is complete when its **evidence** exists and can be executed, not when it is "already scheduled."
