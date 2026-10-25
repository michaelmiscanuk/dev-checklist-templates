---
name: Querying data for a business question
about: Someone asks a question the data can answer and you are about to write the SQL that produces the number.
title: "Data question: "
labels: ["analysis"]
---
## Querying data for a business question

When: Someone asks a question the data can answer and you are about to write the SQL that produces the number.

Before you start: Could the data hold test, internal or deleted records? (any that apply) Test or internal accounts / Deleted or soft-deleted records / Not sure yet / None of these. Checks marked "only if" apply only to that answer.
Before you start: Will anyone run this query again? (any that apply) Me, later on / Teammates or a schedule / None of these. Checks marked "only if" apply only to that answer.

### 1. Before you write any SQL

Someone asked a question and you are about to open the SQL editor.

Must:
- [ ] Restate the question in one sentence
- [ ] Write the metric definition down
  - [ ] What counts as one event
  - [ ] Numerator
  - [ ] Denominator
  - [ ] What is included and left out

Don't forget these too:
- [ ] Ask what decision it feeds
- [ ] Find the existing official number
- [ ] Name who is in the population
- [ ] Agree the exact date range
- [ ] Sketch the output table first
- [ ] Flag cause-and-effect questions early

Stop if:
- If two teams define this metric differently: ask the metric's owner which one to use before writing SQL

Move on when the asker has agreed to a written metric, period and population.

### 2. When you choose the tables

You know what to compute and are looking for where it lives.

Must:
- [ ] Name the grain of every table

Don't forget these too:
- [ ] Use the team's trusted table
- [ ] Find the latest load date
- [ ] Read what each column means
- [ ] Look for test and deleted rows
- [ ] Get access before using personal data
- [ ] Run heavy queries off production

Stop if:
- If nobody can say what a column means: ask the table's owner before you build on it

Move on when you know the grain, freshness and quirks of every table you will use.

### 3. When you write the filters

You are deciding which rows go in: dates, accounts, statuses and missing values.

Must:
- [ ] Use a half-open date range
- [ ] Pin the reporting time zone
- [ ] Exclude test and internal accounts (only if: Test or internal accounts or Not sure yet)

Don't forget these too:
- [ ] Apply the soft-delete marker (only if: Deleted or soft-deleted records or Not sure yet)
- [ ] Decide if deleted records count (only if: Deleted or soft-deleted records or Not sure yet)
- [ ] Count rows each exclusion removes (only if: Test or internal accounts or Deleted or soft-deleted records or Not sure yet)
- [ ] Handle NULLs in exclusion filters
- [ ] Decide which statuses count
- [ ] Use NOT EXISTS over NOT IN

Stop if:
- If you cannot tell test accounts from real ones: ask the data engineering team for the marker

Move on when every filter is one you chose on purpose and can explain.

### 4. When you join and aggregate

Rows are filtered and you are combining tables and rolling them up to the answer.

Must:
- [ ] Count the entity the question names

Don't forget these too:
- [ ] Run each step on its own
- [ ] Pick each join type on purpose
- [ ] Filter left-joined tables in the join
- [ ] Group by the answer's grain only
- [ ] Divide summed totals for ratios
- [ ] Cast before dividing integers
- [ ] Decide what an empty value means
- [ ] Join attributes as of event time

Stop if:
- If a join changes the row count you expected: stop and find which key repeats before going on

Move on when the query runs and its result has the shape of your output sketch.

### 5. Before you hand over the number

The query returns a result and you are about to send it.

Must:
- [ ] Get the number a second way
- [ ] Send the definition with the number
  - [ ] Date range and time zone
  - [ ] Population and exclusions
  - [ ] Data cutoff
  - [ ] Link to the query

Don't forget these too:
- [ ] Follow a few records by hand
- [ ] Plot the result over time
- [ ] Size it against what you know
- [ ] Save the query with the answer
- [ ] Put dates in one place (only if: Me, later on or Teammates or a schedule)
- [ ] Comment why each filter exists (only if: Me, later on or Teammates or a schedule)
- [ ] Store it in the shared library (only if: Teammates or a schedule)
- [ ] Breakdowns add up to the total
- [ ] Have a peer review high-stakes SQL

Stop if:
- If the two routes disagree without a reason: hold the number and tell the asker
- If it contradicts an official report: tell the asker and the report's owner before sharing it

Move on when the number has matched a second route and goes out with its definition.

---
Why each check matters, how to verify it and its source: [full checklist](https://github.com/michaelmiscanuk/dev-checklist-templates/blob/main/checklists/querying-data-for-a-business-question.md). Interactive version: [Querying data for a business question on CheatGrid](https://www.cheatgrid.com/checklists/data-analyst/querying-data-for-a-business-question?utm_source=github&utm_medium=listing&utm_campaign=querying-data-for-a-business-question).
