# Manager Library — PHP/Laravel Implementation Specification

This is the selected technology architecture for Manager Library. The API uses **PHP 8.2+ and Laravel 12**, the administrative client uses **React 19 + TypeScript + Vite 6**, and persistence uses **PostgreSQL 16**.

> This directory documents the selected implementation. `spec-.net/` and `spec-python/` are retained as English-language functional references from earlier alternatives; their technology decisions do not apply to this stack.

## Product requirements and contract

Product behavior, user stories, endpoints, responses, and logical data model are based on:

- [`../spec-.net/spec.md`](../spec-.net/spec.md)
- [`../spec-.net/api-contract.md`](../spec-.net/api-contract.md)
- [`../spec-.net/data-model.md`](../spec-.net/data-model.md)
- [`../spec-.net/constitution.md`](../spec-.net/constitution.md)

These are language-independent functional specifications. Adaptations and discrepancies are recorded in [`../implementation-decisions.md`](../implementation-decisions.md).

## Repositories and boundaries

Clone the six repositories as sibling directories:

| Repository | Responsibility |
|---|---|
| `test-manager-library-3145555-api` | PHP/Laravel REST API, PostgreSQL, authentication, and business rules |
| `test-manager-library-3145555-app` | React + TypeScript operations client |
| `test-manager-library-3145555-page` | Informational static website; no API or database access |
| `test-manager-library-3145555-infra` | Docker Compose for local services |
| `test-manager-library-3145555-tool` | Demo data created exclusively through the API |
| `test-manager-library-3145555-docs` | Requirements and implementation documentation |

The Laravel application lives at the root of `test-manager-library-3145555-api/`. No .NET or Python framework is part of the selected stack. No client or tool writes directly to PostgreSQL.

## Related documentation

- [Architecture](architecture.md)
- [Technical plan](plan.md)
- [Tasks and validation status](tasks.md)

The functional data model retains the six tables `category`, `user`, `book`, `member`, `loan`, and `loan_item`. Do not add fines, reservations, renewals, notifications, branches, or book fields beyond the specified scope.
