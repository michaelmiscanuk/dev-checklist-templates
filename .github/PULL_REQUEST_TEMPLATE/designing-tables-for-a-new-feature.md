## Designing tables for a new feature

When: You are about to write the first migration for a new feature and need tables, keys and constraints that will still fit next year.

Before you start: Will some rows be hidden rather than deleted? (any that apply) Hidden but kept / None of these. Checks marked "only if" apply only to that answer.
Before you start: Can two requests write the same row at once? (any that apply) Yes, it can happen / None of these. Checks marked "only if" apply only to that answer.

### 1. Before you draw any tables

You know what the feature does, and no table or migration is written yet.

Must:
- [ ] List the queries the feature runs

Stop if:
- If the feature stores card, health or identity numbers: stop and ask your security or compliance lead first.

Move on when every entity, relationship, query and retention rule is written down and a teammate has read it.

### 2. When you lay out each table (every time)

You are writing one table's columns, types and primary key; come back here for every table.

Must:
- [ ] Give every table a primary key
- [ ] Store times as zone-aware timestamps

Stop if:
- If one column holds different kinds of value per row: stop and split it into separate columns.

Move on when every planned table has a primary key, the narrowest fitting types and nullability decided.

### 3. When you connect the tables

The tables are drafted; now you make the database enforce the links and rules between them.

Must:
- [ ] Declare every foreign key
- [ ] Enforce uniqueness with a constraint
- [ ] Set each foreign key's delete rule

Stop if:
- If a cascade would remove invoices, payments or audit records: stop and switch that key to restrict.

Move on when every link is a foreign key with a delete rule, and schema-enforceable rules live in the schema.

### 4. When you add indexes

Constraints are in place; now you match indexes to the query list from the start.

Must:
- [ ] Index child-side foreign key columns

Move on when every query on the list has a matching index and no index duplicates another.

### 5. Before you merge the migration

The migration is written and runs on your machine; this is the last look before review.

Must:
- [ ] Test the constraints with bad data
- [ ] Run the migration from empty

Stop if:
- If the migration alters a table holding live data: stop and follow the live schema change checklist.
- If a listed query still scans the whole table: ask your DBA or a senior engineer.

Move on when the migration runs clean from empty, bad rows fail at the database and a reviewer has approved it.

---
Must checks only. Why each check matters, how to verify it and its source: [full checklist](https://github.com/michaelmiscanuk/dev-checklist-templates/blob/main/checklists/designing-tables-for-a-new-feature.md). Interactive version: [Designing tables for a new feature on CheatGrid](https://www.cheatgrid.com/checklists/backend-developer/designing-tables-for-a-new-feature?utm_source=github&utm_medium=listing&utm_campaign=designing-tables-for-a-new-feature).
