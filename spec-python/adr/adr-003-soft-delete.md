# ADR-003 · Books are soft-deleted, not removed

- **Status:** accepted
- **Context:** a loan item points to a book. If the book is physically deleted, history and reports break.
- **Decision:** the book has `deleted_at`. Soft deletion populates it; the catalog filters on `deleted_at IS NULL` in one place
  (global filter). A book with open loans cannot be soft-deleted (409 `BOOK_HAS_OPEN_LOANS`). ISBN uniqueness uses a **partial**
  index on active books, so an ISBN can be registered again after its book is soft-deleted.
- **Rejected alternatives:** physical deletion with cascade (destroys history); setting `active = false` (does not say when).
- **Consequences:** `loan_item.book_id` has an FK `RESTRICT` as a **last-resort safeguard**: a manual `DELETE` or a future
  code change fails loudly instead of corrupting history. The cover binary is the only data that is physically deleted.
- **Verification:** requesting a soft-deleted book returns 404; a `DELETE` via `psql` on a book with items fails because of the FK.
