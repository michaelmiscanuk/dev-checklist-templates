# Designing tables for a new feature

When: You are about to write the first migration for a new feature and need tables, keys and constraints that will still fit next year.

How to use: Open this once the feature's data is clear enough to sketch, before the first migration exists. Work the moments in order; the table moment repeats for every table you add. Each check names one thing to get right, and How to verify says what to look at and what right looks like. Answer the two questions first so only your checks show.

Before you start: Will some rows be hidden rather than deleted? (any that apply) Hidden but kept / None of these. Checks marked "only if" apply only to that answer.
Before you start: Can two requests write the same row at once? (any that apply) Yes, it can happen / None of these. Checks marked "only if" apply only to that answer.

## 1. Before you draw any tables

You know what the feature does, and no table or migration is written yet.

Must:
- [ ] List the queries the feature runs
  Why: Write each screen's and endpoint's reads and writes, with their filters and sort order; tables and indexes follow from that list.
  How to verify: A written list of every read and write sits beside the draft schema, each naming its filter, sort and **expected volume**.

Don't forget these too:
- [ ] Reuse tables that already exist
  Why: Search the schema for users, addresses or files before modelling them again; a second copy drifts from the first.
  How to verify: Each new table has a note on why no existing table fits it.
- [ ] Name each entity in one sentence
  Why: If the sentence needs 'and' to hold two things, it is probably two tables.
  How to verify: Every planned table has a one-line definition a teammate understands without asking what it means.
- [ ] Label each relationship's cardinality
  Why: Say one-to-one, one-to-many (an order and its line items) or many-to-many, and whether the child can exist without the parent.
  How to verify: Every pair of related entities carries its label on the sketch, including whether the link is optional.
- [ ] Estimate rows per year
  Why: Rough orders of magnitude are enough to spot the one table that needs archiving or partitioning.
  How to verify: Each table has a row estimate for year one; the largest few have a note on how they grow.
- [ ] Mark personal and sensitive fields
  Why: Flagging them now decides encryption, access rules, retention and how a deletion request reaches them.
  How to verify: The sketch marks every column holding personal data, with where it must be erased or kept.
- [ ] Scope every row to its tenant
  Why: In a multi-tenant app, put the tenant key on every table so each query can filter on it.
  How to verify: Every table either carries the tenant or owner key or reaches it through a required foreign key.
- [ ] Decide how long rows are kept
  Why: Agree a retention rule per table with whoever owns the feature; it decides delete versus hide and the archive plan.
  How to verify: Each table lists its retention: forever, a fixed period, or until the user deletes it, and what job erases or anonymises expired rows.
- [ ] Avoid type-and-id polymorphic links
  Why: Where a row can point at several tables, use one nullable foreign key per target with a check that **exactly one is set**.
  How to verify: No planned table pairs a type column with an id column that cannot carry a foreign key.
- [ ] Decide which facts need history (source 1)
  Why: To answer what a price or plan was on a past date, keep valid-from and valid-to rows; SQL Server **temporal tables** do this automatically.
  How to verify: Each table whose values change over time is marked overwrite or history, and the history ones have their dated rows planned.

Stop if:
- If the feature stores card, health or identity numbers: stop and ask your security or compliance lead first.

Move on when every entity, relationship, query and retention rule is written down and a teammate has read it.

## 2. When you lay out each table (every time)

You are writing one table's columns, types and primary key; come back here for every table.

Must:
- [ ] Give every table a primary key
  Why: Use a generated id, such as bigint GENERATED ALWAYS AS IDENTITY in Postgres, unless a natural key truly never changes; emails and names change.
  How to verify: Every table in the migration declares a primary key, and no key column holds a value users can edit.
- [ ] Store times as zone-aware timestamps (source 2)
  Why: Use the zone-aware type (timestamptz in Postgres) and **keep values in UTC**; convert to local time only for display.
  How to verify: Every moment-in-time column uses the zone-aware type, never a plain timestamp holding UTC; date columns hold only calendar dates such as birthdays.

Don't forget these too:
- [ ] Keep one id type schema-wide (source 3)
  How to verify: Primary and foreign keys in the new tables use the same type as the rest of the schema.
- [ ] Make columns NOT NULL by default
  Why: Allow null only where 'unknown' is a real answer, and write down what null means there.
  How to verify: Each nullable column in the migration has a comment or note saying what a null value stands for.
- [ ] Use the narrowest fitting type
  Why: Dates as date, flags as boolean, counts as integers; text for everything throws away checks the database gives for free.
  How to verify: No column stores dates, numbers or yes/no values as text.
- [ ] Store money as exact amounts (source 4)
  Why: Use numeric, which is exact but slower, or whole minor units such as cents in an integer, with a currency column beside it.
  How to verify: Money columns are decimal or integer, never floating point, and each has a currency column or a documented fixed currency.
