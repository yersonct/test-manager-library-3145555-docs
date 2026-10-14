# ADR-002 · Optimistic concurrency for copies

- **Status:** accepted
- **Context:** two librarians may loan the **last copy** of the same book at the same time (RNF-2). If both read
  `available_copies = 1` and both write, the system loans a book it does not have.
- **Decision:** **optimistic** concurrency control. The book has a version token; when saving, if the version has changed, the
  operation fails and the server responds **409 `BOOK_NOT_AVAILABLE`** (or retries once if the copy is still available).
  The **final safeguard** is the `CHECK` `ck_book_available_range` (`0 <= available_copies <= total_copies`).
- **Rejected alternatives:** pessimistic row lock (`SELECT … FOR UPDATE`) on every loan: serializes the whole library for
  a rare case; in-memory counter: does not survive more than one process.
- **Consequences:** if the `CHECK` fires, **something wrote outside the adapter**: this is a defect, not a business case. Returning the same item twice
  does not increment twice because `returned_at` is already populated (CA-07.2).
- **Verification:** concurrent test with two simultaneous requests for a book with one copy: exactly one returns 201.
