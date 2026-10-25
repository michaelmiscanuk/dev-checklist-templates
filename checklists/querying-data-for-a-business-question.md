# Querying data for a business question

When: Someone asks a question the data can answer and you are about to write the SQL that produces the number.

How to use: Open this when someone asks a question and you are about to write the SQL that answers it. Work through the moments in order: the Must checks are the ones that most often change the number. Each title is the thing to do or the thing you should see, and 'How to verify' says what to look at and what right looks like.

Before you start: Could the data hold test, internal or deleted records? (any that apply) Test or internal accounts / Deleted or soft-deleted records / Not sure yet / None of these. Checks marked "only if" apply only to that answer.
Before you start: Will anyone run this query again? (any that apply) Me, later on / Teammates or a schedule / None of these. Checks marked "only if" apply only to that answer.

## 1. Before you write any SQL

Someone asked a question and you are about to open the SQL editor.

Must:
- [ ] Restate the question in one sentence
  Why: Write it back in the asker's words: which number, for whom, over what period, split by what.
  How to verify: The asker has replied yes to your sentence, or corrected it, before you write any SQL.
- [ ] Write the metric definition down
  - [ ] What counts as one event
  - [ ] Numerator
  - [ ] Denominator
  - [ ] What is included and left out
  Why: Two correct queries disagree when **definitions differ**, so name exactly what is counted and what it is divided by.
  How to verify: Someone else could compute the same number from your note alone, and it matches the team's metric glossary if there is one.

Don't forget these too:
- [ ] Ask what decision it feeds
  Why: The decision tells you how exact the number must be and which breakdown actually matters.
  How to verify: You can name the choice the asker will make with this number.
- [ ] Find the existing official number
  Why: If a dashboard or finance report already shows this metric, **match its definition** or plan to explain the gap.
  How to verify: You found the existing figure and its definition, or its usual owner confirmed none exists.
- [ ] Name who is in the population
  Why: Say which users, accounts, regions or products count, for example paying customers only, or every signup.
  How to verify: Your definition note lists who is in and who is out, and the asker agreed.
- [ ] Agree the exact date range
  Why: Turn words like 'last month' into dates: a calendar month, a rolling window or a fiscal period.
  How to verify: Written start and end dates, plus whether today's unfinished day is in.
- [ ] Sketch the output table first
  Why: Draw the columns and one example row: it fixes the **grain of the answer**, like one row per week per region.
  How to verify: A mock of the result with column names and what one row stands for.
- [ ] Flag cause-and-effect questions early
  Why: A 'did X cause Y' question needs a **comparison group**, such as an experiment or control period, not one query.
  How to verify: For any 'did it work' question, your restated sentence names the comparison, or the asker agreed it needs a separate analysis.

Stop if:
- If two teams define this metric differently: ask the metric's owner which one to use before writing SQL

Move on when the asker has agreed to a written metric, period and population.

## 2. When you choose the tables

You know what to compute and are looking for where it lives.

Must:
- [ ] Name the grain of every table (source 1)
  Why: Say what one row is, such as one order line; the grain can be a combination of columns, like user plus address.
  How to verify: For each table, the row count equals the count of distinct combinations of its key columns.

Don't forget these too:
- [ ] Use the team's trusted table
  Why: Prefer the documented, modeled table the team already reports from over raw event logs or a colleague's copy.
  How to verify: The table has docs or an owner in the data catalog, and existing reports read from it.
- [ ] Find the latest load date
  Why: Pipelines lag, and the most recent day is often only partly loaded.
  How to verify: The newest date in the table reaches the end of your range, or you note the cutoff.
- [ ] Read what each column means
  Why: Look for units, currency, and whether amounts include tax, discounts or refunds.
  How to verify: Docs or the table's owner confirm the meaning and unit of every column your metric uses.
- [ ] Look for test and deleted rows
  Why: Staff, contractors and QA runs land beside real users, so scan for a test flag, staff email domain, deleted date or status column.
  How to verify: You know which columns mark test, internal and deleted records, or the owner confirmed there are none.
- [ ] Get access before using personal data
  Why: Use only the personal fields the answer needs, and ask the data owner first when they sit outside your usual access.
  How to verify: The query reads no personal fields beyond what the answer needs, and you hold **access for this use**.
- [ ] Run heavy queries off production
  Why: Point large scans at the warehouse or a read replica, or ask the database admin, so the live app stays fast.
  How to verify: Open your connection settings: the host is the warehouse or a read-only replica, not the live app database.

