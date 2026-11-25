# Deployment trace — PROTOCOL §7

**Date:** 23 November 2026 · **Account:** author's own AWS account (us-east-1),
signed in as root via `aws login` temporary credentials · **Cost:** $0 (one
128 MB Lambda, seconds of runtime; all stacks deleted in the same session)

One pinned commit of the sample application, traced end to end:
GitHub PR → CI check → GitHub Actions deploy over OIDC → live invocation →
evidence saved → stacks deleted.

**PR checkout.** The CI check on this PR (run 45200153853) was triggered by
the push of `413b523ecd360aa3089986db64bf7198251ad330` on branch `submission`.
That SHA is what the run page records. A pull_request check does not test the
branch head itself: the default checkout is the synthetic merge of that head
with main at `03919e5`. That merge is trivial here: `413b523`'s parent is
`03919e5`, so there was nothing on main to merge in, and the merge tree is
`413b523`'s tree. The only difference between `413b523` and `03919e5` is the
added SUBMISSION.md; every other file, including the whole sample application,
is identical. So the check in run 45200153853 ran against the same application
tree as the deployed commit `03919e5`.

## The chain

1. **PR.** Submission PR
   [#1](https://github.com/upreti-labs/ai-review-benchmarks/pull/1) in this
   repository.
2. **CI check.** The starter's check workflow ("Check the sample
   application") passed on PR #1 —
   [run 45200153853](https://github.com/upreti-labs/ai-review-benchmarks/actions/runs/45200153853)
   (success, 6s). It also passed on the deployed commit itself: the push
   of `03919e5` to `main` ran the same check in
   [run 45195891744](https://github.com/upreti-labs/ai-review-benchmarks/actions/runs/45195891744)
   (success), so the revision that was deployed is a revision that was
   checked.
3. **One-time setup.** The account already had GitHub's OIDC identity
   provider, so none was created. The bootstrap stack
   `ai-review-bench-github` (template `aws/github-roles.yaml`) created the
   two scoped roles — a deployment role usable only by this repository's
   `main` branch (immutable subject
   `repo:upreti-labs@…/ai-review-benchmarks@…:ref:refs/heads/main`) and a
   Lambda runtime role that can only write the sample's log group — and
   their ARNs were set as repository variables. No AWS access key exists
   in GitHub.
4. **Deploy.** Workflow "AWS sample deployment", operation `deploy`,
   dispatched from `main`:
   [run 45204420111](https://github.com/upreti-labs/ai-review-benchmarks/actions/runs/45204420111)
   — success in 46s, head commit
   `03919e532b6d52e1e3c41b8b4a112c7e233c48c1`. Log line:
   `Deployed 03919e532b6d52e1e3c41b8b4a112c7e233c48c1 to ai-review-bench in us-east-1.`
   (2026-11-23T15:22:45Z).
5. **Verified invocation (live AWS response).** The same run invoked the
   deployed function and checked the response:

   ```json
   {
     "commit": "03919e532b6d52e1e3c41b8b4a112c7e233c48c1",
     "request_id": "00e6268a-8236-44d9-b4df-b4adfc94146d",
     "report": "Weekly orders",
     "delivered_this_run": 1,
     "receipt_at": 101,
     "summary": { "delivered": 1, "pending": 0 }
   }
   ```

   The request ID is the Lambda's own; the summary matches the starter's
   expected result (one delivered, zero pending).
6. **Local smoke result (labelled local, not a deployment).** The same
   workflow's pre-deploy artifact check ran the packaged code locally and
   returned the same commit, report and counts with request ID
   `local-smoke`. It is recorded here only to keep the two kinds of
   evidence distinct, as `docs/aws.md` requires.
7. **Deletion.** Delete operation dispatched from `main`:
   [run 45204701798](https://github.com/upreti-labs/ai-review-benchmarks/actions/runs/45204701798)
   — success. The bootstrap roles stack was then deleted from the CLI.
   Verified afterwards in the same session: no stacks in any live status
   in us-east-1, and `lambda list-functions` returns 0 functions. The
   pre-existing GitHub OIDC provider was left in place, as the setup
   guide instructs for shared providers.
