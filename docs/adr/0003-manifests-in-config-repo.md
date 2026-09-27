# 3. Store Kubernetes manifests in a config repo, not the app repo

Date: 2026-09-27
Status: Accepted

## Context
ledger-api's image is built in its own repo. We must decide where its Kubernetes
manifests (Deployment, Service) live and where Argo CD reads desired state from.
Options: (a) manifests alongside app code in ledger-api, (b) a separate
config/platform repo holding manifests for all apps.

## Decision
Keep manifests in this `platform` config repo under `apps/<name>/`, separate from
application source. Argo CD watches this repo; app repos only build and push images.

## Consequences
- Clean separation: app CI builds images; the platform repo declares what runs.
- One place to see everything deployed; scales as more apps are added.
- App CI needs no write access to deployment state (smaller blast radius).
- Trade-off: a new image tag from ledger-api must be propagated into this repo to
  deploy — an explicit step (manual now, automated later). Follow-up decision:
  image-tag flow.
