# Implementation Decisions

This document accompanies the product specification. It does not replace requirements or expand project scope.

## Repository boundaries

The six repositories remain independent and are cloned as sibling directories:

| Repository | Responsibility |
|---|---|
| `test-manager-library-3145555-api` | PHP/Laravel REST API and PostgreSQL persistence |
| `test-manager-library-3145555-app` | React + TypeScript operations application |
| `test-manager-library-3145555-page` | Static public website with no library-data access |
| `test-manager-library-3145555-infra` | Local container composition |
| `test-manager-library-3145555-tool` | Demo-data loading through the API |
| `test-manager-library-3145555-docs` | Product requirements and shared implementation decisions |

User-facing applications call the HTTP contract; no consumer writes directly to PostgreSQL. The public website is informational and does not query or expose the operational catalog.

## Language and identifiers

Technical documentation, source code, comments, identifiers, and developer-facing messages use English. User-visible interface text and API error messages remain Spanish. Product category values remain `Novela`, `Ciencia`, `Tecnología`, `Historia`, and `Infantil` because they are displayed to users.

## Contract discrepancies

1. The cover section requires `GET /books/{id}/cover`, although that route is missing from the endpoint table. The API implements it so clients can retrieve the image associated with `cover_key`.
2. The authentication row omits the header scheme. The implementation uses the Bearer scheme specified by the login example and security section.
3. ADR-002 mentions a version token that is absent from the data model. The documented schema is preserved without a version column: the API reserves copies within a transaction using an atomic conditional update.
4. The demo-data tool cannot create overdue loans through the public contract: loan creation sets `due_at` to 14 days in the future, with no operation to change that date. No privileged route or direct SQL write is added to work around this. The seeder creates regular books, members, and loans; overdue demos require an explicit contract decision.

## Scope constraints

Do not add fines, reservations, renewals, notifications, branches, bulk inventory, or book fields beyond title, author, ISBN, category, copies, and cover. The schema remains limited to the six functional tables described by the data model.
