# Title: CNCP Out of First Release

* Caroline Harbitz
* Matthew Lewandowski
* Jake Partusch
* Stephen Yee

Date: 2026-10-29
Context: DEVPROD-46015 (CNCP sign-off), epic DEVPROD-44688

## Summary

The first release of the DevProd metrics dashboard shows the CI failures
metric for Evergreen only. CNCP gets a separate metric, the pipeline failure
rate, in a later release, after the data keeps every attempt of a workflow run.

## Background

Our goal is commit stability: a commit goes from merge to production with no
failure. No single metric measures this goal. Each metric measures one part of
the life of a commit.

The CI failures metric asks: "How frequently does an approved change cause a
CI failure?" It stops before deployment.

Evergreen is mainly CI:

* Evergreen builds and tests.
* Evergreen does not deploy most services.
* GitHub Actions and Argo deploy `10gen/mms`.
* Drone deploys Evergreen.
* Only the `evergreen-ui` project deploys from tasks.

CNCP is mainly CI/CD. One system workflow (`systems-<name>-deploy.yml` in
`10gen/mms`) does both:

    push to master
      -> lint, unit tests, integration tests, build
      -> deploy dev -> deploy qa -> deploy staging -> deploy prod
         (some systems also deploy to GovCloud)

    A failed job at any step stops the chain.

On CNCP, most failures occur in the CD part. Data from 2026-09-28 to
2026-10-25, push-to-master runs:

| Item | Count |
|---|---|
| System workflow runs | 1,960, in 91 workflows |
| Tested commits (runs with test jobs, 57 workflows) | 1,456 |
| Tested commits with a failed test job, last attempt | 7 (0.5%) |
| Deploy-stage failures (dev, qa, staging, prod, prod-gov, build) | 108 |
| Deploy jobs that failed on two or more commits in sequence | 10 |
| Runs with a re-run | 183 (9%) |
| Re-runs started by `mms-deploy-trigger[bot]` / by people | 136 / 47 |

Last-attempt run conclusions in the same window:

| Conclusion | Runs |
|---|---|
| `success` | 1,216 |
| `cancelled` | 465 |
| `failure` | 154 |
| not finished | 125 |
| `startup_failure` | 0 |

The CI failures metric measures only the CI part of CNCP. That is the part that
seldom fails.

We sent the CNE team four proposals for counting CNCP in the CI failures metric
(`docs/ci-failures-cncp-methodology-one-pager.md` in the dashboard repo). Jake
Partusch (CNE/CNCP) rejected two of them.

On proposal 2, "only the last attempt counts":

> This makes sense for test-specific metric, but for change failure rate, all
> counts should be counted — even if they are re-run. Something that prevented
> the pipeline from continuing is important to capture in the metric.

On proposal 3, "a commit gets a failure only if it caused it":

> Again, for change failure rate, these would all count! This is not really
> applicable to tests in CNCP.

These two rejections come from the CI versus CI/CD difference. They are not
two separate problems.

The feedback uses the term "change failure rate". DORA defines change fail
rate as "the ratio of deployments that require immediate intervention following
a deployment" ([dora.dev](https://dora.dev/guides/dora-metrics-four-keys/)).
The feedback asks for a different item: the share of runs that did not get
through the pipeline. This memo calls that item the **pipeline failure rate**.

Constraints:

* The data warehouse (`iceberg.raw_github`) usually keeps only the last
  attempt of a run. Of 183 runs with a re-run, only 11 have jobs from an
  earlier attempt. The runs table also keeps only the last attempt.
* To count every attempt, the loader must read jobs with `filter=all`. We do
  not know which team owns the loader.
* The dashboard ships two CNCP rates today, after merge only, last attempt
  only: failed test jobs / test jobs (0.2%), and failed deploy jobs / deploy
  jobs.
* No other dashboard metric measures CNCP pipeline failures. "PR creation to
  production" measures time, for the MMS monolith only.

## Problem

The CI failures metric and the CNE team measure different parts of commit
stability. We must decide if the first release includes CNCP in the CI
failures metric, and what happens to the CNCP rates that ship today.

## Decision evaluation criteria

