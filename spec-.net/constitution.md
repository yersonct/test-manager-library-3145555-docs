# Constitution — Manager Library (.NET)

Non-negotiable project principles. These are not matters of style or preference: each article includes **how it is verified**, because a principle
no one can verify is only an intention. A change that violates an article is rejected without further discussion.

---

## Article I — The domain depends on nothing

`src/Library.Domain/` imports no other project module or third-party library: only the base library (`record`, `DateTimeOffset`); no EF Core, ASP.NET, or FluentValidation.

**Why:** this is the only way to test invariants without a database, server, or container, and to change ORMs without touching the rules.
**Verification:** NetArchTest prohibits the domain from depending on `application`, `adapters`, or `bootstrap`.

## Article II — The core declares its ports; the outside implements them

The application layer knows only the domain. Everything it needs from the outside (repositories, clock, password hashing, cover storage,
token generation) is expressed as a **port that it defines itself**. No framework or ORM type appears in a port signature.

**Verification:** NetArchTest prevents `application` from importing `adapters`. Application tests run with fakes and without referencing `adapters`.

## Article III — A single composition point

Only one module (`src/Library.Bootstrap/Composition/PortBindings.cs`) connects a port to its adapter. Infrastructure is not instantiated outside it.
**Verification:** no instance of an `adapters` class is created outside that module; checked with an architecture test.

## Article IV — SOLID is not decorative

| Principle | Concrete application | Verification |
|---|---|---|
| **SRP** | One application service = one use case. `LendBooksService` lends and does nothing else | A service with two reasons to change is split |
| **OCP** | Changing cover storage (disk → S3) does not touch the domain or application | The new adapter adds only one line to composition |
| **LSP** | Test fakes replace the real adapter without changing expectations | The same test passes against the fake and the real adapter |
| **ISP** | Small ports: `Clock`, `PasswordHasher`, `TokenIssuer`, `CoverStorage` | No implementer has unused methods |
| **DIP** | The consumer defines the port | Ports live in the application package |

## Article V — Migrations own the schema

The `infra` repository starts an **empty** database engine. All DDL (tables, indexes, constraints, extensions, and seed data) lives in the application's
migrations (EF Core migrations). See [ADR-001](adr/adr-001-schema-ownership.md).
**Verification:** `infra` contains no `CREATE TABLE`.

## Article VI — The database enforces every invariant it can

A rule that exists only in code protects the application, not the data. Every invariant expressible in the engine (`CHECK`, `UNIQUE`, FK)
is declared there. Anything not enforced is marked **pending** in [`data-model.md`](data-model.md); it is not promised.
**Verification:** the queries in `data-model.md` §10 match what the document declares.

## Article VII — Derived values are calculated, not stored

Loan status (`open`, `overdue`, `closed`) and days late are calculated. There is no column for them.
**Verification:** `data-model.md` lists no `status` or `days_late` column.

## Article VIII — Frozen values do not follow the catalog

The loan item stores the title, ISBN, and category as of the loan instant. No report reads them from the live catalog.
**Verification:** a test renames a book after it is loaned and checks that the report preserves the old title ([ADR-004](adr/adr-004-aggregated-frozen-report.md)).

## Article IX — No secret enters the repository or a log

No passwords, precomputed hashes, or token keys. All secrets come from the environment. The real `.env` is in `.gitignore`; only `.env.example` is versioned.
**Verification:** `gitleaks` or a pattern `grep` in CI; no HTTP response includes `password` or `password_hash`.

## Article X — The database wins

If the data document and the database disagree, the document is wrong and must be corrected.

## Article XI — Language

Code, columns, and routes are in English. Documentation and user-facing messages are in Spanish.

## Article XII — Nothing is assumed without evidence

A task in [`tasks.md`](tasks.md) is complete when its **evidence** exists and can be executed, not when “it has already been scheduled.”
