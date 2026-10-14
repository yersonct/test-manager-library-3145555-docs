# Manager Library Technical Plan (PHP/Laravel)

This plan selects **PHP/Laravel + React** to implement the functional requirements in [`../spec-.net/spec.md`](../spec-.net/spec.md). The `spec-.net/` and `spec-python/` directories are behavior references and do not indicate that those technologies are part of the solution.

## Technical decisions

| Area | Decision |
|---|---|
| API | PHP 8.2+ and Laravel 12 |
| Persistence | PostgreSQL 16; Laravel migrations own the schema |
| Administrative client | React 19, TypeScript 5.8, Vite 6 |
| Contract | REST JSON under `/api/v1`, `snake_case` fields, Spanish user-facing messages |
| Identity | JWT HS256 with an eight-hour lifetime; BCrypt password hashing |
| Covers | JPG/PNG up to 2 MB, stored outside the database |
| Dates | `timestamptz` instants serialized in UTC |
| Local startup | Docker Compose; database remains empty until migrations are explicitly run |

## Configuration

The API reads environment variables documented in `test-manager-library-3145555-api/.env.example`:

| Variable | Purpose |
|---|---|
| `DB_CONNECTION`, `DB_HOST`, `DB_PORT`, `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD` | PostgreSQL connection |
| `JWT_SECRET`, `JWT_TTL` | Token signing and lifetime |
| `ADMIN_USERNAME`, `ADMIN_PASSWORD` | Initial administrator provisioning |
| `CORS_ORIGIN` | Authorized web-client origin |
| `APP_KEY`, `APP_ENV`, `APP_DEBUG` | Laravel configuration |

Do not commit `.env` files, keys, or passwords. The app sets `VITE_API_URL` when building; locally, it defaults to `http://localhost:8000/api/v1`.

## Local startup sequence

1. Clone the sibling repositories and prepare infrastructure and API `.env` files from their examples.
2. Set secure credentials for PostgreSQL, JWT, and the administrator.
3. Generate `APP_KEY` by running `php artisan key:generate --force` in the API service.
4. Start services with `docker compose up --build`.
5. Explicitly initialize the schema with `docker compose exec api php artisan migrate --force`.
6. Optionally load demo records through the Compose demo profile.

Do not run migrations automatically during container startup: a new database must remain empty until the explicit migration action.

## Validation

- Frontend: `npm run build` runs TypeScript checks and Vite.
- API: run `composer validate`, `php artisan test`, and PHP syntax/style checks when PHP and Composer are available.
- Integration: migrate an isolated PostgreSQL test database and validate contracts, authentication, constraints, concurrency, and loan/return flows.
- Infrastructure: run `docker compose config --quiet` with local `.env` files, then build and start with Docker Desktop running.
- Demo: verify the seeder writes only through the API and does not duplicate records on a second run.

## Known differences

The contract sets loan due dates to 14 days after creation and exposes no operation to modify `due_at`. Therefore the current seeder cannot create overdue loans and does not meet the original demo-volume task (40 books, 10 members, 15 loans, and 3 overdue items). Do not claim these criteria are complete or add an endpoint/direct SQL access to simulate them without an approved contract change.
