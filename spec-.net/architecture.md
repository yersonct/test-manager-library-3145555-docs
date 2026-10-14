# Architecture — Manager Library (.NET)

**Hexagonal** architecture (ports and adapters). Dependency rules are Articles I through III of [`constitution.md`](constitution.md).

## 1. Layers and dependencies

```mermaid
flowchart LR
    subgraph Inbound
      HTTP["adapters/inbound/http<br/>(ASP.NET Core (Minimal API))"]
    end
    subgraph Core
      APP["application<br/>use cases + ports"]
      DOM["domain<br/>aggregates and value objects"]
    end
    subgraph Outbound
      DB["adapters/outbound/persistence<br/>(EF Core 8 + EF migrations)"]
      STO["adapters/outbound/storage<br/>covers"]
      SEC["adapters/outbound/security<br/>hashing and token"]
    end
    HTTP --> APP --> DOM
    DB -. implements ports from .-> APP
    STO -. implements .-> APP
    SEC -. implements .-> APP
```

Dependency arrows point **inward**. `bootstrap` is the only module that sees both sides.

## 2. Folder structure

```text
src/Library/
├── domain/              # Book, Loan, LoanItem, Member, User; Isbn, Quantity, DateRange
├── application/
│   ├── ports/outbound/  # BookRepository, LoanRepository, MemberRepository, UserRepository,
│   │                    # ReportReader, Clock, PasswordHasher, TokenIssuer, CoverStorage
│   └── services/        # LendBooksService, ReturnItemService, CreateBookService, SearchBooksService,
│                        # OverdueService, MostBorrowedReportService, LoginService, CreateLibrarianService
├── adapters/
│   ├── inbound/http/    # routers/controllers, request schemas, error handling
│   └── outbound/        # persistence/, storage/, security/
└── bootstrap/composition/   # src/Library.Bootstrap/Composition/PortBindings.cs — single composition point
```

## 3. Aggregates

| Root | Contains | Invariants it enforces |
|---|---|---|
| `Book` | `Isbn`, total and available copies, soft delete | available copies between 0 and total; total ≥ 1; non-empty title |
| `Loan` | collection of `LoanItem` | 1 to 3 items; no duplicate books; 14-day due date; immutable except for returns |
| `Member` | document number | required and unique document number |
| `User` | role, hash | normalized username; role in (`admin`, `librarian`) |

`Loan.AddItem` calls `Book.Withdraw` before adding the item: deducting the copy and creating the item are **one operation**.
`Loan.ReturnItem` calls `Book.Restock` once per item.

## 4. Flow: record a loan

```mermaid
sequenceDiagram
    participant C as Client
    participant H as HTTP
    participant S as LendBooksService
    participant M as MemberRepository
    participant B as BookRepository
    participant L as LoanRepository
    C->>H: POST /api/v1/loans
    H->>S: Lend(memberId, bookIds)
    S->>M: fetch member + unreturned items
    S->>S: has overdue items? exceeds 3?
    S->>B: fetch books in batch (Q3)
    S->>S: Loan.AddItem → Book.Withdraw (for each book)
    S->>L: save loan + books (one transaction)
    L-->>S: ok or version conflict
    S-->>H: loan or BOOK_NOT_AVAILABLE
    H-->>C: 201 or 409
```

## 5. Errors

The domain and application throw their own errors (`BookNotAvailable`, `MemberHasOverdue`, `MemberLimitExceeded`, …). **Only** the HTTP adapter
translates them into the single body and status code in [`api-contract.md`](api-contract.md). A framework error never reaches the client.

## 6. What is not included

No events, queues, cache, microservices, or multitenancy. Add them if a requirement calls for them, not before.
