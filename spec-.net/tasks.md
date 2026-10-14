# Tasks — Manager Library (.NET)

Each task is complete when its **evidence** exists and can be executed (Article XII). Initial status for all tasks: **pending**.

| # | Task | Depends on | Evidence |
|---|---|---|---|
| T-01 | Project skeleton and `GET /health` | — | `GET /health` returns `{"status":"ok"}`; the container starts with one command |
| T-02 | Initial migration: six tables, FKs, unique indexes, and the five seeded categories | T-01 | The queries in `data-model.md` §10 return the documented results; no column has a `DEFAULT` |
| T-03 | Domain: `Isbn`, `Quantity`, `DateRange` (value objects) | T-01 | Unit tests: 13-digit ISBN, quantity > 0, end ≥ start |
| T-04 | Domain: `Book` (`Withdraw`, `Restock`, soft delete) | T-03 | Tests: never < 0, never > total |
| T-05 | Domain: `Loan` (`AddItem`, `ReturnItem`, calculated status, 14 days, max. 3) | T-04 | Tests with a fake `Clock`: `open`, `overdue`, `closed` |
| T-06 | Domain: `Member` and `User` (name normalization, valid role) | T-03 | `"Ana "` becomes `ana`; `admin` role rejected through API |
| T-07 | **Authentication** ports and use cases (`LoginService`, `CreateLibrarianService`) with fakes | T-06 | HU-01 and HU-02 pass against fakes |
| T-08 | **Books** ports and use cases (create, search, edit, soft delete, cover) with fakes | T-04 | HU-03 and HU-04 pass against fakes |
| T-09 | **Lend** use case (`LendBooksService`) with fakes | T-05, T-08 | HU-06 complete: limits, overdue items, no available copies |
| T-10 | **Return** and **overdue items** use cases | T-09 | HU-07 and HU-09; returning twice → `ITEM_ALREADY_RETURNED` |
| T-11 | **Most-borrowed report** use case and its read port | T-09 | HU-10; the old title is preserved after renaming (ADR-004) |
| T-12 | Persistence adapter: book, member, and user repositories | T-02, T-08 | Integration tests against real PostgreSQL |
| T-13 | Persistence adapter: loan repository + version token | T-12, T-09 | Two loans for the last copy: one 201, one 409 (RNF-2) |
| T-14 | Security adapter: password hashing and JWT | T-07 | The password never appears in logs or responses (RNF-3) |
| T-15 | Cover adapter (disk) | T-08 | Uploading a 2 MB PNG succeeds; 3 MB → 422; no cover → `cover_key` is null |
| T-16 | HTTP adapter: routes in [`api-contract.md`](api-contract.md), single error body, role-based authorization | T-10, T-11, T-14, T-15 | Contract test exercises all routes and all status codes in §2 |
| T-17 | Composition and startup (initial admin from environment) | T-16 | System starts with an empty database and allows signing in as admin |
| T-18 | Move **domain-only** invariants into the database: `ck_book_total_positive`, `ck_user_role` | T-12 | An `INSERT` via `psql` that violates them fails; `data-model.md` §4 is updated |
| T-19 | Architecture tests (Articles I, II, and III) | T-17 | NetArchTest passes and fails if a prohibited dependency is introduced |
| T-20 | `infra`: `docker compose up` starts an empty database engine, API, and frontend | T-17 | One command; the database starts without tables until migrations are run |
| T-21 | `tool`: demonstration seeder **through the API** (≥ 40 books, 10 members, 15 loans, 3 overdue items) | T-17 | After it runs, `/loans/overdue` returns 3 items and the report has rows |
| T-22 | `app`: login, catalog, members, lend, return, overdue items, report | T-16 | End-to-end manual walkthrough at the fair |
| T-23 | `page`: static public presentation site | — | Opens without the API; works on mobile and desktop |

## Test completion definition

Anyone who reaches T-17 with evidence has completed the **core**. T-18 through T-23 distinguish good work from outstanding work. Performance
is evaluated using SDD, not by the number of tasks: one task with evidence is worth more than three half-finished tasks.
