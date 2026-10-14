# ADR-004 · The report is calculated by the engine and groups by snapshotted values

- **Status:** accepted
- **Context:** the most-borrowed report (HU-10) joins `loan` and `loan_item` over a range. If it reads the title or category from the
  live catalog, renaming a book or category would change a report for a period that has already closed.
- **Decision:** (1) the report is **not persisted**: it is calculated with a single aggregated query through a read port. (2) It groups by
  `book_id, book_title, category_name` **as snapshotted on the item**. (3) `loan_item.category_name` **intentionally has no FK**.
  (4) If a book's category changed during the range, the report shows **two rows**: it does not choose a winner.
- **Rejected alternatives:** joining the live catalog (the report is no longer stable); taking "the latest value in the range"
  (a new loan would change what was already read); materializing a report table (second source of truth).
- **Consequences:** CA-10.1 says "one row per book and snapshotted category name," not "one row per book." The report does not
  include members or librarians (privacy).
- **Verification:** loan a book, rename it, and request the report for the same range: the report title is the old one.
