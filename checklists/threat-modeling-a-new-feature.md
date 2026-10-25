# Threat modeling a new feature

When: Before building a feature that touches logins, payments, personal data, file handling or a new outside integration.

How to use: Use this before building a feature that touches sign-in, payments, personal data, files or an outside service. Work through the moments in order, ideally in one session with the people who will build it. Answer the question first so the checks for your feature's risky parts appear. Each check's 'How to verify' says what to look at to know it is done.

Before you start: What does this feature touch? (any that apply) Input from outside people / Calls to or from services / Personal data / Payment data / File uploads or downloads / None of these. Checks marked "only if" apply only to that answer.

## 1. Before you sit down to model

The feature is designed on paper, nothing is built yet, and you are setting up the threat modeling session.

Must:
- [ ] Model while the design can change (source 1)
  Why: Hold the session once the design is sketched and before code, when a finding costs a whiteboard edit.
  How to verify: Compare the session date with the build plan: it comes **before implementation starts** on the parts it covers.

Don't forget these too:
- [ ] Write the scope in one line
  Why: Name the feature, its entry points and what you treat as given, such as the shared login service.
  How to verify: Someone outside the team reads the scope line and can say which components are in and which are out.
- [ ] Invite the people who build it (source 2)
  Why: Bring the engineers who will write the code, the product owner and testers; a security specialist helps but need not block the session.
  How to verify: Look at the invite: at least one person who will **write the code** is on it.
- [ ] List what an attacker would want
  Why: Name the assets first, such as accounts, money, personal data, admin rights and uptime; threats cluster around them.
  How to verify: The notes hold a short asset list, each one something an attacker gains or you lose.
- [ ] Start from the existing model
  Why: Pull in the system's earlier threat model, past incidents and test findings for this area, then extend them.
  How to verify: The notes link to the earlier model and findings, or say plainly there are none.

Move on when the scope, the asset list and the people are fixed.

## 2. Draw what you are building

The team is at a whiteboard or shared diagram, turning the design into a picture of data moving.

Must:
- [ ] Draw a data flow diagram (source 3)
  Why: Use only these shapes: outside actors such as users, processes, data stores such as caches or databases, data flows and trust boundaries.
  How to verify: Every arrow names the data it carries, and every box is one of those kinds.

Don't forget these too:
- [ ] Draw the side doors too
  Why: Add admin and support tools, debug routes, batch jobs and scripts: they reach the same data by another path.
  How to verify: The diagram shows how operators reach the data, and every debug or test route is drawn or confirmed removed.
- [ ] Record how senders are known
  Why: Next to each flow into a process, write how the sender is known: a session, a token, a signature, or nothing.
  How to verify: No flow into a process is left blank, and 'nothing' is written where that is the truth.
- [ ] Walk one real request through
  Why: Trace one real request, such as a password reset, from click to database and back; vague steps show gaps in the diagram.
  How to verify: A builder walked a request through the diagram without saying 'somehow' or 'I think'.
- [ ] Draw outside services as outside (only if: Calls to or from services)
  Why: Place partner APIs, webhooks, SDKs and queues beyond your boundary, even when the vendor is well known.
  How to verify: Every outside service sits across a trust boundary, with each call out and each callback in drawn as its own flow.
- [ ] Follow each file end to end (only if: File uploads or downloads)
  Why: Each uploaded file passes through storage and any processing, such as thumbnails or parsing, before it is served back.
  How to verify: The diagram shows the storage, every processor that opens the file and the route that serves it back.
- [ ] Mark every copy of sensitive data (only if: Personal data or Payment data)
  Why: Follow personal and payment fields into logs, caches, analytics, backups and exports, not only the main database.
  How to verify: Each sensitive field appears in every store that holds it, logs and exports included.
- [ ] Draw every trust boundary (source 4)
  Why: Put a boundary wherever data passes between levels of trust: browser to server, service to partner, app to database.
  How to verify: Each flow from a less trusted part to a more trusted one is cut by a **dashed boundary line**.

Stop if:
- If nobody present can explain a part's data flow: stop and bring in its designer
- If a route skips the login or a trust boundary: tell the tech lead before going on

