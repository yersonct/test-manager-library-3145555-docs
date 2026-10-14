# ADR-002 · Optimistic concurrency for book copies

- **Status:** accepted
- **Context:** two librarians can loan the **last copy** of the same book at the same time (RNF-2). If both read
  `available_copies = 1` and both write, the system loans a book it does not have.
- **Decision:** **optimistic** concurrency control. The book has a version token; when saving, if the version has changed, the
  operation fails and the server returns **409 `BOOK_NOT_AVAILABLE`** (or retries once if the copy is still available).
  The **last line of defense** is the `CHECK` `ck_book_available_range` (`0 <= available_copies <= total_copies`).
- **Rejected alternatives:** pessimistic row locking (`SELECT … FOR UPDATE`) on every loan: serializes the entire library for
  a rare case; in-memory counter: does not survive more than one process.
- **Consequences:** if the `CHECK` fails, **something wrote outside the adapter**: it is a defect, not a business case. Returning the same item twice
  does not increment twice because `returned_at` is already set (CA-07.2).
- **Verification:** concurrent test with two simultaneous requests for a book with one copy: exactly one receives 201.
