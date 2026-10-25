# Building a feature end to end

When: You pick up a feature that needs a new screen, new API endpoints and new tables, and you are about to start.

How to use: Open this when you pick up a feature that needs a new screen, new endpoints and new tables. Work through the moments in order and answer the question first so only your checks show. Each check says what to do; 'How to verify' says what to look at and what right looks like.

Before you start: Which of these does your feature have? (any that apply) Changes data people rely on / Access rules beyond signing in / A step slower than a request / Adds columns to existing tables / None of these. Checks marked "only if" apply only to that answer.

## 1. Before you write any code

You have the feature request and nothing is built yet.

Must:
- [ ] Trace each action end to end
  Why: Follow it until every side effect is done: emails, jobs and cache updates, not just the first write.
  How to verify: In your plan, each user action names its endpoint, the tables it writes and **every side effect** that follows.
- [ ] Decide who may see each record
  Why: Signing in proves who the user is, not that a record is theirs; missing that check is broken object level authorization (BOLA).
  How to verify: Each table in the plan names the **owner column** or rule that decides who can read and change its rows.

Don't forget these too:
- [ ] Build a thin slice first
  Why: Take one field through screen, API and table first, so the wiring works before you widen it.
  How to verify: The first working version saves and shows one real value through all three layers.
- [ ] List every state of the screen
  Why: Each state, such as loading, empty, error, success or no access, needs its own design, not only the happy path.
  How to verify: The plan or mockup shows each state, including **the empty screen** a brand new user sees.
- [ ] Work backward from the screen's data
  Why: Write down the fields the screen shows and edits, then map each to a column or a calculation.
  How to verify: Every field on the mockup traces to one column, one join or one computed value.
- [ ] Agree the request and response shapes
  Why: Write them down before front and back work split, so neither side guesses.
  How to verify: A shared type or written example exists for each endpoint, including its **error shape**, ideally the standard Problem Details format with field names.
- [ ] Decide what happens on delete (source 1)
  Why: Cascade a child that cannot exist without its parent; for independent records, block the delete or keep them with an empty link.
  How to verify: Every new foreign key has a written on-delete rule that matches what the user expects to lose.
- [ ] Write permissions as a table (only if: Access rules beyond signing in)
  Why: Rows are roles, columns are actions such as view, create, edit and delete, so gaps show at a glance.
  How to verify: Every cell is filled, including what a **removed member** or a guest can still do.
- [ ] Plan the change to existing data (only if: Changes data people rely on or Adds columns to existing tables)
  Why: List which existing rows and older app versions read what you change; parallel change (expand, migrate, contract) keeps each working.
  How to verify: The plan names the backfill, its default values and what older clients see while it runs.
- [ ] Mark slow steps as jobs (source 2) (only if: A step slower than a request)
  Why: Slow outside calls, bulk emails, file processing, retries and scheduled tasks run outside the request, or the user waits.
  How to verify: Each step expected to take more than **a few seconds** is marked as a background job in the plan.
- [ ] Put the feature behind a flag (source 3)
  Why: Small pieces can merge daily while the feature stays hidden until the whole flow works.
  How to verify: Turn the flag off and open the existing screens and endpoints: they behave exactly as before.
- [ ] Copy how the codebase does it
  Why: Find a similar feature already shipped and follow its naming, folder layout, error format and access checks.
  How to verify: Your plan names **the existing feature** you are following, and new files sit where its files sit.

Stop if:
- If nobody can say who may see a record: stop and ask the product owner before building

Move on when A short plan names the flow, endpoints, tables, screen states and access rule.

## 2. When you change the database

You are writing the migrations for the new tables and columns.

Must:
- [ ] Put the rules in the database (source 1)
  Why: Not-null, unique and foreign key constraints hold even when a script or bug skips the API; most columns should be not null.
  How to verify: Insert a bad row by hand in your local database: the database **rejects it** with a constraint error.
- [ ] Add new columns nullable or defaulted (only if: Changes data people rely on or Adds columns to existing tables)
  Why: Add it nullable or with a constant default, backfill, then enforce not null as a separate step your database runs without a long lock.
  How to verify: The migration runs on a copy with real rows, and **no step** rewrites the whole table under a lock.

Don't forget these too:
- [ ] Backfill existing rows in batches (only if: Changes data people rely on or Adds columns to existing tables)
  Why: Small batches keep locks short and let you stop halfway if something looks wrong.
  How to verify: A count of rows still missing the value reaches zero before the required constraint goes on.
