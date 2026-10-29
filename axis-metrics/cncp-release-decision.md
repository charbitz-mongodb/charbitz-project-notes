# CNCP Excluded From First Release

List of participants: 
* Caroline Harbitz
* Matthew Lewandowski
* Stephen Yee

Date: 2026-10-29
Context: [DEVPROD-46015](https://jira.mongodb.org/browse/DEVPROD-46015) (CNCP sign-off)

## Summary

The first release of the DevProd metrics dashboard shows the CI failures metric for Evergreen only. CNCP gets a separate metric, the pipeline failure
rate, in a later release.

## Background

We want to know how often a commit fails between its merge to `master` and its
deployment to production. The dashboard's metric for this is **CI failures
after PR approval**: the share of CI runs on approved changes that end in a
failure that the change caused, at merge and after merge. On Evergreen, it
counts tasks. On CNCP, the UI shows two job rates: one for test jobs and one
for deploy jobs (see Constraints). 

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

We first built the CNCP method from the Evergreen method
([`docs/ci-failures-cncp-methodology-one-pager.md`](https://github.com/10gen/devprod-metrics-dashboard/blob/main/docs/ci-failures-cncp-methodology-one-pager.md) in the dashboard repo): a
tested commit gets at most one failure (proposal 1), only the last attempt
counts (proposal 2), and a failure counts only if the commit caused it
(proposal 3). With these rules, the metric tells whether a change broke CI.

The CNE team reviewed the method, and their feedback on proposals 2 and 3 is
that these rules leave out CNCP failures that matter. For CNCP, they count
every failure that stopped the pipeline, including a failure that a
re-run later cleared and a failure that the commit did not cause, because each
of these failures stopped or delayed a deployment.

With the CNE rules, the metric tells whether a pipeline run got through to
production. That question includes the deploy stages, where most CNCP failures
occur: in the same four weeks there were 108 deploy-stage failures and 7 test
failures. Evergreen does not deploy most services, so the CI failures metric
cannot answer the same question for Evergreen, and one metric cannot measure
the same thing on both systems.

The CNE team calls this measure "change failure rate". It is close to DORA's
change fail rate, which DORA defines as "the ratio of deployments that require
immediate intervention following a deployment"
([dora.dev](https://dora.dev/guides/dora-metrics-four-keys/)), but DORA counts
deployments and we count pipeline runs. To prevent confusion with DORA, this
memo calls it the **pipeline failure rate**.

Constraints:

* The data warehouse (`iceberg.raw_github`) usually keeps only the last
  attempt of a run. Of 183 runs with a re-run, only 11 have jobs from an
  earlier attempt. The runs table also keeps only the last attempt.
* To count every attempt, the loader must read jobs with `filter=all`. The
  GitHub API documents this parameter, so the change is small. We do not yet
  know which team owns the loader.
* The dashboard ships two CNCP rates today, after merge only, last attempt
  only: failed test jobs / test jobs, and failed deploy jobs / deploy
  jobs.
* No other dashboard metric measures CNCP pipeline failures.

## Problem

The CI failures metric asks whether a change broke CI, while the CNE team asks
whether a pipeline run got through to production. We must decide whether the
first release includes CNCP in the CI failures metric, and what happens to the
CNCP rates that ship today.

## Decision evaluation criteria

Ranked in priority order:
1. The CNE team, as the owner of CNCP, signs off on the method.
   We work out the method with them, so that the number shows how CNCP is
   doing.
2. The data that exists at release can produce the number as its definition states it.
3. A reader can compare the number across CI systems.
4. Low cost before the first release.

## Solutions

### Option 1: CNCP excluded from the first release, pipeline failure rate is a fast-follow

This option shows the CI failures metric for Evergreen only, and we remove the CNCP "Test jobs"
and "Deploy jobs" series from the UI. On the backend, the CNCP collector and the
`gha_job_failure_rate` processor continue to run, because this costs little
and keeps the last-attempt test and deploy counts.

In a later release, CNCP gets a pipeline failure rate. Its unit is the
**pipeline run**, one system workflow run on push to `master`, which also
covers the 34 workflows that deploy but run no test jobs. A pipeline run fails
if a failed job in any attempt stopped it, and a cancelled run is left out of
both the numerator and the denominator. The CNE team must confirm this unit.

Before we can build the later metric, the loader must keep every attempt
(`filter=all`). This is a small change once we find the owner of the loader
(questionnaire question 6).

### Option 2: CNCP included in the CI failures metric, proposals as written

This option ships proposals 1 to 4 as the one-pager gives them, which gives 7 of 1,456
tested commits (0.5%). The CNE feedback is that proposals 2 and 3 leave out
failures that matter for CNCP, and the number also leaves out deploy failures,
which are most of the CNCP failures.

### Option 3: Ship CNCP pipeline rate now, from run conclusions

This option is based on the last-attempt run conclusion, which is available now: 154 runs ended
`failure`. A failure that a re-run cleared is not visible in this data, though,
and the CNE feedback is that this failure counts. 172 of the 183 runs with a
re-run have no jobs from an earlier attempt.

## Evaluation

| Option | CNE sign-off | Correct with current data | Comparable across CI systems | Low cost now |
|-|-|-|-|-|
| **1: out now, pipeline rate later** | Possible, the unit is still to be worked out with the CNE team | Yes, nothing ships until the data is complete | No | Yes, remove UI series |
| **2: proposals as written** | Not as written, the feedback asks for re-run, not-caused and deploy failures | Yes, for the CI part | No, see the one-pager | Yes, small processor change |
| **3: run conclusions now** | Not as written, the feedback asks for re-run failures | No, earlier attempts are missing | No | Medium, new processor |

Options 2 and 3 each fail a mandatory criterion, while option 1 meets both. No
option meets the comparability criterion.

## Conclusion

We choose option 1: the first release shows the CI failures metric for
Evergreen only, and CNCP gets a pipeline failure rate in a later release.

The CNE team owns CNCP, and their feedback is that the CI failures metric does
not show how CNCP is doing. The metric leaves out failures that a re-run
cleared and failures that the commit did not cause, and it stops before the
deploy stages, where most CNCP failures occur. Because the
data cannot count every attempt today, no correct pipeline number is possible
until the loader changes.

Some readers will want to compare the CI part of CNCP with Evergreen. The
dashboard already shows that comparison as the CNCP test-job rate (0.2%), and
the CNCP one-pager says that this rate is not comparable with Evergreen,
because the two systems test different code with different jobs.

### Known risks

These are the known risks that come from choosing option 1:

1. The dashboard shows no CNCP failure numbers between the first release and
   the pipeline failure rate. Teams that read the CNCP rates today lose them
   for that time. The dashboard records page views
   (`web/src/observability/pageViews.ts`), so we can measure how many readers
   this affects.
2. The pipeline failure rate starts with little history. The data that the
   collector keeps today has job counts from the last attempt only, so it
   cannot give the pipeline failure rate for earlier weeks. A backfill with
   `filter=all` can recover earlier attempts only for the runs that GitHub
   still returns, and we have not checked how far back that goes.

### Open questions

We also have four questions for the CNE team about the pipeline failure rate:

1. Does a deploy failure that the environment caused count? The
`voyage-entitlements` QA deploy, for example, failed on 8 of 19 commits, and 7
of those failures came directly after another failure. 
2. How do we count a run that waits for a prod approval, given that 125 runs had no conclusion
when they were last loaded? 
3. Does a re-run by `mms-deploy-trigger[bot]`,
which started 136 of the 183 re-runs, mean that an earlier attempt failed?
4. Is a run with a higher `run_number` always a later commit, given that
some workflows use `cancel-in-progress: false`?
