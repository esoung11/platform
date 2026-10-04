# Project state — cloud / DevOps platform

_Last updated: 2026-10-04 (end of B2)._

## What this is & why
Building toward a generalist cloud/DevOps/platform engineer — the "glue" who knows enough of everything to coordinate and jump in. **One evolving fintech-style platform I build AND operate**; pragmatic fundamentals over shiny demos. Target: platform/DevOps roles, EU fintech; near-term internal transition; ~10 hrs/week. Breadth now, with a deep spine (Terraform + ops).

## How I work
See `../CLAUDE.md`. Short: I drive the reasoning, mentor verifies; one step at a time; I write files + run commands; predict→run→see, break-then-fix, curveballs, end-of-session recall; no nudges unless I ask; cap meta-work.

## Environment & access (NO SECRETS in this file)
- **Beelink** — Proxmox host, always-on. `ssh root@192.168.1.250` (home LAN) or via Tailscale. 12 cores / 31GB.
- **ThinkPad** (Ubuntu) — client. Repos at `~/homelab`.
- **Remote access** — Tailscale (Beelink = subnet router advertising `10.10.20.0/24`; ThinkPad on tailnet with `--accept-routes`). sshuttle retired. If `10.10.20.x` connects but hangs: don't run Ethernet + WiFi at once.
- **Lab net `10.10.20.0/24`** — OPNsense .1, host .5, GitLab .10, runner .11, k3s-server .20, k3s-agent .21.
- **GitLab CE** — http://10.10.20.10 (group `homelab`). Shell into a container: `ssh root@192.168.1.250` then `pct enter <id>` (101 GitLab, 102 runner).
- **kubectl** — from ThinkPad (`~/.kube/config`, server → 10.10.20.20). **Argo CD** — `kubectl port-forward -n argocd svc/argocd-server 8080:443` → https://localhost:8080 (user `admin`, password in my notes).
- **AWS** — scoped IAM user `terraform-cli` (NOT root), region `eu-west-1`, creds in `~/.aws`. Cost guardrails live: zero-spend budget + $5 runway + anomaly detection.

## Repos (GitLab → mirrored to github.com/esoung11)
- `platform/`   GitOps config: `apps/ledger-api/` (Deployment 3 replicas, Service), `argocd/` (Application, auto-sync), `docs/adr/` 0001–0008, `architecture.md`. (This file lives here.)
- `ledger-api/` Flask `/health` + `/metrics`; CI lint(ruff) → test(pytest) → dockerize.
- `aws-infra/`  Terraform for AWS. **CURRENT WORK.** Has `main.tf` (S3 state bucket), remote state in S3.
- `sample-app/` original bash pipeline (frozen).

## Done
- **01** network segmentation (OPNsense).
- **02** GitLab CE + runner, pipelines, branch protection.
- **03** containerize: image build/push to GitLab registry, SHA + main-`latest` tags (DinD).
- **04** k3s (2-node) + Argo CD GitOps; `ledger-api` deployed (namespace `ledger`).
- **12** Tailscale remote access.
- **P0** off-site: public GitHub mirrors, git history scanned clean.
- **B1** AWS cost guardrails.
- **B2** Terraform foundations: Terraform + AWS CLI installed; `terraform-cli` IAM user; `aws-infra/main.tf` → S3 state bucket (versioned, encrypted, public-access-blocked); **remote state in S3** (`encrypt` + `use_lockfile`).

## Key decisions (ADRs in `platform/docs/adr/`)
0001 record ADRs · 0002 k3s · 0003 manifests in config repo · 0004 Argo CD · 0005 Tailscale · 0006 pin images by SHA · 0007 push-mirror deploy keys · 0008 scan + rewrite history before publishing.
**TODO ADRs (B2 — write in `aws-infra`, my own words):** region `eu-west-1`; remote state on S3; IAM user not root.

## Known weak points (documented; fix over time)
Privileged DinD runner · plain-HTTP GitLab registry · single k3s control-plane · manual image-tag bumps · AWS IAM user has `AdministratorAccess` (not least-privilege).

## NEXT — B3
1. Add the six evidence docs to aws-infra; write a 3 TODO ADRs (region, remote state, IAM-not-root) in my own words.
2. Build **EC2 VM + RDS** in Terraform.
3. Then **healthchecks + routing + logging**; then **break-&-fix** ops drills.

## Learning log
- **2026-10-02** — Terraform state = Terraform's "save file" (maps config → real resources); moved local → S3 so it's shared, encrypted, locked. Set cost guardrails *before* building.

## Starting a new chat
Open `~/homelab` as the folder. First message: _"Read CLAUDE.md and platform/STATE.md. We're on <stage>."_ Update this file at the end of each stage.