- [ ] Index what new queries filter on
  Why: Foreign keys and the columns in your filters and sort order usually need one.
  How to verify: The query plan for each new query shows an **index scan**, not a full table scan, on realistic data.
- [ ] Write the down migration too
  Why: Rolling back code is easy; rolling back a schema needs a step written in advance.
  How to verify: Run up, down and up again locally: it ends in the same schema with no errors.
- [ ] Store times in UTC (source 4)
  Why: Convert to local time only on screen; PostgreSQL's timestamp with time zone stores UTC and drops the original zone, so keep that separately.
  How to verify: New timestamp columns use a time zone aware type, and stored values read the same from any server.

Stop if:
- If a migration would lock or rewrite a large live table: stop and ask whoever owns the database

Move on when migrations run up and down cleanly on a copy with realistic data.

## 3. When you build the endpoints

The tables exist and you are writing the API the screen will call.

Must:
- [ ] Scope every query to the user (source 5)
  Why: Look records up by **id and owner** in one query, never by id alone, even when ids are random.
  How to verify: Call the endpoint with a valid token and another user's record id: it answers not found or forbidden.
- [ ] Validate every input on the server (source 6)
  Why: Form checks help the user; the server's check is the one that protects the data.
  How to verify: Send a request with bad values straight to the API: it refuses it with a **field-level error**, never silently cleaning and saving it.

Don't forget these too:
- [ ] Share validation rules with the form
  Why: One schema used by both sides keeps lengths, formats and required fields from drifting apart.
  How to verify: Change one limit in one place: both the form and the API enforce the new value.
- [ ] Accept only fields users may set (source 7)
  Why: Bind requests to an input object (DTO) listing only editable fields; this blocks mass assignment of fields like owner id or role.
  How to verify: Add an extra field like an owner id to a request: the saved row **ignores it**.
- [ ] Make create requests safe to retry (source 8)
  Why: Send a unique key with each submit and store the first response, so a retry with that key returns it unchanged.
  How to verify: Send the same create request twice with one key: **one row** is saved and both responses match.
- [ ] Wrap related writes in one transaction (source 9)
  Why: Writes to several tables for one action either all land or none do, the guarantee called atomic commit.
  How to verify: Force an error after the first write: no partial rows remain afterwards.
- [ ] Page every list endpoint (source 10)
  Why: Set a default and a maximum page size so one request never returns the whole table.
  How to verify: Ask for a huge page size: the response stops at the maximum and includes a way to fetch the next page.
- [ ] Enforce role rules on the server (only if: Access rules beyond signing in)
  Why: Check the action itself and deny by default, since hiding a button stops nobody from calling the API.
  How to verify: Call each restricted endpoint as each role: responses match the **permission table** cell for cell.
- [ ] Return a job id fast (only if: A step slower than a request)
  Why: Answer straight away with an id, then let the screen poll the status or get notified: the long running operation (LRO) pattern.
  How to verify: The request returns in normal response time, and the status endpoint shows the job moving to done.
- [ ] Make jobs safe to run twice (source 11) (only if: A step slower than a request)
  Why: Queues retry after crashes, so a job that refunds a charge and then emails must first check the refund did not already happen.
  How to verify: Run the same job twice: the result matches **one run**, with no duplicate emails or rows.
- [ ] Keep internals out of error responses (source 12)
  Why: Log the full error on the server and send the client a short message and a request id, since queries and paths help attackers.
  How to verify: Force a server error: the response shows **no stack trace**, query or file path.
- [ ] Return only needed fields
  Why: Build each response from a named list of fields, never by sending the whole database row.
  How to verify: Read a raw response: it holds no password hashes, internal flags or other **fields the screen never shows**.

Move on when every endpoint refuses bad input and other users' records when called directly.

## 4. When you build the screen

The API answers and you are wiring the new screen to it.

Must:
- [ ] Block double submits
  Why: Disable the button and show progress the moment the request starts, until it answers.
  How to verify: With the network throttled, click submit several times fast: **one request** leaves the browser.

Don't forget these too:
- [ ] Show each state you planned
  - [ ] Loading
  - [ ] Empty
  - [ ] Error
  - [ ] Success
  - [ ] No access
  How to verify: Force each state in the browser: each shows a message and a next step, never a blank area.
- [ ] Keep input after an error (source 13)
  Why: Show the server's message next to its field and in a summary at the top, and keep everything the user typed, even failing answers.
  How to verify: Submit a form the server rejects: the fields keep their values and each error sits by **the right field**.
- [ ] Hide actions the user cannot take (only if: Access rules beyond signing in)
  Why: It spares confusion; the server still makes the real decision.
  How to verify: Signed in as each role, the screen shows only the actions that role's permission cells allow.
