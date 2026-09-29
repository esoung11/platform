# 8. Scan and rewrite Git history before publishing to GitHub

Date: 2026-09-29
Status: Accepted

## Context
The repos were about to become public via the GitHub mirror (ADR-0007). Anything
pushed to a public repo should be treated as permanently exposed, even if deleted
later. Two risks: secrets in history (the legacy GitLab PAT, the registry pull
secret, Argo CD's access token), and personal data in commit metadata. An audit of
authors found a placeholder email, GitLab web-UI commits, and one personal email.
Options: (a) publish history as-is; (b) squash each repo into a fresh single
commit; (c) scan the full history, then rewrite only the author identities.

## Decision
Scan every ref of every repo with Gitleaks (plus a search for committed Kubernetes
Secrets, which Gitleaks can miss because values are base64-encoded) before
mirroring. All clean. Then rewrite all author/committer identities to the GitHub
noreply address with `git filter-repo --mailmap`, force-pushed to GitLab with
branch protection relaxed for the push only and restored straight after. The
global git identity now uses the noreply address for all new commits.

## Consequences
- No secrets or personal email in public history; GitHub also blocks future pushes
  that would expose a real email.
- Every commit is attributed to the GitHub profile (counts toward its activity).
- Real history and commit messages are kept, unlike a squash.
- All commit SHAs changed. Images tagged with old SHAs no longer match a commit;
  accepted because the deployment used `:latest` at the time (see ADR-0006), and
  the image was rebuilt and pinned to the new SHA.
- Old commits still exist in GitLab's internal merge-request refs; these are
  private and are never mirrored.
- One-off cost: local clones had to be reset to the rewritten history.
- A rewrite is only cheap before publishing; after going public it would break
  forks and links, so this check belongs before the first public push.
