---
name: Threat modeling a new feature
about: Before building a feature that touches logins, payments, personal data, file handling or a new outside integration.
title: "Threat model: "
labels: ["security"]
---
## Threat modeling a new feature

When: Before building a feature that touches logins, payments, personal data, file handling or a new outside integration.

Before you start: What does this feature touch? (any that apply) Input from outside people / Calls to or from services / Personal data / Payment data / File uploads or downloads / None of these. Checks marked "only if" apply only to that answer.

### 1. Before you sit down to model

The feature is designed on paper, nothing is built yet, and you are setting up the threat modeling session.

Must:
- [ ] Model while the design can change

Don't forget these too:
- [ ] Write the scope in one line
- [ ] Invite the people who build it
- [ ] List what an attacker would want
- [ ] Start from the existing model

Move on when the scope, the asset list and the people are fixed.

### 2. Draw what you are building

The team is at a whiteboard or shared diagram, turning the design into a picture of data moving.

Must:
- [ ] Draw a data flow diagram

Don't forget these too:
- [ ] Draw the side doors too
- [ ] Record how senders are known
- [ ] Walk one real request through
- [ ] Draw outside services as outside (only if: Calls to or from services)
- [ ] Follow each file end to end (only if: File uploads or downloads)
- [ ] Mark every copy of sensitive data (only if: Personal data or Payment data)
- [ ] Draw every trust boundary

Stop if:
- If nobody present can explain a part's data flow: stop and bring in its designer
- If a route skips the login or a trust boundary: tell the tech lead before going on

Move on when the builders agree the diagram matches the plan, with every entry point, store and boundary on it.

### 3. Find what can go wrong

The diagram is up and the team walks it piece by piece, asking how each part could be abused.

Must:
- [ ] Ask STRIDE at every boundary
  - [ ] Spoofing: pretending to be someone else
  - [ ] Tampering: changing data in transit or at rest
  - [ ] Repudiation: denying an action afterwards
  - [ ] Information disclosure: seeing what you should not
  - [ ] Denial of service: making it unavailable
  - [ ] Elevation of privilege: gaining more rights

Don't forget these too:
- [ ] Write each threat as a sentence
- [ ] Write abuse cases beside stories
- [ ] Find the expensive operations
- [ ] Ask what you could prove later
- [ ] Ask what a script could do (only if: Input from outside people or Personal data)
- [ ] Compare replies for unknown accounts (only if: Input from outside people or Personal data)
- [ ] Ask how an account changes hands
- [ ] Try each action as another user

Stop if:
- If the list outgrows the session: finish the boundary crossings first and book a second session

Move on when every element and boundary crossing has been asked its STRIDE categories, and each threat is a full sentence.

### 4. Go deeper where the risk sits

The general pass is done; now take the parts of your feature that carry extra risk one at a time.

Must:
- [ ] Picture one stolen admin account
- [ ] Grant access from the verified webhook (only if: Payment data)

Don't forget these too:
- [ ] Treat partner data as untrusted (only if: Calls to or from services)
- [ ] Ask how a callback proves itself (only if: Calls to or from services or Payment data)
- [ ] Ask where outbound calls can point (only if: Calls to or from services)
- [ ] Run a privacy pass (only if: Personal data)
- [ ] Ask who sets the amount (only if: Payment data)
- [ ] Ask what a double request does (only if: Payment data)
- [ ] Ask what a hostile file does (only if: File uploads or downloads)
- [ ] Plan for partner outages (only if: Calls to or from services)
- [ ] Ask what can steer a model

Stop if:
- If one operator can export every tenant's data alone: take it to your security team now

Move on when each risky part you picked in the question at the top has its own threats written down.

### 5. Decide what to do about each

The threat list is written and the team decides, threat by threat, what changes in the design.

Must:
- [ ] Pick a response for every threat
  - [ ] Reduce
  - [ ] Eliminate
  - [ ] Transfer
  - [ ] Accept
- [ ] Turn each fix into a ticket

Don't forget these too:
- [ ] Rank threats by likelihood and impact
- [ ] Get accepted risks signed off
- [ ] Scope and log operator access
- [ ] Keep card numbers off your servers (only if: Payment data)
- [ ] Serve uploads from a separate domain (only if: File uploads or downloads)
- [ ] Give each integration least access (only if: Calls to or from services)
- [ ] Keep keys in a secret store (only if: Calls to or from services)
- [ ] Name the rules for your data (only if: Personal data or Payment data)

Stop if:
- If a high threat has no fix and no one accepts it: hold launch, call the security lead

Move on when every threat has a response, a rank and, for fixes, a ticket with an owner.

### 6. Before you ship, and on every change (every time)

The feature is built or nearly so, and you check the model against what actually exists.

Must:
- [ ] Test each fix as an attack

Don't forget these too:
- [ ] Reopen it when the design changes

Stop if:
- If the built feature has an entry point the model never saw: model it before release

Move on when the model matches the built feature and every high threat has a passing test.

---
Why each check matters, how to verify it and its source: [full checklist](https://github.com/michaelmiscanuk/dev-checklist-templates/blob/main/checklists/threat-modeling-a-new-feature.md). Interactive version: [Threat modeling a new feature on CheatGrid](https://www.cheatgrid.com/checklists/web-security-essentials/threat-modeling-a-new-feature?utm_source=github&utm_medium=listing&utm_campaign=threat-modeling-a-new-feature).