Move on when the builders agree the diagram matches the plan, with every entry point, store and boundary on it.

## 3. Find what can go wrong

The diagram is up and the team walks it piece by piece, asking how each part could be abused.

Must:
- [ ] Ask STRIDE at every boundary (source 5)
  - [ ] Spoofing: pretending to be someone else
  - [ ] Tampering: changing data in transit or at rest
  - [ ] Repudiation: denying an action afterwards
  - [ ] Information disclosure: seeing what you should not
  - [ ] Denial of service: making it unavailable
  - [ ] Elevation of privilege: gaining more rights
  Why: STRIDE means spoofing, tampering, repudiation, information disclosure, denial of service and elevation of privilege; spend most time where flows cross a boundary.
  How to verify: Every flow and store was asked tampering, disclosure and denial of service, every process all six, and each threat names its element.

Don't forget these too:
- [ ] Write each threat as a sentence
  Why: Use the shape 'an attacker who has X can do Y to Z'; it makes the fix and its test obvious.
  How to verify: Each entry names the attacker, what they do and which asset it hits.
- [ ] Write abuse cases beside stories (source 6)
  Why: For each story, ask how a fraudster, a curious customer or a careless insider would misuse it, such as editing a price.
  How to verify: Each main user story has at least one matching abuse case.
- [ ] Find the expensive operations (source 7)
  Why: Searches, exports, reports and file processing cost far more than a page view; note who can start them.
  How to verify: The list names each costly operation and who can trigger it, with entries for denial of service and for the cloud bill.
- [ ] Ask what you could prove later (source 8)
  Why: For each sensitive action, ask whether logs would show who did it and when, while keeping secrets out of them.
  How to verify: Changes to money, roles and data exports each have a repudiation entry or a logging decision naming who, what, target and outcome.
- [ ] Ask what a script could do (only if: Input from outside people or Personal data)
  Why: Reset, code delivery, signup and export routes are cheap to call thousands of times for enumeration or credential stuffing.
  How to verify: Each public route has an entry for bulk abuse, naming what the attacker gains.
- [ ] Compare replies for unknown accounts (source 9) (only if: Input from outside people or Personal data)
  Why: Different messages, status codes or timing for known and unknown emails tell an attacker who has an account; signup counts too.
  How to verify: The list covers account enumeration on sign-in, signup and reset, or notes the feature has none.
- [ ] Ask how an account changes hands
  Why: Walk password reset, email change and account recovery, and ask whether each asks for the **current password** or second factor.
  How to verify: The list has an entry for each route that can change who controls an account, including sessions and reset links left open afterwards.
- [ ] Try each action as another user (source 10)
  Why: For every read, change, delete and export, ask what happens when someone swaps in another user's or tenant's id, even an unguessable one.
  How to verify: The list has an entry on reaching other users' objects, or a line saying why that cannot happen.

Stop if:
- If the list outgrows the session: finish the boundary crossings first and book a second session

Move on when every element and boundary crossing has been asked its STRIDE categories, and each threat is a full sentence.

## 4. Go deeper where the risk sits

The general pass is done; now take the parts of your feature that carry extra risk one at a time.

Must:
- [ ] Picture one stolen admin account
  Why: Trace what a single support or admin login could read, change, export or delete across every tenant.
  How to verify: The list says what one operator account can reach, and bulk actions across tenants have an entry.
- [ ] Grant access from the verified webhook (source 11) (only if: Payment data)
  Why: Give access to what was paid for only when the processor's **signed event** arrives, never because the browser reached the success page.
  How to verify: Each payment flow has an entry for a faked or never-loaded success page, and the design grants access from the verified event.

Don't forget these too:
- [ ] Treat partner data as untrusted (source 12) (only if: Calls to or from services)
  Why: Data arriving from a partner API or webhook gets the same validation as form input; partners get breached too.
  How to verify: Each inbound partner flow has an entry for malformed or hostile data, oversized replies and followed redirects.