Stop if:
- If nobody can say what a column means: ask the table's owner before you build on it

Move on when you know the grain, freshness and quirks of every table you will use.

## 3. When you write the filters

You are deciding which rows go in: dates, accounts, statuses and missing values.

Must:
- [ ] Use a half-open date range (source 2)
  Why: Write >= the first day and < the day after the last; BETWEEN drops **late timestamps** on the final day.
  How to verify: Rows from the last hour of the final day show up in the result.
- [ ] Pin the reporting time zone
  Why: Storage is often UTC: convert with your database's time zone function to the **business's named zone** before cutting into days.
  How to verify: The query converts to one named zone, such as America/Denver, before taking the date, and a known day's total matches the official figure.
- [ ] Exclude test and internal accounts (only if: Test or internal accounts or Not sure yet)
  Why: Use the shared test flag or exclusion table, not your own list, so every report removes the same accounts.
  How to verify: Known staff, QA and test accounts return zero rows after your filter.

Don't forget these too:
- [ ] Apply the soft-delete marker (only if: Deleted or soft-deleted records or Not sure yet)
  Why: A soft delete only marks the row, so it stays in every count until you filter on the marker as agreed.
  How to verify: Rows carrying a deleted marker are in or out exactly as your definition note says.
- [ ] Decide if deleted records count (only if: Deleted or soft-deleted records or Not sure yet)
  Why: For past periods, an account deleted later was still a **real user then**; agree which view the question needs.
  How to verify: Your definition note says whether records deleted after the period are counted, and why.
- [ ] Count rows each exclusion removes (only if: Test or internal accounts or Deleted or soft-deleted records or Not sure yet)
  Why: Run the count with and without each exclusion so you can say how much it moved the number.
  How to verify: Each exclusion removes a plausible share, and you can quote it if asked.
- [ ] Handle NULLs in exclusion filters (source 3)
  Why: A 'not equal to' filter drops empty rows too, since any comparison with NULL is unknown; add OR col IS NULL to keep them.
  How to verify: Count rows with an empty value before and after the filter; any rows lost were meant to go.
- [ ] Decide which statuses count
  Why: Cancelled, refunded, failed and pending records each need an explicit rule that matches the metric definition.
  How to verify: Every status value in the column appears in your filter as either in or out.
- [ ] Use NOT EXISTS over NOT IN (source 4)
  Why: NOT EXISTS ignores empty values in the exclusion list, while NOT IN returns no rows once that list holds a single empty value.
  How to verify: The exclusion returns rows, and its count matches the same filter written with NOT EXISTS.

Stop if:
- If you cannot tell test accounts from real ones: ask the data engineering team for the marker

Move on when every filter is one you chose on purpose and can explain.

## 4. When you join and aggregate

Rows are filtered and you are combining tables and rolling them up to the answer.

Must:
- [ ] Count the entity the question names
  Why: A customer with three orders is three rows; count **distinct customers** when the question asks about customers.
  How to verify: Plain and distinct counts compared, and the one you kept matches the unit in your definition.

Don't forget these too:
- [ ] Run each step on its own
  Why: Give each step its own named block and look at its rows before adding the next one.
  How to verify: Each step returns the rows and count you expected when run alone.
- [ ] Pick each join type on purpose
  Why: An inner join keeps only matches; use a left join when every row of the main table must stay.
  How to verify: The main table's row count is unchanged after a left join meant to keep everything.
- [ ] Filter left-joined tables in the join (source 5)
  Why: Put conditions on the optional table inside the join condition so unmatched rows from the main table stay.
  How to verify: Main-table rows with no match still appear with empty joined values; the same condition moved to WHERE would drop them.
- [ ] Group by the answer's grain only
  Why: Group by exactly the columns in your output sketch; any extra column splits the totals.
  How to verify: One result row per combination in your sketch, with no repeated keys.
- [ ] Divide summed totals for ratios
  Why: Sum the numerator and the denominator, then divide once, so each group weighs in by its size.
  How to verify: The overall rate equals the total numerator over the total denominator.
- [ ] Cast before dividing integers
  Why: Turn one side into a decimal first: some databases return a whole number when both sides are integers.
  How to verify: Rates show decimal places, not only whole numbers.
- [ ] Decide what an empty value means
  Why: Averages and column counts skip empty values; fill them with zero only when empty truly means zero.
  How to verify: The count of empty values in each measure is noted, and your handling matches what empty means there.
