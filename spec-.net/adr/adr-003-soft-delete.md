# ADR-003 · Books are soft-deleted, not removed

- **Status:** accepted
- **Context:** a loan item points to a book. If the book is physically deleted, the history and report break.
- **Decision:** the book has `deleted_at`. Soft-deleting sets it; the catalog filters on `deleted_at IS NULL` in one place
  (a global filter). A book with open loans is **not** soft-deleted (409 `BOOK_HAS_OPEN_LOANS`). ISBN uniqueness is a
  **partial** index on active books, so an ISBN from a soft-deleted book can be registered again.
- **Rejected alternatives:** physical deletion with cascade (destroys history); setting `active = false` (does not say when).
- **Consequences:** `loan_item.book_id` has a `RESTRICT` FK as a **last line of defense**: a manual `DELETE` or future code change
  fails loudly instead of corrupting history. The cover binary is the only data that is physically deleted.
- **Verification:** looking up a soft-deleted book returns 404; a `DELETE` via `psql` on a book with loan items fails because of the FK.