- [ ] Constrain status to known values
  Why: A check constraint or lookup table is easier to change later than a database enum type.
  How to verify: Writing a misspelled status in a test fails with a constraint error.
- [ ] Add created and updated timestamps (source 5)
  Why: Set the defaults in the database, not the app, so scripts and jobs get them and created never lands after updated.
  How to verify: Both columns exist, are not null and have a database default; updated-at changes when a row is edited.
- [ ] Keep each fact in one place
  Why: Copy a value only when you mean a snapshot, like the price at the time of the order.
  How to verify: Every duplicated column has a note saying it is a snapshot or a cache, and what keeps it fresh.
- [ ] Keep queried fields out of JSON
  Why: A JSON column suits data the app only reads whole; fields you filter, join or constrain on need real columns.
  How to verify: No query from your list filters or sorts on a key inside a JSON column.

Stop if:
- If one column holds different kinds of value per row: stop and split it into separate columns.

Move on when every planned table has a primary key, the narrowest fitting types and nullability decided.

## 3. When you connect the tables

The tables are drafted; now you make the database enforce the links and rules between them.

Must:
- [ ] Declare every foreign key (source 6)
  Why: Let the database refuse orphans; scripts, jobs and concurrent requests all skip checks done in app code.
  How to verify: Every column that points at another table has a foreign key constraint in the migration.
- [ ] Enforce uniqueness with a constraint (source 7)
  Why: A check in code cannot stop two concurrent inserts; **only a unique constraint** or unique index can.
  How to verify: Inserting the same value twice in a test raises a unique violation, not a second row.
- [ ] Set each foreign key's delete rule
  Why: Cascade only to **true parts of the parent**, like order lines; restrict anything with a life of its own.
  How to verify: Every foreign key states cascade, restrict or set null on purpose, and the choice is noted for the reviewer.

Don't forget these too:
- [ ] Model many-to-many with a join table
  Why: Make the pair of foreign keys unique, and give the link its own columns when it carries data.
  How to verify: The join table has both foreign keys and a unique rule on the pair, and no list of ids sits in a single column.
- [ ] Add check constraints for value rules (source 8)
  Why: Put single-row rules like quantity above zero or end after start in a check; rules across rows need unique, exclusion or foreign keys.
  How to verify: A test that inserts a negative quantity or a reversed date range fails at the database.
- [ ] Compare emails case-insensitively (source 9)
  Why: Put the unique index on the lowercased email, or use a case-insensitive text type, so values differing only in case are refused.
  How to verify: Inserting the same email in different letter case is rejected as a duplicate.
- [ ] Add a deleted-at timestamp (only if: Hidden but kept)
  Why: A nullable deleted-at time says both whether and when a row was hidden, which a boolean flag does not.
  How to verify: Hidden rows have a deleted-at time; live rows have null there.
- [ ] Limit unique rules to live rows (only if: Hidden but kept)
  Why: Use a partial unique index on rows not deleted, so a hidden row does not block a new one.
  How to verify: After hiding a row, inserting a new row with the same unique value succeeds.
- [ ] Filter hidden rows in one place (source 10) (only if: Hidden but kept)
  Why: Use a view or the ORM's default scope so no query has to remember the filter.
  How to verify: The feature's list and detail queries return no hidden rows without each one adding the filter by hand.
- [ ] Add a version column for edits (source 11) (only if: Yes, it can happen)
  Why: This is optimistic locking: each update names the version it read and bumps it, so a stale write changes nothing.
  How to verify: With two test sessions editing the same row, the second save fails and the app reports a conflict.
- [ ] Update counters in one statement (only if: Yes, it can happen)
  Why: Let the database add to the stored value instead of reading it, changing it and writing it back.
  How to verify: Counter and balance changes use a single update that adds to the column, never a read followed by a write.
- [ ] Make retried creates idempotent (source 12)
  Why: Store the client's idempotency key, a unique id per request, under a unique constraint so a retry finds the first row.
  How to verify: Sending the same create request twice leaves one row and returns it both times.

Stop if:
- If a cascade would remove invoices, payments or audit records: stop and switch that key to restrict.

Move on when every link is a foreign key with a delete rule, and schema-enforceable rules live in the schema.

## 4. When you add indexes

Constraints are in place; now you match indexes to the query list from the start.

Must:
- [ ] Index child-side foreign key columns (source 13)
  Why: Postgres indexes only the referenced side; add the child-side index so joins and **parent deletes stay fast**.
  How to verify: Every foreign key column you join, filter or delete through has an index that starts with it.

Don't forget these too:
- [ ] Index each query's filter and sort (source 14)
  Why: Follow **equality, sort, range**: in a composite index, equality columns first, then the sort column, then any range column.
  How to verify: Each read on the query list has an index whose leading columns match its filter and order.
- [ ] Reuse indexes constraints already build (source 8)
  Why: Primary keys and unique constraints create their own index; a second one on the same columns only slows writes.
  How to verify: No two indexes in the migration start with the same columns in the same order.
