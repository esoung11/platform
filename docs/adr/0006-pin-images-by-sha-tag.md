# 6. Pin container images by immutable SHA tag, not :latest

Date: 2026-09-29
Status: Accepted

## Context
ADR-0003 left a follow-up: how a new ledger-api image reaches this repo. The
Deployment referenced `ledger-api:latest`. During P0 we found the cluster still
running a two-day-old build (digest d79aacf) while `:latest` already pointed to a
newer one (aaeb96f). Git said "latest", the cluster ran something else, and
nothing recorded which. Because `:latest` never changes in Git, Argo CD cannot see
a new image, and any pod restart silently pulls whatever `:latest` means then.
Options: (a) keep `:latest` with `imagePullPolicy: Always`; (b) pin the short-SHA
tag CI already publishes, bumped by a commit to this repo; (c) pin by digest
(`@sha256:…`).

## Decision
Reference images by the 8-character short-SHA tag CI publishes
(`$CI_COMMIT_SHORT_SHA`), e.g. `ledger-api:c0935b12`. Deploying a new version is
a commit that changes the tag here, merged via MR. Manual for now.

## Consequences
- Git is an exact record of what runs; every pod traces back to a ledger-api commit.
- Rollback is a `git revert` of the tag change; Argo CD reconciles.
- Deploys are explicit and reviewable (MR + pipeline).
- Cost: a manual tag bump per release until automated (Argo CD Image Updater or a
  CI job that opens an MR). Future ADR.
- Registry tags can technically be overwritten; digest pinning or image signing
  (P1, cosign) would close that gap. Accepted for the lab.
