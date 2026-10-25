## Building a feature end to end

When: You pick up a feature that needs a new screen, new API endpoints and new tables, and you are about to start.

Before you start: Which of these does your feature have? (any that apply) Changes data people rely on / Access rules beyond signing in / A step slower than a request / Adds columns to existing tables / None of these. Checks marked "only if" apply only to that answer.

### 1. Before you write any code

You have the feature request and nothing is built yet.

Must:
- [ ] Trace each action end to end
- [ ] Decide who may see each record

Stop if:
- If nobody can say who may see a record: stop and ask the product owner before building

Move on when A short plan names the flow, endpoints, tables, screen states and access rule.

### 2. When you change the database

You are writing the migrations for the new tables and columns.

Must:
- [ ] Put the rules in the database
- [ ] Add new columns nullable or defaulted (only if: Changes data people rely on or Adds columns to existing tables)

Stop if:
- If a migration would lock or rewrite a large live table: stop and ask whoever owns the database

Move on when migrations run up and down cleanly on a copy with realistic data.

### 3. When you build the endpoints

The tables exist and you are writing the API the screen will call.

Must:
- [ ] Scope every query to the user
- [ ] Validate every input on the server

Move on when every endpoint refuses bad input and other users' records when called directly.

### 4. When you build the screen

The API answers and you are wiring the new screen to it.

Must:
- [ ] Block double submits

Stop if:
- If the screen needs data no endpoint returns: stop and agree the change with the API owner

Move on when every planned state shows correctly on a throttled network.

### 5. Before you call it done

Everything is wired together and you think the feature works.

Must:
- [ ] Walk the flow as a user
- [ ] Try it as someone else

Stop if:
- If any step needs a manual database fix to finish: stop and talk to your lead before release

Move on when A fresh user and a second account both walk the flow with no surprises.

---
Must checks only. Why each check matters, how to verify it and its source: [full checklist](https://github.com/michaelmiscanuk/dev-checklist-templates/blob/main/checklists/building-a-feature-end-to-end.md). Interactive version: [Building a feature end to end on CheatGrid](https://www.cheatgrid.com/checklists/full-stack-developer/building-a-feature-end-to-end?utm_source=github&utm_medium=listing&utm_campaign=building-a-feature-end-to-end).
