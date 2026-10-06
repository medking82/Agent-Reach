# Agent-Reach fork synchronization

The fork tracks `Panniantong/Agent-Reach` `main`. The workflow only targets
`medking82/Agent-Reach` `main`, and verifies the parent and both default branches
before preparing or publishing a candidate. It runs at 00:41, 06:41, 12:41 and
18:41 UTC (`41 */6 * * *`). GitHub may delay scheduled runs.

## Prepared versus enabled

Adding this workflow on a proposal branch does **not** enable scheduled syncing.
It must be reviewed and merged into `main` after maintenance approval.
Pull requests touching this workflow run preparation and validation only.
Manual dispatch on any branch except `main` skips all jobs.

At preparation time (2026-10-06), Actions is enabled, the default token permission
is `read`, and `can_approve_pull_request_reviews` is `false`: GitHub Actions may
not create or approve PRs. No setting, secret, or token has been changed.
The proposed `publish` job needs `contents: write` and `pull-requests: write`,
but is skipped until repository variable `AGENT_REACH_ALLOW_SYNC_PRS` is exactly
`true`. The read-only repository default can remain unchanged.

Activation requires explicit approval for each of the following:

1. Review and merge the setup PR into `main`. This enables scheduled candidate
   preparation and read-only validation, subject to the workflow being enabled
   in the fork's Actions tab. If GitHub shows the initial fork workflow banner,
   enable the reviewed workflow after approval.
2. Approve the proposed publisher's two job-scoped token write permissions.
   In this repository's **Settings → Actions → General → Workflow permissions**,
   enable **Allow GitHub Actions to create and approve pull requests**. This
   repository-wide checkbox also allows approvals; this workflow never approves
   or merges a PR. Do not enable the broader default read-and-write setting.
3. In **Settings → Secrets and variables → Actions → Variables**, create
   `AGENT_REACH_ALLOW_SYNC_PRS` with value `true` after that permission decision.
4. Run **Prepare upstream sync → Run workflow**, selecting `main` and keeping
   `validate_current` checked. Verify read-only checks and, when upstream has
   updates, the resulting draft PR. Every sync PR still requires human review
   and merge confirmation. Use **Create a merge commit**, not squash/rebase,
   so the next run recognizes preserved upstream and sync-branch ancestry.

Until steps 2 and 3 are approved, scheduled runs can validate candidates and
save bundles, but cannot publish a branch or create automatic review PRs.
No PAT, external GitHub App, third-party credential, deployment, release,
Agent-Reach runtime installation, browser, or cookie setup is involved.

## Safety and validation

- The candidate starts at fork `main`, then merges upstream `main`. Both commits
  must remain ancestors. There is no force push, reset, rebase or direct write
  to `main`, and merge conflicts abort the run and save the conflicting paths.
- An open sync PR pauses new candidates. An existing sync branch with unmerged
  work blocks the run, preserving manual edits and resolutions. If a PR was
  closed without merging, review/retire its branch explicitly; it is not replaced.
- A Git bundle and its SHA-256 identify the candidate. Separate read-only jobs
  run the existing unit suite on Linux Python 3.10-3.13 and Windows Python 3.12,
  plus check wheel packaging. Development dependencies exist only in disposable
  CI runners. A fresh publisher imports verified Git objects without checking
  out or executing candidate code, and rechecks both remote heads before writing.
- Upstream changes to `.github/workflows` stay as tested bundles for manual
  review. `GITHUB_TOKEN` cannot publish workflow-file updates; no extra credential
  is requested or created. This deliberate stop is not an automatic full sync.
- Only GitHub-owned actions pinned to commit SHAs are used. Existing upstream CI
  is left intact. The unrelated `scripts/sync-upstream.sh` compares x-reader
  channel code and is never invoked by this workflow.

Review a bundle in a separate checkout after checking its reported SHA-256:

```bash
git bundle verify /path/to/candidate.bundle
git fetch /path/to/candidate.bundle refs/heads/codex/upstream-sync:refs/remotes/review/sync
git log --oneline main..refs/remotes/review/sync
git diff main...refs/remotes/review/sync
```

Candidates and conflict reports are retained for seven days. A failed run needs
review in Actions; no separate messaging or notification integration is installed.
If branch publication succeeds but PR creation fails, the unmerged branch is
preserved. A maintainer can review it and create the draft PR manually after
confirming the PR-creation permission; subsequent schedules never overwrite it.

GitHub's `GITHUB_TOKEN` pushes do not trigger push CI. PRs created with this token
may require a maintainer to approve workflow runs. The candidate's validation in
the scheduled run is therefore performed before publication rather than relying
on downstream PR CI.

Reference: [GitHub token behavior](https://docs.github.com/en/actions/concepts/security/github_token),
[workflow token permissions](https://docs.github.com/en/actions/tutorials/authenticate-with-github_token),
and [scheduled workflow rules](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#schedule).
