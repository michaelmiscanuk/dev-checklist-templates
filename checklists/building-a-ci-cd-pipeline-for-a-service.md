# Setting up CI/CD for a service

When: Starting a new service or repo, or taking over one that is still built and deployed by hand from somebody's laptop.

How to use: Use this when a service is still built or deployed by hand, or before its first pipeline goes live. Work through the moments in order; the middle three come back on every change and every release. Each title is the thing to do or the state to reach, and 'How to verify' says what to look at to know it is right.

Before you start: Does production wait for a person's approval? (any that apply) Waits for a person's approval / None of these. Checks marked "only if" apply only to that answer.
Before you start: Which of these describe your repository? (any that apply) Several services in one repo / None of these. Checks marked "only if" apply only to that answer.

## 1. Before you write the pipeline

You have the repo and a rough idea of how it ships today; nothing runs automatically yet.

Must:
- [ ] List every manual step today (source 1)
  Why: Watch the last person deploy, or read their shell history: the **unwritten steps** are the ones a pipeline has to replace.
  How to verify: A written list from checkout to live traffic, each step marked with the stage that will take it over; the count is your baseline to cut.
- [ ] Build from a clean checkout
  Why: Run the build in a fresh container holding only the repo, so **hidden laptop state** shows up now.
  How to verify: One documented command builds and tests the service in an empty container, with no files copied in by hand.

Don't forget these too:
- [ ] Pin tool and runtime versions
  Why: Commit the lockfile and name exact versions for the runtime, build tools and runner image, so every build resolves the same.
  How to verify: The lockfile is committed and matches the manifest, and the pipeline names fixed versions for runtime, base image and runner, not floating tags.
- [ ] Keep pipeline config in the repo (source 2)
  Why: Pipeline changes then go through the same review as code, and the history shows who changed each stage.
  How to verify: The pipeline definition lives in the service repo, and no stage exists only as settings in a web console.
- [ ] Choose what triggers each stage
  Why: A common split: pull requests build, test and scan; merges to the main branch also package and deploy.
  How to verify: The pipeline file shows which events, branches and paths start which jobs, and the team has agreed to that split.
- [ ] Write down the environment order (source 3)
  Why: Name each environment and the gate a build must pass to move to the next one.
  How to verify: The README lists the environments in order, such as staging then production, and the gate between each pair.
- [ ] Keep environment config out of builds (source 4)
  Why: Join settings and secrets to the build only at release, the twelve-factor **build, release, run** split, so one artifact serves every environment.
  How to verify: The build stage and the image contain no environment names, URLs or secrets; the deploy step supplies them.
- [ ] Pick a registry for artifacts
  Why: Use one store per artifact type that the deploy step can pull from in every environment.
  How to verify: Packages and images land in a named registry, and staging and production can both pull from it.
- [ ] List the laptop's secrets
  Why: Note every key, token and config file the deploy reads and where each lives, so each gets a **planned home**.
  How to verify: A list of every credential the deploy uses, with its current location and the secret store it will move to.
- [ ] Scan the full history for secrets
  Why: Run the secret scanner over every past commit and treat any key it finds as leaked: **rotate it**.
  How to verify: The history scan report is clean, or each finding has a rotated key and a ticket.

Stop if:
- If nobody knows how production is built today: ask the last person who deployed before changing anything

Move on when the build runs from a clean checkout with one command, and every manual step has a planned stage.

## 2. On every change: build, test, scan (every time)

A pull request is open or a merge just landed; the pipeline decides whether this change is safe to keep.

Must:
- [ ] Fail the run on every error
  Why: Remove ignore-error flags and catch-all fallbacks so a broken test or scan turns the **whole run red**.
  How to verify: Search the pipeline file for steps set to ignore errors; none remain, and a broken test makes the run fail.

Don't forget these too:
- [ ] Require green checks to merge (source 5)
  Why: Make the pipeline's jobs required status checks on the main branch, so every merge goes through them.
  How to verify: Branch protection on main lists the pipeline jobs as required, each job name unique, and a red pull request shows merging blocked.
- [ ] Run cheap checks first (source 6)
  Why: Put lint, format and type checks before slow test suites so small mistakes fail within a minute.
  How to verify: The run view shows fast checks starting first and slow suites after them.
- [ ] Cache dependencies by lockfile hash (source 7)
  Why: Key the cache on the lockfile so it refreshes exactly when dependencies change.
  How to verify: Run logs show a cache hit when the lockfile is unchanged and a miss right after it changes.
- [ ] Quarantine flaky tests with an owner (source 8)
  Why: Move a flaky test into a non-blocking reliability suite with a ticket and a named fixer, so it still runs; retry nothing else.
  How to verify: A short list of quarantined tests, each with an owner and a ticket, and no retry setting on the whole suite.
- [ ] Scan dependencies for known CVEs (source 9)
  Why: Run Software Composition Analysis (SCA), which matches dependencies against CVEs, the public record of known flaws, on every run and weekly.
  How to verify: The scan job runs on pull requests and on a schedule, and its report lists each finding with a severity.
