# ADR-004 · The report is computed in the database and grouped by frozen values

- **Status:** accepted
- **Context:** the most-borrowed report (HU-10) joins `loan` and `loan_item` over a range. If it read the title or category from the live catalog,
  renaming a book or category would change a report for a period that has already closed.
- **Decision:** (1) the report is **not persisted**: it is computed with one aggregate query through a read port. (2) It groups by
  `book_id, book_title, category_name` **as frozen in the item**. (3) `loan_item.category_name` intentionally **has no FK**.
  (4) If a book changed categories within the range, the report shows **two rows**: it does not pick a winner.
- **Rejected alternatives:** joining with the live catalog (the report becomes unstable); taking “the most recent value in the range”
  (a new loan would change what has already been read); materializing a report table (a second source of truth).
- **Consequences:** CA-10.1 says “one row per book and frozen category name,” not “one row per book.” The report does not
  include members or librarians (privacy).
- **Verification:** loan a book, rename it, request the report for the same range: the report title is the old one.