- [ ] Ask how a callback proves itself (source 13) (only if: Calls to or from services or Payment data)
  Why: Signed payloads with a timestamp let the receiver turn away forged and replayed calls.
  How to verify: Each inbound callback has threats for forgery and replay, a maximum age for timestamps, and skips event ids already processed.
- [ ] Ask where outbound calls can point (only if: Calls to or from services)
  Why: When any part of a fetched address comes from a user, the server can be steered inside your network, called server-side request forgery (SSRF).
  How to verify: Each outbound call lists where its address comes from, and user-influenced ones have an SSRF entry.
- [ ] Run a privacy pass (only if: Personal data)
  Why: LINDDUN, a privacy threat framework, adds what STRIDE misses: linking records, identifying people and keeping data too long.
  How to verify: The list covers who can see each personal field, how long it is kept, and what leaves in exports.
- [ ] Ask who sets the amount (only if: Payment data)
  Why: Prices, quantities, discounts and currency from the client can be edited; plan for the server to work them out.
  How to verify: Each payment flow has an entry on client-supplied amounts, and the design recalculates them on the server.
- [ ] Ask what a double request does (source 14) (only if: Payment data)
  Why: Retries, double clicks and replayed callbacks arrive together; an idempotency key makes each money step run once.
  How to verify: Every step that moves money has an entry for repeated and simultaneous requests, and a retry with the same key returns the first result.
- [ ] Ask what a hostile file does (only if: File uploads or downloads)
  Why: Think of huge files, archives that expand, scripts dressed as images and filenames with path tricks.
  How to verify: Each processor that opens a file, and the download route, has at least one threat entry.
- [ ] Plan for partner outages (only if: Calls to or from services)
  Why: Set a timeout for each outside call, and have sign-in and payment checks **fail closed**, refusing the request rather than letting it through.
  How to verify: Each outside call has a timeout and an outage behaviour written down, with sign-in and payment ones refusing.
- [ ] Ask what can steer a model (source 15)
  Why: If a model reads your content, any text in it can carry instructions, called **prompt injection**; treat its output as untrusted.
  How to verify: Each model call has an entry for injected instructions and for what its output is allowed to trigger.

Stop if:
- If one operator can export every tenant's data alone: take it to your security team now

Move on when each risky part you picked in the question at the top has its own threats written down.

## 5. Decide what to do about each

The threat list is written and the team decides, threat by threat, what changes in the design.

Must:
- [ ] Pick a response for every threat
  - [ ] Reduce
  - [ ] Eliminate
  - [ ] Transfer
  - [ ] Accept
  Why: Choose one: fix it, remove that part, hand it to someone else such as a payment processor, or accept it.
  How to verify: No threat on the list is left **without a response**.
- [ ] Turn each fix into a ticket
  Why: Put each fix in the same tracker as the feature work, with an owner, so it ships with the feature.
  How to verify: Every fix links to a ticket with an owner and a target release.

Don't forget these too:
- [ ] Rank threats by likelihood and impact (source 16)
  Why: Plain high, medium and low is enough; the team agreeing on the order matters more than the scoring scheme.
  How to verify: Every threat has a rank using the worst-case attacker, and each high one has a fix planned before launch.
- [ ] Get accepted risks signed off
  Why: Have a person with the authority to accept the risk, often the product owner, name themselves and a review date.
  How to verify: Each accepted risk shows who accepted it and when it will be looked at again.
- [ ] Scope and log operator access
  Why: Limit support accounts to the tenant they are helping, ask for a second person on bulk exports, and log every action.
  How to verify: The design shows tenant scoping, a second approval for bulk actions and an audit log for operators.
- [ ] Keep card numbers off your servers (source 17) (only if: Payment data)
  Why: Let the payment processor's hosted page or fields take card details, which shrinks what you must protect.
  How to verify: Every part of the payment form loads straight from the processor, and card numbers never touch your servers or logs.
- [ ] Serve uploads from a separate domain (only if: File uploads or downloads)
  Why: A separate domain plus a download header keeps an uploaded file from running scripts as your app.
  How to verify: Uploaded files come from a different domain than the app, with types checked by content, not by name.
