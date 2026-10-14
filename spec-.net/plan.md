# Technical Plan — Manager Library (.NET)

How to build what is described in [`spec.md`](spec.md) and [`constitution.md`](constitution.md). The schema is **not** described here: it lives in
[`data-model.md`](data-model.md). The system structure is in [`architecture.md`](architecture.md).

## 1. Technical decisions

| # | Decision | Choice (.NET) | Why |
|---|---|---|---|
| D-01 | Language and runtime | .NET 8 / C# 12 | Reference spec stack |
| D-02 | HTTP framework | ASP.NET Core (Minimal API) | Contract in [`api-contract.md`](api-contract.md) |
| D-03 | Persistence | EF Core 8 + EF migrations on PostgreSQL 16 | The schema belongs to the migrations (ADR-001) |
| D-04 | Concurrency | Per-book version token + range `CHECK` ([ADR-002](adr/adr-002-optimistic-concurrency.md)) | Last copy (RNF-2) |
| D-05 | Dates | UTC instants (`timestamptz`); the clock is a port (`Clock`) | Test overdue behavior without waiting 14 days |
| D-06 | Report | One aggregate query through a read port ([ADR-004](adr/adr-004-aggregated-frozen-report.md)) | Stable and inexpensive |
| D-07 | Value objects | `Isbn`, `Quantity`, `DocumentNumber`, `DateRange`: live within their owner's row, with no table | They have no identity |
| D-08 | Covers | Binary data in external storage (disk locally, S3 in production); the database stores only `cover_key` | Opaque key, never a path |
| D-09 | Passwords | BCrypt (factor 12) behind the `PasswordHasher` port; the domain never sees plaintext | Article IX |
| D-10 | Startup | The application creates the initial admin from `ADMIN_USERNAME` / `ADMIN_PASSWORD` | The database cannot produce the hash |
| D-11 | Authentication | JWT HS256, 8 hours, key in `JWT_SECRET` | Simple and stateless |
| D-12 | Tests | xUnit, with fakes for ports and a real database only for integration tests | Testing pyramid |

## 2. Environment configuration

| Variable | Required | Example |
|---|---|---|
| `DATABASE_URL` | yes | `postgresql://library:library@db:5432/library` |
| `JWT_SECRET` | yes | long random string |
| `ADMIN_USERNAME` / `ADMIN_PASSWORD` | yes | defined in `.env`, never in the repository |
| `COVERS_DIR` | no | `./var/covers` |
| `CORS_ORIGIN` | yes | `http://localhost:5173` |

## 3. Testing strategy

| Level | What it tests | Against what |
|---|---|---|
| Domain | Invariants: maximum of 3, 14-day due date, available copies in range | None: pure objects |
| Application | Use cases: lend, return, report | Port fakes |
| Integration | Repositories and report query | Real PostgreSQL |
| Contract | Single error body and status codes | Running API |
| Architecture | Articles I, II, and III | NetArchTest |
| Concurrency | Two loans for the last copy | Real PostgreSQL |

## 4. Build order

Skeleton and health → initial migration with seed → domain → use cases with fakes → adapters → API → contract and concurrency tests.
Details and evidence are in [`tasks.md`](tasks.md).