- [ ] Show progress for long jobs (only if: A step slower than a request)
  Why: Tell the user it is running and let them leave and come back; with no visible change, users assume it failed and retry.
  How to verify: Start a job and refresh the page: the screen still shows its status and later the result.
- [ ] Update every view after a save (source 14)
  Why: Refetch each query that shows the changed record, or write the saved response into the cache to skip a round trip.
  How to verify: Save, then open the list and the detail view: both show the new value **without a reload**.
- [ ] Make the flow keyboard accessible
  Why: Label every input and move focus to the first error after a failed submit, so keyboard and screen reader users can recover.
  How to verify: Finish the whole flow with the keyboard alone, and a screen reader names every field.
- [ ] Check the screen at phone width
  Why: Narrow the browser to a phone size and watch long text, tables and buttons.
  How to verify: At phone width every control is reachable and **nothing scrolls sideways**.

Stop if:
- If the screen needs data no endpoint returns: stop and agree the change with the API owner

Move on when every planned state shows correctly on a throttled network.

## 5. Before you call it done

Everything is wired together and you think the feature works.

Must:
- [ ] Walk the flow as a user
  Why: Start from a fresh account and realistic data, not the records you built with.
  How to verify: A new user finishes the flow, and **every side effect** on the plan, emails and jobs included, happened once.
- [ ] Try it as someone else
  Why: Use a second account and a signed-out browser on every new URL and endpoint, hunting horizontal privilege escalation: reaching another user's data.
  How to verify: Every new page and call, tried with another account's ids, shows **no foreign data**.

Don't forget these too:
- [ ] Count the queries per page load
  Why: Look for one query per row, the N plus one pattern, batch it into a join, then pin the query count in a test.
  How to verify: The query log for one page load shows a fixed number of queries, whatever the list length.
- [ ] Refresh and go back mid-flow
  Why: Reload, press Back and open the page in a second tab at each step.
  How to verify: Each reload lands on a sensible state, and **no step saves twice**.
- [ ] Delete a parent and check children
  How to verify: After deleting a parent, its child rows are gone, blocked or kept exactly as the on-delete rule says.
- [ ] Read the logs from your walk
  Why: Warnings during a clean walk usually point at a real bug; the DevTools console shows the call stack under each error.
  How to verify: Server and browser consoles show no new errors or warnings after the full walk.
- [ ] Keep secrets out of the logs
  Why: The event and a request id are enough to trace a problem without any secrets.
  How to verify: Search the logs from your walk for the test password and email: **neither appears**.

Stop if:
- If any step needs a manual database fix to finish: stop and talk to your lead before release

Move on when A fresh user and a second account both walk the flow with no surprises.

Not covered: Table design in depth, the REST contract on its own, choosing tests and deploy order between old and new versions each have their own checklist in this roadmap. Login and token handling live in Adding user login.

Last reviewed: 2026-10-12

Sources:
1. PostgreSQL Docs: Constraints, foreign key ON DELETE: https://www.postgresql.org/docs/current/ddl-constraints.html
2. Vercel: How to run background jobs in Next.js: https://vercel.com/kb/guide/how-to-run-background-jobs-in-nextjs-on-vercel
3. Martin Fowler: Feature Toggles: https://www.martinfowler.com/articles/feature-toggles.html
4. PostgreSQL Docs: Date/Time Types: https://www.postgresql.org/docs/current/datatype-datetime.html
5. OWASP: Authorization Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html
6. OWASP: Input Validation Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html
7. OWASP: Mass Assignment Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Mass_Assignment_Cheat_Sheet.html
8. Stripe: Idempotent requests API reference: https://docs.stripe.com/api/idempotent_requests
9. SQLite: Atomic Commit In SQLite: https://www.sqlite.org/atomiccommit.html
10. Django REST framework: Pagination: https://www.django-rest-framework.org/api-guide/pagination/
11. Sidekiq: Best Practices, idempotent jobs: https://sidekiq.org/wiki/Best-Practices
12. MITRE CWE-209: Error message information exposure: https://cwe.mitre.org/data/definitions/209.html
13. GOV.UK Design System: Error message component: https://design-system.service.gov.uk/components/error-message/
14. TkDodo: Automatic Query Invalidation after Mutations: https://tkdodo.eu/blog/automatic-query-invalidation-after-mutations

From CheatGrid: https://www.cheatgrid.com/checklists/full-stack-developer/building-a-feature-end-to-end?utm_source=github&utm_medium=listing&utm_campaign=building-a-feature-end-to-end
