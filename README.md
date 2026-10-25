# Developer checklist templates

Pull request and issue templates for everyday engineering work: designing tables, setting up CI/CD, laying out a responsive page, threat modeling a feature and more. Copy one into your repo and the checklist shows up every time someone opens a PR or an issue.

Every checklist puts its **Must checks** first, ends each step with a **Stop if** list (the signs to stop and ask), and has a "How to verify" line and a source under every check in its full version.

## Pull request templates

The Must checks of the work a PR ships, plus the Stop if lines.

| Checklist | Use it when | PR template | Full checklist |
|---|---|---|---|
| **Building a feature end to end** | You pick up a feature that needs a new screen, new API endpoints and new tables, and you are about to start. | [template](.github/PULL_REQUEST_TEMPLATE/building-a-feature-end-to-end.md) | [full](checklists/building-a-feature-end-to-end.md) |
| **Designing tables for a new feature** | You are about to write the first migration for a new feature and need tables, keys and constraints that will still fit next year. | [template](.github/PULL_REQUEST_TEMPLATE/designing-tables-for-a-new-feature.md) | [full](checklists/designing-tables-for-a-new-feature.md) |
| **Setting up CI/CD for a service** | Starting a new service or repo, or taking over one that is still built and deployed by hand from somebody's laptop. | [template](.github/PULL_REQUEST_TEMPLATE/building-a-ci-cd-pipeline-for-a-service.md) | [full](checklists/building-a-ci-cd-pipeline-for-a-service.md) |
| **Laying out a responsive page** | Starting a new page or section that has to work from a small phone up to a wide desktop screen. | [template](.github/PULL_REQUEST_TEMPLATE/laying-out-a-responsive-page.md) | [full](checklists/laying-out-a-responsive-page.md) |
| **Designing classes for a new feature** | You are about to write the classes for a new feature and want them to hold up once other code depends on them. | [template](.github/PULL_REQUEST_TEMPLATE/designing-classes-for-a-new-feature.md) | [full](checklists/designing-classes-for-a-new-feature.md) |
| **Writing a prompt for an app feature** | You are adding a new LLM-powered feature, or rewriting the prompt behind one, and want it to hold up on real user input. | [template](.github/PULL_REQUEST_TEMPLATE/writing-a-prompt-for-an-app-feature.md) | [full](checklists/writing-a-prompt-for-an-app-feature.md) |

## Issue templates

Every check, for planning the work in an issue before anyone writes code.

| Checklist | Use it when | Issue template | Full checklist |
|---|---|---|---|
| **Threat modeling a new feature** | Before building a feature that touches logins, payments, personal data, file handling or a new outside integration. | [template](.github/ISSUE_TEMPLATE/threat-modeling-a-new-feature.md) | [full](checklists/threat-modeling-a-new-feature.md) |
| **Querying data for a business question** | Someone asks a question the data can answer and you are about to write the SQL that produces the number. | [template](.github/ISSUE_TEMPLATE/querying-data-for-a-business-question.md) | [full](checklists/querying-data-for-a-business-question.md) |
| **Running a one-on-one with a report** | Before a weekly or biweekly 1:1, or when your 1:1s have turned into status updates nobody gets anything from. | [template](.github/ISSUE_TEMPLATE/running-a-one-on-one-with-a-report.md) | [full](checklists/running-a-one-on-one-with-a-report.md) |

## How to use one

- **One PR template for the whole repo:** copy the file to `.github/pull_request_template.md` in your repo.
- **Several PR templates:** put them in `.github/PULL_REQUEST_TEMPLATE/`, then open a PR with `?template=<file name>` added to the compare URL, for example `?template=designing-tables-for-a-new-feature.md`.
- **Issue templates:** copy the file into `.github/ISSUE_TEMPLATE/`. It appears in the "New issue" chooser.
- Some checks say **"only if"**: they apply to one kind of change only (rows hidden instead of deleted, two writers on one row). Delete them when they don't fit yours.

## Where they come from

Each checklist is one of the free checklists on [CheatGrid](https://www.cheatgrid.com/checklists?utm_source=github&utm_medium=listing&utm_campaign=dev-checklist-templates), where every check links to the official documentation or study that backs it. The interactive versions there ask one or two questions first and hide the checks that don't apply, and you can tick them as you go.

## License

[CC BY 4.0](LICENSE): use, change and share them, in work projects too, with a credit to CheatGrid (a link to this repo or to the checklist is enough).
