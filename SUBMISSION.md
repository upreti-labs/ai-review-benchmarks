# Submission — AI Review Benchmarks

**Author:** Bhavya Upreti · **For:** Proofline Software Engineering Intern,
Developer Tools application · **Date:** 23 November 2026

## What this is

A controlled benchmark of five AI code reviewers — Proofline, Greptile,
CodeRabbit, Qodo and Sourcery — run against the same nine cases on the
sample application from `prooflinedev/ai-review-benchmarks` (pinned at
commit `79d03777`): the template's eight changes plus one original
held-out case written for this benchmark. The scoring protocol was committed before any reviewer was
installed (pre-registration commit `df47bb1`); grading was per claim,
against that frozen rubric, on copies with provider names replaced by
random codes.

## The finding in one paragraph

On ordinary code defects, all five reviewers performed well (3 of 3 for
every provider, and 5 of 5 on the held-out case). Two of the deployment
changes, cases 04 and 05, have defects that exist only against live
deployment facts: no reviewer caught either from the code alone (0 of 10
observations). The third, case 06, is different: the worker and queue
mismatch is already written in the README, so all five reviewers caught it
from the code alone. Across the 15 code-only observations of the three
changes, the appropriate-uncertainty grade was earned zero times, including
one observation, Proofline on case 05, where no review arrived at all. That
is a count of grades, not of everything the reviewers wrote: grading is per
filed claim, and risk prose that files no finding earns no grade. Given the deployment snapshot in the PR body, they
caught **13 of 14** completed observations of the same defects: the snapshot
confirmed which site was affected in case 06, and it unlocked cases 04 and
05, 8 of 9 completed observations once it was included. The evidence is
the bottleneck, and the missing "I don't have what I need to judge this"
is the gap.

## How to review this PR

1. **`REPORT.md`** — the full report: headline table with denominators,
   results by condition, one result traced end to end, methods, plan
   reality, limitations.
2. **`PROTOCOL.md`** — the pre-registered protocol; §4 is the frozen
   scoring rubric. It has never been amended.
3. **`reproduce/test_cases.py`** — the executable ground truth: 12 checks
   demonstrating the code defects (01–03), the held-out case (09) and both
   correct changes (07, 08). The deployment cases 04–06 are verified by
   line-by-line trace against their snapshot data, the check PROTOCOL §4
   prescribes for snapshot cases.
4. **`bench/results/`** — generated score and coverage tables;
   **`bench/grades.csv`** — every graded claim with its reason;
   **`bench/raw/`** — every review received, verbatim.

## Honest notes

- One observation is still outstanding: Proofline's review of case 05
  with the snapshot had not arrived when the report was written. It is
  excluded from Proofline's denominators in the report and will be graded
  as a late observation if it lands. Proofline also returned no review on
  two other PRs; both are recorded in the report, not hidden.
- The deployment trace described in `PROTOCOL.md` §7 (PR → CI → deploy →
  verify → delete) was completed on 23 Nov 2026 and is committed on this
  branch as `trace/aws-trace.md`, with the workflow links and the live
  invocation receipt.
- Total spend: **$0**. No payment card was entered anywhere; four of the
  five providers were run on free trials. The report's plan-reality table
  has the provider-specific reasons.
- Grading was blind by provider code, but the blinding was partial: the
  review copies still carry the PR URLs, which name the run repos, and
  product boilerplate remains. The report's limitations state this in full.