- [ ] Give each integration least access (only if: Calls to or from services)
  Why: Grant each partner only the scopes and endpoints it needs, and use narrowly scoped keys of your own towards it.
  How to verify: Each integration lists its scopes both ways, and none holds broad admin rights.
- [ ] Keep keys in a secret store (source 18) (only if: Calls to or from services)
  Why: Store partner keys and tokens in a secret manager such as Vault or AWS Secrets Manager, never in the repo or browser code.
  How to verify: Each partner key and token the feature uses has its place in the **secret manager** and an owner for rotation.
- [ ] Name the rules for your data (only if: Personal data or Payment data)
  Why: GDPR and CCPA follow where users live, PCI DSS follows card data; bring in your **privacy or compliance lead** before the design sets.
  How to verify: The notes name each rule that applies, any data processing agreement a partner needs, and who from legal or compliance reviewed the design.

Stop if:
- If a high threat has no fix and no one accepts it: hold launch, call the security lead

Move on when every threat has a response, a rank and, for fixes, a ticket with an owner.

## 6. Before you ship, and on every change (every time)

The feature is built or nearly so, and you check the model against what actually exists.

Must:
- [ ] Test each fix as an attack
  Why: Write a test that tries the threat itself, such as fetching another user's record, not one that only checks the fix exists.
  How to verify: Each high threat has a passing test or a recorded manual check against the built feature.

Don't forget these too:
- [ ] Reopen it when the design changes (source 1)
  Why: Keep the model in the repo or design doc, and revisit it when a new entry point, store, partner or user type arrives.
  How to verify: The pull request template or change checklist asks whether the threat model needs an update.

Stop if:
- If the built feature has an entry point the model never saw: model it before release

Move on when the model matches the built feature and every high threat has a passing test.

Not covered: It does not cover company-wide risk assessments, penetration testing or how to build each fix; injection, sessions, authorization code and rate limiting have their own checklists in this roadmap.

Last reviewed: 2026-10-13

Sources:
1. Threat Modeling Manifesto: Principles: https://www.threatmodelingmanifesto.org/
2. Martin Fowler: Agile Threat Modelling: https://martinfowler.com/articles/agile-threat-modelling.html
3. Microsoft Learn: Threat Modeling Tool feature overview: https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-feature-overview
4. Shostack: Trust and security boundaries: https://shostack.org/blog/trust-and-security-boundaries/
5. Shostack: Uncover Security Design Flaws Using STRIDE: https://shostack.org/files/essays/uncover
6. OWASP: Abuse Case Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Abuse_Case_Cheat_Sheet.html
7. OWASP: API4:2023 Unrestricted Resource Consumption: https://owasp.org/API-Security/editions/2023/en/0xa4-unrestricted-resource-consumption/
8. OWASP: Logging Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html
9. OWASP: Authentication Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
10. OWASP: Insecure Direct Object Reference Prevention Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html
11. Stripe Docs: Fulfill orders after Checkout: https://docs.stripe.com/checkout/fulfillment?payment-ui=checkout-form
12. OWASP API Security Top 10: Unsafe Consumption of APIs: https://owasp.org/API-Security/editions/2023/en/0xaa-unsafe-consumption-of-apis/
13. Stripe Docs: Webhooks, signatures and replay: https://docs.stripe.com/webhooks?lang=node
14. Stripe Docs: Idempotent requests: https://docs.stripe.com/api/idempotent_requests
15. OWASP GenAI: LLM05 Improper Output Handling: https://genai.owasp.org/llmrisk/llm052025-improper-output-handling/
16. OWASP: Risk Rating Methodology: https://owasp.org/www-community/OWASP_Risk_Rating_Methodology
17. PCI SSC: SAQ A, PCI DSS v4.0: https://listings.pcisecuritystandards.org/documents/PCI-DSS-v4-0-SAQ-A.pdf
18. OWASP: Secrets Management Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html

From CheatGrid: https://www.cheatgrid.com/checklists/web-security-essentials/threat-modeling-a-new-feature?utm_source=github&utm_medium=listing&utm_campaign=threat-modeling-a-new-feature
