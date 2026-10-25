## Setting up CI/CD for a service

When: Starting a new service or repo, or taking over one that is still built and deployed by hand from somebody's laptop.

Before you start: Does production wait for a person's approval? (any that apply) Waits for a person's approval / None of these. Checks marked "only if" apply only to that answer.
Before you start: Which of these describe your repository? (any that apply) Several services in one repo / None of these. Checks marked "only if" apply only to that answer.

### 1. Before you write the pipeline

You have the repo and a rough idea of how it ships today; nothing runs automatically yet.

Must:
- [ ] List every manual step today
- [ ] Build from a clean checkout

Stop if:
- If nobody knows how production is built today: ask the last person who deployed before changing anything

Move on when the build runs from a clean checkout with one command, and every manual step has a planned stage.

### 2. On every change: build, test, scan (every time)

A pull request is open or a merge just landed; the pipeline decides whether this change is safe to keep.

Must:
- [ ] Fail the run on every error

Stop if:
- If a test fails only in the pipeline: rerun it in the pipeline's image before skipping it
- If a critical finding has no fix yet: get a dated written exception from your security team

Move on when every required check on the change is green and merging is allowed because of it.

### 3. When you package the artifact (every time)

Tests passed on the main branch and the pipeline turns the code into the thing that will run.

Must:
- [ ] Build the artifact once
- [ ] Tag images with the commit SHA

Stop if:
- If a tag now points to a different image: stop promotions and tell the service owner

Move on when one artifact per commit sits in the registry, tagged with its commit and ready to promote.

### 4. When the pipeline deploys (every time)

An artifact is ready and the pipeline sends it to staging, then on to production.

Must:
- [ ] Allow one deploy per environment
- [ ] Deploy to staging before production
- [ ] Smoke test after every deploy

Stop if:
- If production runs a digest staging never ran: stop deploys and tell the release owner

Move on when production runs the artifact staging tested, and the smoke test passed there.

### 5. Before you call the pipeline done

Every stage exists; now prove the pipeline can ship and undo a release without anyone's laptop.

Must:
- [ ] Redeploy the previous version

Stop if:
- If the previous version will not redeploy through the pipeline: keep the manual path until it does

Move on when A change went from commit to production and back again through the pipeline alone.

---
Must checks only. Why each check matters, how to verify it and its source: [full checklist](https://github.com/michaelmiscanuk/dev-checklist-templates/blob/main/checklists/building-a-ci-cd-pipeline-for-a-service.md). Interactive version: [Setting up CI/CD for a service on CheatGrid](https://www.cheatgrid.com/checklists/devops-engineer/building-a-ci-cd-pipeline-for-a-service?utm_source=github&utm_medium=listing&utm_campaign=building-a-ci-cd-pipeline-for-a-service).