- [ ] Set which severities block a merge (source 10)
  Why: Agree up front, for example critical and high block while the rest open a ticket, so the gate means something.
  How to verify: The scan step fails the run at the agreed severity and above, so a high threshold also blocks critical, and lower findings land in a queue someone reads.
- [ ] Scan commits for leaked secrets (source 11)
  Why: Turn on push protection, which scans each push for secrets, so a key committed by mistake is blocked before it enters history.
  How to verify: A test branch with a fake key in it turns the secret scan red.
- [ ] Build only the services that changed (source 12) (only if: Several services in one repo)
  Why: Give each service path filters so a change to one folder runs that service's jobs only.
  How to verify: A pull request touching one service's folder starts only that service's jobs in the run list.
- [ ] Include shared code in path filters (only if: Several services in one repo)
  Why: List shared libraries and root build files under every service that uses them, so a shared change rebuilds them all.
  How to verify: Change a file in a shared folder on a test branch: every dependent service's pipeline runs.

Stop if:
- If a test fails only in the pipeline: rerun it in the pipeline's image before skipping it
- If a critical finding has no fix yet: get a dated written exception from your security team

Move on when every required check on the change is green and merging is allowed because of it.

## 3. When you package the artifact (every time)

Tests passed on the main branch and the pipeline turns the code into the thing that will run.

Must:
- [ ] Build the artifact once
  Why: Promote the exact file or image that passed tests through every environment, **never rebuilding** it per environment.
  How to verify: Staging and production jobs pull an existing artifact by version, and neither contains a build step.
- [ ] Tag images with the commit SHA
  Why: The SHA is the commit's unique ID; deploy by the image digest, an **immutable content hash**, since tags can move.
  How to verify: The registry shows one tag per commit, and deploy manifests use a SHA tag or digest, never a floating tag like latest.

Don't forget these too:
- [ ] Publish only from the main branch
  Why: Pull request runs can build and test the image; only main-branch runs push to the release registry.
  How to verify: Every push in the registry's history comes from a main-branch run or a release tag.
- [ ] Stamp the commit into the build
  Why: Expose it on a version endpoint or an image label so anyone can see what is running.
  How to verify: The running service's version endpoint returns the same commit SHA as the latest deploy run.
- [ ] Keep release artifacts for rollback
  Why: Set retention so the releases you might go back to are never cleaned up.
  How to verify: A dry run of the registry's cleanup rule spares recent release artifacts, and the previous release still pulls today.
- [ ] Name artifacts per service (only if: Several services in one repo)
  Why: Put the service name in every artifact path so services built from one commit stay separate.
  How to verify: The registry holds a separate image or package path per service, each tagged with the same commit SHA.

Stop if:
- If a tag now points to a different image: stop promotions and tell the service owner

Move on when one artifact per commit sits in the registry, tagged with its commit and ready to promote.

## 4. When the pipeline deploys (every time)

An artifact is ready and the pipeline sends it to staging, then on to production.

Must:
- [ ] Allow one deploy per environment (source 13)
  Why: Use one **concurrency group** per environment, so one deploy runs, one waits, and a newer commit replaces the waiting one.
  How to verify: Push two commits a minute apart: the deploys run one after the other and production ends on the newer commit.
- [ ] Deploy to staging before production
  Why: Production pulls the same digest staging ran, so what you tested is what ships.
  How to verify: For one release, the image digest in staging's deploy log **matches production's**.
- [ ] Smoke test after every deploy
  Why: Call the health endpoint and one real user path from outside, and fail the job when either does not answer.
  How to verify: The deploy job's last step calls the live service and turns red when that call fails.

Don't forget these too:
- [ ] Wait for the rollout to finish (source 14)
  Why: Have the deploy step watch the rollout until new instances are healthy, with an explicit timeout; in kubectl rollout status, zero means wait forever.
  How to verify: A deploy that never becomes healthy ends red after the timeout instead of reporting success early.
- [ ] Make migrations work with old code (source 15)
  Why: Use parallel change, or expand and contract: add new columns first, switch the code next release, remove old ones after.
  How to verify: Each migration in a release runs in staging while the previous version still serves traffic.
- [ ] Run migrations as their own step (source 16)
  Why: Run them once before the new code starts, not at startup, so they get review and the app keeps **least-privilege database credentials**.
  How to verify: The pipeline shows one migration step ahead of the deploy step, and the app does not migrate on boot.