- [ ] Index only the rows queries touch (source 15)
  Why: A partial index on, say, open orders or live rows stays small when most rows are closed or hidden.
  How to verify: Hot queries on a small slice of a large table use a partial index matching their condition.
- [ ] Scope unique rules to the tenant
  Why: In a multi-tenant app, include the tenant key in unique rules on names and slugs, and lead tenant-filtered indexes with it.
  How to verify: Two tenants can each create a row with the same name, and tenant-filtered queries use an index that **starts with the tenant** column.

Move on when every query on the list has a matching index and no index duplicates another.

## 5. Before you merge the migration

The migration is written and runs on your machine; this is the last look before review.

Must:
- [ ] Test the constraints with bad data
  Why: Try a duplicate, an orphan, a null and an out-of-range value; each should fail **at the database**.
  How to verify: A test inserting each bad row fails with the named constraint in the error.
- [ ] Run the migration from empty
  Why: Run it on a fresh schema and, if your tool supports it, roll it back and run it again.
  How to verify: Up, down and up again all finish without errors on a fresh database.

Don't forget these too:
- [ ] Read query plans on realistic data
  Why: Seed the estimated row counts first; a plan on ten rows says nothing about a million.
  How to verify: The query plan for each listed read uses an index, not a full table scan, on the seeded data.
- [ ] Keep ORM models matching the schema
  Why: Let the database own defaults, nullability and constraints; the model mirrors them so validation errors read well.
  How to verify: Model fields match the migration's types, nullability and defaults, and a generated diff between them shows nothing.
- [ ] Ship lookup rows with the migration
  Why: Insert fixed rows such as status codes or roles in the same change (data seeding), so every environment has them.
  How to verify: A fresh database after migrating holds every lookup row the feature reads.
- [ ] Walk a reviewer through the queries
  Why: Show the query list beside the schema; reviewers spot missing indexes and keys faster than in raw migration text.
  How to verify: A second engineer has approved the schema after seeing the query list and the delete rules.
- [ ] Name constraints and indexes explicitly (source 16)
  Why: Follow one pattern, such as table, columns and kind, so later migrations can drop or change them by name.
  How to verify: Every constraint and index in the migration has a written name that follows the pattern, none left to the tool.

Stop if:
- If the migration alters a table holding live data: stop and follow the live schema change checklist.
- If a listed query still scans the whole table: ask your DBA or a senior engineer.

Move on when the migration runs clean from empty, bad rows fail at the database and a reviewer has approved it.

Not covered: Changing tables that already hold live data has its own checklist, as does finding why an endpoint got slow. Choosing a database engine and designing the API are left out.

Last reviewed: 2026-10-12

Sources:
1. Microsoft Learn: Temporal tables, history and validity periods: https://docs.microsoft.com/en-us/sql/relational-databases/tables/temporal-tables
2. PostgreSQL Wiki: Don't Do This, timestamp vs timestamptz: https://wiki.postgresql.org/wiki/Don%27t_Do_This
3. ietf.org: https://www.ietf.org/archive/id/draft-peabody-dispatch-new-uuid-format-04.html
4. PostgreSQL Docs: Numeric Types: https://www.postgresql.org/docs/current/datatype-numeric.html
5. Gunnar Morling: Last-updated columns with Postgres: https://www.morling.dev/blog/last-updated-columns-with-postgres/
6. PostgreSQL Docs: Foreign Keys tutorial: https://www.postgresql.org/docs/current/tutorial-fk.html
7. PostgreSQL Docs: Index Uniqueness Checks: https://www.postgresql.org/docs/15//index-unique-checks.html
8. PostgreSQL Docs: Constraints: https://www.postgresql.org/docs/current/ddl-constraints.html
9. PostgreSQL Docs: Indexes on Expressions: https://www.postgresql.org/docs/15/indexes-expressional.html
10. Rails API: ActiveRecord default_scope: https://api.rubyonrails.org/v7.2/classes/ActiveRecord/Scoping/Default/ClassMethods.html
11. AWS DynamoDB Docs: Optimistic locking with version numbers: https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/BestPractices_OptimisticLocking.html
12. Stripe Docs: Idempotent requests: https://docs.stripe.com/api/idempotent_requests
13. CYBERTEC: Index your foreign key: https://www.cybertec-postgresql.com/en/index-your-foreign-key/
14. MongoDB Docs: Equality, Sort, Range guideline: https://www.mongodb.com/docs/manual/tutorial/equality-sort-range-guideline/
15. PostgreSQL Docs: Partial Indexes: https://www.postgresql.org/docs/current/indexes-partial.html
16. Alembic: Naming Conventions: https://alembic.sqlalchemy.org/en/latest/naming.html

From CheatGrid: https://www.cheatgrid.com/checklists/backend-developer/designing-tables-for-a-new-feature?utm_source=github&utm_medium=listing&utm_campaign=designing-tables-for-a-new-feature
