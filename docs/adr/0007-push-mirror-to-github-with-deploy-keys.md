# 7. Push-mirror GitLab repos to GitHub using per-repo SSH deploy keys

Date: 2026-09-29
Status: Accepted

## Context
All repos (sample-app, ledger-api, platform) lived only on the self-hosted GitLab
on the Beelink: no off-site copy, and no public visibility for the portfolio.
GitLab also runs CI, the container registry and Argo CD's source of truth, so it
must stay the primary.
Options for getting code to GitHub: (a) make GitHub the primary and retire
GitLab; (b) push a second remote by hand from the ThinkPad; (c) GitLab push
mirroring. For (c), authentication options: a GitHub personal access token (PAT)
over HTTPS, or an SSH deploy key per repo.

## Decision
Use GitLab push mirroring to github.com/esoung11/<repo>, over SSH, with a
GitLab-generated deploy key per repo (write access to that repo only). Mirror
protected branches only (`main`). GitHub's host key was verified against its
published fingerprints when the mirror was created.

## Consequences
- Every merge to `main` reaches GitHub automatically; no manual step to forget.
- Blast radius: a compromised lab can write to one GitHub repo per key, never the
  whole account (a PAT would grant account-wide scopes).
- Work-in-progress branches stay private in GitLab; GitHub shows reviewed `main`.
- GitHub is a read-only replica: changes made there are not synced back, so all
  work happens in GitLab.
- A replica is not a backup: a force-push or history rewrite on GitLab propagates
  to GitHub. Registry images, MRs/issues and cluster state are not copied
  (backups are handled in P2).
- Deploy keys don't expire; rotate them manually if the lab is ever compromised.
- Depends on outbound SSH (port 22) from the GitLab container to github.com.