1. **Mandatory.** The CNE team accepts the method.
2. **Mandatory.** The number is correct with the data that we have at release.
3. **Nice to have.** A reader can compare the number across CI systems.
4. **Nice to have.** Low cost before the first release.

## Solutions

### Option A: CNCP out of the first release, pipeline failure rate later

* The CI failures metric shows Evergreen only.
* Remove the CNCP "Test jobs" and "Deploy jobs" series from the UI.
* Keep the CNCP collector and the `gha_job_failure_rate` processor running.
  History continues to build up for the later metric.
* Later release: a pipeline failure rate for CNCP.
  * Unit: one **pipeline run**, which is one system workflow run on push to
    `master`. This includes the 34 workflows that deploy but run no test jobs.
  * A pipeline run fails if a failed job in any attempt stopped it.
  * A cancelled run is not in the numerator or the denominator.
  * The CNE team must confirm this unit.
* Prerequisites for the later metric:
  * Find the owner of the loader (questionnaire question 6).
  * Change the loader to keep every attempt (`filter=all`).

### Option B: CNCP in the CI failures metric, proposals as written

* Ship proposals 1 to 4 as in the one-pager.
* The proposed number is 7 of 1,456 tested commits (0.5%).
* The CNE team rejects proposals 2 and 3.
* The number does not include deploy failures, which are most CNCP failures.

### Option C: CNCP pipeline rate now, from run conclusions

* Use the last-attempt run conclusion: 154 runs ended `failure`.
* The data is available now.
* A failure that a re-run cleared is not visible. The CNE team says that this
  failure must count. 172 of the 183 runs with a re-run have no jobs from an
  earlier attempt.

## Evaluation

| Option | 1. CNE accepts | 2. Correct with current data | 3. Comparable across CI systems | 4. Low cost now |
|-|-|-|-|-|
| **A: out now, pipeline rate later** | Yes, if the CNE team confirms the unit | Yes, nothing ships until the data is complete | No | Yes, remove UI series |
| **B: proposals as written** | No, proposals 2 and 3 rejected | Yes, for the CI part | No, see the one-pager | Yes, small processor change |
| **C: run conclusions now** | No, re-run failures are not counted | No, earlier attempts are missing | No | Medium, new processor |

Option B and option C each fail a mandatory criterion. Option A meets both
mandatory criteria. No option meets criterion 3.

## Conclusion

Option A. The first release shows the CI failures metric for Evergreen only.
CNCP gets a pipeline failure rate in a later release.

Reasons:

* The CNE team owns CNCP, and it rejects the CI-only method.
* The CI-only method misses most CNCP failures, because they occur at deploy
  stages.
* The data cannot count every attempt today, so no correct pipeline number is
  possible before the loader changes.

Dissent:

* Some readers will want to compare the CI part of CNCP with Evergreen.
* Response: the dashboard already shows this as the CNCP test-job rate (0.2%).
  The CNCP one-pager says that this rate is not comparable with Evergreen. The
  two systems test different code with different jobs.

Risks:

* **The loader change does not occur.** The owner is unknown. Then the
  pipeline failure rate stays blocked. Action: ask the CNE team (questionnaire
  question 6), then the data platform team.
* **The CNE team does not accept the pipeline run as the unit.** Action: ask
  before we build the metric.

Questions for the CNE team about the pipeline failure rate:

1. Does a deploy failure that the environment caused count? The
   `voyage-entitlements` QA deploy failed on 8 of 19 commits. 7 of these
   failures came directly after a failure.
2. How do we count a run that waits for a prod approval? 125 runs had no
   conclusion when they were last loaded.
3. Does a re-run by `mms-deploy-trigger[bot]` mean that an earlier attempt
   failed? The bot started 136 of 183 re-runs.
4. Is a run with a higher `run_number` always a later commit? Some workflows
   use `cancel-in-progress: false`.

Follow-up work (no tickets yet):

* Remove the CNCP series from the CI failures pages
  (`web/src/routes/ci-failures/ChainsPage.tsx`) and update `docs/metrics.md`
  and the Methodology page.
* Add "Commit stability", "Pipeline failure rate" and "Pipeline run" to
  `CONTEXT.md`. Remove the CNCP lines from "Caused failure" and "Flaky
  failure".
* Close or change DEVPROD-46015, because proposals 1 to 4 are no longer for
  sign-off.
