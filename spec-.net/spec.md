# Specification — Manager Library

**What is being built and why.** A system for a small library to manage its book catalog,
members, and loans, and see which books are borrowed most often.

Governed by [`constitution.md`](constitution.md). The data model lives only in [`data-model.md`](data-model.md)
and the HTTP contract only in [`api-contract.md`](api-contract.md).

---

## 1. Actors

| Actor | What they do | How they authenticate |
|---|---|---|
| **Administrator** (`admin`) | Maintains the book catalog and members; creates librarian accounts; views all reports | Username and password → token |
| **Librarian** (`librarian`) | Records loans and returns; views the catalog, members, and overdue items | Username and password → token |
| **Member** | Person who borrows books. **Does not access the system**: a data record, not a user | — |

The deployment provisions the `admin` role from environment variables. **No one grants it at runtime.**

## 2. Scope

**In scope:** book catalog with optional covers, five fixed categories, members, loans with multiple
items, return by item, overdue books, most-borrowed report, role-based login.

**Out of scope and not to be revisited:** fines and payments, reservations, renewals, email or SMS, bulk import,
inter-branch loans, book attributes beyond those in §3 (no publisher, year, or language).

## 3. User stories and acceptance criteria

### HU-01 · Log in
As a user, I want to authenticate so I can operate according to my role.
- **CA-01.1** Valid credentials return a token valid for 8 hours.
- **CA-01.2** Invalid credentials return 401 with a message that does **not** say whether the username or password was incorrect.
- **CA-01.3** Every endpoint except login and `/health` requires a token. No token: 401. Insufficient role: 403.

### HU-02 · Create librarian accounts (admin)
- **CA-02.1** The admin creates a user with a username, password, and `librarian` role.
- **CA-02.2** The username is stored in lowercase and trimmed; it is unique. Duplicate: 409.
- **CA-02.3** Requesting the `admin` role through the API returns 422. The password is never returned or logged.

### HU-03 · Manage books (admin)
- **CA-03.1** Create a book with a title, author, ISBN, category, and total number of copies. Available copies initially equal the total.
- **CA-03.2** The ISBN has 13 digits and is unique among active books. Duplicate: 409.
- **CA-03.3** Total copies ≥ 1. Changing the total adjusts available copies by the same difference and **never** allows available copies < 0 (422).
- **CA-03.4** Uploading a cover (JPG/PNG, max. 2 MB) stores an **opaque key**; without a cover, the field is `null`, never an empty string.
- **CA-03.5** Deleting a book is a **soft delete**: it disappears from the catalog but remains in past loans. A book with open loans cannot be soft-deleted (409).

### HU-04 · Search books
- **CA-04.1** Search by partial text in title or author, filter by category, active books only, sorted by title, paginated (`page`, `size`, max. 50).
- **CA-04.2** The response includes the total number of results.
- **CA-04.3** Looking up a nonexistent or soft-deleted book by ID: 404.

### HU-05 · Manage members
- **CA-05.1** Admins and librarians create members with a name and document number. The document number is unique: duplicate returns 409.
- **CA-05.2** Members can be listed and searched by name or document number, with pagination.
- **CA-05.3** Members are not deleted.

### HU-06 · Record a loan
- **CA-06.1** A loan belongs to **one** member, is recorded by the authenticated user, and contains 1 to 3 items.
- **CA-06.2** The same book cannot appear more than once in a loan (422).
- **CA-06.3** Each item deducts **1** available copy in the same operation. No copies available: 409 and **nothing is recorded**.
- **CA-06.4** A member cannot have more than **3 unreturned books** in total (422 with the current count).
- **CA-06.5** A member with any **overdue** book cannot borrow another (409).
- **CA-06.6** The due date is `loaned_at + 14 days`. The item freezes the title, ISBN, and category name **at that moment**.
- **CA-06.7** A recorded loan is not edited or deleted.

### HU-07 · Record a return
- **CA-07.1** Return **one item** at a time. Set `returned_at` and add 1 to available copies.
- **CA-07.2** Returning an already returned item: 409. **Never** increment twice.
- **CA-07.3** Available copies never exceed the total (guaranteed by the database).
- **CA-07.4** When all items are returned, the loan is shown as **closed**.

### HU-08 · View a loan and list loans
- **CA-08.1** Looking up a loan returns its items and calculated status (`open`, `overdue`, `closed`).
- **CA-08.2** List by date range, status, and member, sorted by date descending, with pagination.
- **CA-08.3** Status **is calculated**, not stored.

### HU-09 · Overdue books
- **CA-09.1** List unreturned items with `due_at` earlier than now: book, member, days late.
- **CA-09.2** Sorted by most days late first. `admin` and `librarian` only.

### HU-10 · Most-borrowed books report
- **CA-10.1** Given a date range, return one row per **book and frozen category name**, with the number of times borrowed, sorted descending.
- **CA-10.2** End before start: 422. Range with no loans: empty list, not an error.
- **CA-10.3** The report is **not** broken down by librarian or member.
- **CA-10.4** A report for a closed period **does not change** if the book or its category is later renamed.
- **CA-10.5** `admin` only.

## 4. Business rules

| # | Rule |
|---|---|
| RN-1 | Available copies are always between 0 and total copies. |
| RN-2 | Loan period: 14 days. Maximum of 3 unreturned books per member. |
| RN-3 | A member with overdue items cannot borrow more until returning them. |
| RN-4 | Frozen item data (title, ISBN, category) does not follow the catalog. |
| RN-5 | Loans and returns are not deleted. Books are soft-deleted only. |
| RN-6 | Dates are UTC instants. User-facing messages are in Spanish; code and columns are in English. |

## 5. Non-functional requirements

| # | Requirement | How it is measured |
|---|---|---|
| RNF-1 | Book search responds in under 300 ms with 5,000 books | Load test using the `tool` seeder |
| RNF-2 | Two librarians loaning the **last copy** at the same time: one succeeds, the other receives 409 | Concurrent test |
| RNF-3 | The password is never transmitted or stored in plaintext; it never appears in logs | Review of logs and responses |
| RNF-4 | Every HTTP error uses the same body (see contract) | Contract test |
| RNF-5 | The system starts with one command and the database starts empty | `infra` `README` |
