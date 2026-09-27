# 4. Use Argo CD for GitOps continuous deployment

Date: 2026-09-27
Status: Accepted

## Context
We need a GitOps controller to reconcile the k3s cluster against manifests in the
platform repo, so deployments happen by merging to Git rather than running kubectl.
Main options: Argo CD and Flux.

## Decision
Use Argo CD — a clear web UI showing per-app sync/health, a gentle learning curve,
and heavy industry use (good interview signal). Flux is a valid CNCF alternative
(lighter, CLI-oriented) but Argo's visibility suits learning and demos better.

## Consequences
- Visual dashboard makes drift, sync and health obvious.
- Automated sync with selfHeal + prune: Git is the single source of truth.
- Extra components run in-cluster (argocd namespace) — more resource use than Flux.
- Argo's own config (the Application) is itself declarative in Git (`argocd/`).
