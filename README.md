# Manager Library Documentation

## Selected stack

The selected implementation stack is **PHP 8.2+ with Laravel 12 for the API** and **React 19 + TypeScript + Vite for the web application**, backed by PostgreSQL 16. Architecture, technical decisions, and implementation status are documented in [`spec-php-laravel/`](spec-php-laravel/).

The `spec-.net/` and `spec-python/` directories contain English-language reference specifications from earlier technology alternatives. They are not the selected implementation stacks. Their functional requirements are retained for traceability; do not follow their ASP.NET, EF Core, Python, FastAPI, or SQLAlchemy implementation guidance for this project.

## Documents

- [`spec-php-laravel/README.md`](spec-php-laravel/README.md): selected stack, repositories, and boundaries.
- [`spec-php-laravel/architecture.md`](spec-php-laravel/architecture.md): Laravel API, React, and PostgreSQL architecture.
- [`spec-php-laravel/plan.md`](spec-php-laravel/plan.md): technical decisions for PHP/Laravel and React.
- [`spec-php-laravel/tasks.md`](spec-php-laravel/tasks.md): implementation scope and validation status.
- [`implementation-decisions.md`](implementation-decisions.md): language policy, scope, and contract discrepancies.
- [`spec-.net/`](spec-.net/) and [`spec-python/`](spec-python/): retained functional reference specifications.

Functional requirements and the HTTP contract are referenced from [`spec-.net/spec.md`](spec-.net/spec.md), [`spec-.net/api-contract.md`](spec-.net/api-contract.md), and [`spec-.net/data-model.md`](spec-.net/data-model.md). These behavior specifications are language-independent and apply to the Laravel implementation, subject to the documented clarifications.

## Language policy

Technical documentation, source code, comments, identifiers, and developer-facing messages use English. Text presented to end users remains in Spanish. Spanish category names (`Novela`, `Ciencia`, `Tecnología`, `Historia`, `Infantil`) are product data displayed in the user interface and are intentionally retained.