- [ ] Join attributes as of event time (source 6)
  Why: Take region, plan or owner from the type 2 dimension row, one per version, whose effective dates cover the event, not **today's value**.
  How to verify: A customer who changed region last year still shows the old region on older orders, with one dimension row per order.

Stop if:
- If a join changes the row count you expected: stop and find which key repeats before going on

Move on when the query runs and its result has the shape of your output sketch.

## 5. Before you hand over the number

The query returns a result and you are about to send it.

Must:
- [ ] Get the number a second way
  Why: Rebuild it by a different route, such as another table or without the join; **agreement between routes** is strong evidence.
  How to verify: Both routes give the same figure, or you can name the reason for the gap.
- [ ] Send the definition with the number
  - [ ] Date range and time zone
  - [ ] Population and exclusions
  - [ ] Data cutoff
  - [ ] Link to the query
  Why: Put it in one line right under the figure, where anyone who forwards it keeps it.
  How to verify: Your reply states the period, time zone, who is counted, what is excluded and how fresh the data is.

Don't forget these too:
- [ ] Follow a few records by hand
  Why: Pick three or four real records, including an odd one, and trace each to the row it lands in.
  How to verify: Each sampled record appears once, in the group you expect.
- [ ] Plot the result over time
  Why: A quick daily line shows missing days, a partial last day and jumps from a time zone or loading problem.
  How to verify: No gaps, dips or spikes in the line that you cannot explain.
- [ ] Size it against what you know
  Why: Compare with total users, last quarter's figure or the asker's own guess before trusting the decimals.
  How to verify: The number sits in a range you can defend, or you found why it does not.
- [ ] Save the query with the answer
  Why: Keep the exact query that made the sent number; the follow-up question almost always comes.
  How to verify: Your reply links to the saved query that produced the number.
- [ ] Put dates in one place (only if: Me, later on or Teammates or a schedule)
  Why: Set the dates and thresholds once at the top, or as named parameters like :start_date, so a rerun changes them in one spot.
  How to verify: Moving to a new period means editing one place only.
- [ ] Comment why each filter exists (only if: Me, later on or Teammates or a schedule)
  Why: Say which accounts or statuses are left out and the reason, not just what the line does.
  How to verify: A colleague can read the comments and explain every filter without asking you.
- [ ] Store it in the shared library (only if: Teammates or a schedule)
  Why: Keep it in the team's repository or saved-query library with a named owner, not in your editor history.
  How to verify: A teammate can open the current version from a link, and it names an owner.
- [ ] Breakdowns add up to the total
  Why: Sum the split rows and compare them with the overall figure; a customer who sits in two segments is a common, **explainable gap**.
  How to verify: The segment rows sum to the total, or you can name the overlap that explains the difference.
- [ ] Have a peer review high-stakes SQL
  Why: For a number going to the board, regulators, public reporting or a big money decision, a second reader catches what you stopped seeing.
  How to verify: A colleague has read the query and the definition note and signed off before the number goes out.

Stop if:
- If the two routes disagree without a reason: hold the number and tell the asker
- If it contradicts an official report: tell the asker and the report's owner before sharing it

Move on when the number has matched a second route and goes out with its definition.

Not covered: Wrong or duplicate rows after joins, cleaning raw exports and speeding up slow queries have their own checklists. Charting and presenting the result are covered separately too.

Last reviewed: 2026-10-12

Sources:
1. dbt Labs: A guide to data grain: https://www.getdbt.com/blog/guide-to-data-grain
2. SQLblog: What do BETWEEN and the devil have in common?: https://sqlblog.org/2011/10/19/what-do-between-and-the-devil-have-in-common
3. Modern SQL: Three-valued logic: https://modern-sql.com/concept/three-valued-logic
4. Red Gate: Consider using NOT EXISTS instead of NOT IN: https://www.red-gate.com/hub/product-learning/sql-prompt/consider-using-not-exists-instead-not-subquery
5. IDUG: ON clause vs WHERE clause filtering predicates in joins: https://www.idug.org/news/on-clause-vs-where-clause-filtering-predicates-in-joins
6. Kimball Group: Slowly Changing Dimensions Are Not Always Easy: https://www.kimballgroup.com/2005/03/slowly-changing-dimensions-are-not-always-as-easy-as-1-2-3/

From CheatGrid: https://www.cheatgrid.com/checklists/data-analyst/querying-data-for-a-business-question?utm_source=github&utm_medium=listing&utm_campaign=querying-data-for-a-business-question