- [ ] Name who approves production (source 17) (only if: Waits for a person's approval)
  Why: Set required reviewers on the production environment itself; one approval is enough, so name a backup, and block self-review.
  How to verify: The production environment lists required reviewers, and a deploy waits until one of them approves.
- [ ] Show approvers what is shipping (only if: Waits for a person's approval)
  Why: Print the commits and the artifact digest since the last production deploy on the approval page.
  How to verify: A pending approval shows the commit list and digest, and the approver can open each commit.
- [ ] Gate production on staging checks (source 18)
  Why: Start the production job only after staging's smoke and automated tests pass for that same artifact.
  How to verify: The pipeline graph shows production depending on the staging test jobs, and a red staging run stops there.
- [ ] Ship unfinished work behind flags (source 19)
  Why: Every merge goes live, so merge half-built features behind a release toggle, a feature flag kept off until the work is done.
  How to verify: Code for an unreleased feature on the main branch is reachable only when its flag is on.
- [ ] Record every deploy
  Why: Log who deployed which version where and when, and send a marker to monitoring so graphs show releases.
  How to verify: The deploy history lists each release with commit, environment and time, and dashboards show a deployment marker at each one.
- [ ] Send failed runs to the team
  Why: Post failures to the channel the team watches and name an owner for the pipeline itself.
  How to verify: A failed deploy produces a message in the team channel within minutes, linking the run.

Stop if:
- If production runs a digest staging never ran: stop deploys and tell the release owner

Move on when production runs the artifact staging tested, and the smoke test passed there.

## 5. Before you call the pipeline done

Every stage exists; now prove the pipeline can ship and undo a release without anyone's laptop.

Must:
- [ ] Redeploy the previous version
  Why: Trigger the deploy job with the last release's artifact, so going back is **the same button** as going forward.
  How to verify: After the run, production serves the earlier digest and the smoke test passes on it.

Don't forget these too:
- [ ] Ship one change end to end
  Why: Make a small visible change and let it travel from pull request to production with nobody touching a server.
  How to verify: The run history shows one change passing every stage, and production's version endpoint shows its commit.
- [ ] Break a test on purpose
  Why: Push a branch with a failing test and a known vulnerable dependency to prove each gate stops it.
  How to verify: That pull request shows red checks and merging stays blocked.
- [ ] Retire the laptop deploy path
  Why: Once the pipeline has shipped and rolled back a release on its own, remove personal deploy access and old scripts, and tell the team.
  How to verify: Nobody can deploy to production from a laptop, and the old deploy script is gone from the repo.
- [ ] Deploy the live version first
  Why: Make the pipeline's first production deploy the version already running, so it proves the pipeline **without changing** what users get.
  How to verify: After that run, production's version endpoint shows the same commit as before and the smoke test passes.
- [ ] Rotate every laptop credential
  Why: Once the pipeline is the only way to deploy, give it fresh keys and revoke the old ones; agree timing with shared-key owners.
  How to verify: Each credential on the laptop's secrets list shows a rotation date after cutover, and the **old keys fail**.

Stop if:
- If the previous version will not redeploy through the pipeline: keep the manual path until it does

Move on when A change went from commit to production and back again through the pipeline alone.

Not covered: Locking down the build system itself (credentials, triggers, runners), writing the test suite, and pipelines for infrastructure code have their own checklists. Undoing a bad release lives in Rolling back a bad release.

Last reviewed: 2026-10-13

Sources:
1. DORA: Deployment automation capability: https://dora.dev/capabilities/deployment-automation/
2. Jenkins: Using a Jenkinsfile: https://www.jenkins.io/doc/book/pipeline/jenkinsfile/
3. Microsoft Learn: What is continuous delivery: https://learn.microsoft.com/en-us/devops/deliver/what-is-continuous-delivery
4. The Twelve-Factor App: Build, release, run: https://www.12factor.net/build-release-run
5. GitHub Docs: Managing a branch protection rule: https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/managing-a-branch-protection-rule
6. Martin Fowler: Deployment Pipeline: https://martinfowler.com/bliki/DeploymentPipeline.html?source=post_page---------------------------
7. GitHub actions/cache: caching strategies: https://github.com/actions/cache/blob/main/caching-strategies.md
8. Martin Fowler: Eradicating Non-Determinism in Tests: https://martinfowler.com/articles/nonDeterminism.html
9. OWASP: Dependency-Check project: https://owasp.org/www-project-dependency-check/
10. Anchore: Grype, filter vulnerability results: https://oss.anchore.com/docs/guides/vulnerability/filter-results/
11. GitHub Docs: About push protection: https://docs.github.com/en/code-security/concepts/secret-security/push-protection
12. GitHub Docs: Workflow syntax, paths filters: https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax
13. GitHub Docs: Control deployments using concurrency: https://docs.github.com/en/enterprise-server@3.17/actions/how-tos/deploy/configure-and-manage-deployments/control-deployments
14. Kubernetes docs: kubectl rollout status: https://kubernetes.io/docs/reference/kubectl/generated/kubectl_rollout/kubectl_rollout_status
15. Martin Fowler: Parallel Change: https://martinfowler.com/bliki/ParallelChange.html
16. Microsoft Learn: Applying EF Core migrations: https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/applying
17. GitHub Docs: Deployments and environments: https://docs.github.com/en/actions/reference/workflows-and-actions/deployments-and-environments
18. GitHub Docs: Using jobs in a workflow: https://docs.github.com/en/enterprise-server@3.20/actions/how-tos/write-workflows/choose-what-workflows-do/use-jobs
19. Martin Fowler: Feature Toggles: https://martinfowler.com/articles/feature-toggles.html

From CheatGrid: https://www.cheatgrid.com/checklists/devops-engineer/building-a-ci-cd-pipeline-for-a-service?utm_source=github&utm_medium=listing&utm_campaign=building-a-ci-cd-pipeline-for-a-service
