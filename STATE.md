# Project state — cloud security platform

_Last updated: 2026-11-07 (pivot to cloud security; B3 step 1 done)._

## What this is & why
Building toward **cloud security engineer**, NZ-based (pivoted from platform/DevOps on 2026-11-07). **One evolving fintech-style platform I build, attack and defend.** The homelab and AWS work so far become the target environment; security is layered on top. ~10 hrs/week, alongside a security course and TryHackMe.

## How I work
See `../CLAUDE.md`. Short: I drive the reasoning, mentor verifies; one step at a time; I write files + run commands; predict→run→see, break-then-fix, curveballs, end-of-session recall; no nudges unless I ask; cap meta-work. Mechanical chores (placeholders, boilerplate) can be "write it for me".

## Environment & access (NO SECRETS in this file)
- **Beelink** — Proxmox host, always-on. `ssh root@192.168.1.250` (home LAN) or via Tailscale. 12 cores / 31GB.
- **ThinkPad** (Ubuntu) — client. Repos at `~/homelab`.
- **Remote access** — Tailscale (Beelink = subnet router advertising `10.10.20.0/24`; ThinkPad on tailnet with `--accept-routes`). If `10.10.20.x` connects but hangs: don't run Ethernet + WiFi at once.
- **Lab net `10.10.20.0/24`** — OPNsense .1, host .5, GitLab .10, runner .11, k3s-server .20, k3s-agent .21.
- **GitLab CE** — http://10.10.20.10 (group `homelab`). Shell into a container: `ssh root@192.168.1.250` then `pct enter <id>` (101 GitLab, 102 runner).
- **kubectl** — from ThinkPad (`~/.kube/config`, server → 10.10.20.20). **Argo CD** — `kubectl port-forward -n argocd svc/argocd-server 8080:443` → https://localhost:8080 (user `admin`, password in my notes).
- **AWS** — scoped IAM user `terraform-cli` (NOT root), creds in `~/.aws`. Region **moving `eu-west-1` → `ap-southeast-2` (Sydney)**. Cost guardrails live: zero-spend budget + $5 runway + anomaly detection.
- **AWS Free plan** — $119.76 credits (2026-10-19), **expires 2027-01-31**. On expiry/zero credits the account closes unless upgraded — the state bucket goes with it. Decide on upgrade before Jan.

## Repos (GitLab → mirrored to github.com/esoung11)
- `platform/`   GitOps config: `apps/ledger-api/`, `argocd/`, `docs/adr/` 0001–0008, `architecture.md`. (This file lives here.)
- `ledger-api/` Flask `/health` + `/metrics`; CI lint(ruff) → test(pytest) → dockerize.
- `aws-infra/`  Terraform for AWS. **CURRENT WORK.** `main.tf` (S3 state bucket, eu-west-1), remote state in S3; `docs/` six-piece evidence set, ADRs 0001–0004.
- `sample-app/` original bash pipeline (frozen). `repo-template/` starter with the six-piece evidence standard.

## Done
- **01** network segmentation (OPNsense).
- **02** GitLab CE + runner, pipelines, branch protection.
- **03** containerize: image build/push to GitLab registry, SHA + main-`latest` tags (DinD).
- **04** k3s (2-node) + Argo CD GitOps; `ledger-api` deployed (namespace `ledger`).
- **12** Tailscale remote access.
- **P0** off-site: public GitHub mirrors, git history scanned clean.
- **B1** AWS cost guardrails.
- **B2** Terraform foundations: `terraform-cli` IAM user; S3 state bucket (versioned, encrypted, public-access-blocked); remote state (`encrypt` + `use_lockfile`).
- **B3.1** six evidence docs in `aws-infra/docs`; ADRs 0002 region, 0003 remote state, 0004 IAM user not root.

## Key decisions
- `platform/docs/adr/`: 0001 record ADRs · 0002 k3s · 0003 manifests in config repo · 0004 Argo CD · 0005 Tailscale · 0006 pin images by SHA · 0007 push-mirror deploy keys · 0008 scan + rewrite history before publishing.
- `aws-infra/docs/adr/`: 0001 record ADRs · 0002 eu-west-1 (to be superseded) · 0003 remote state on S3 · 0004 IAM user not root.
- **TODO ADR:** `aws-infra` 0005 — use `ap-southeast-2` (Sydney): mature, full service set, where most NZ companies run; Auckland (`ap-southeast-6`) rejected for now — newer, fewer services. Supersedes 0002.
- B3 design (2026-10-19): full commercial layout — VPC, private RDS, ALB, SSM (no SSH), CloudWatch — paid from credits, **`terraform destroy` after every session**.

## Known weak points → findings to scan, report and fix
Privileged DinD runner · plain-HTTP GitLab registry · single k3s control-plane · manual image-tag bumps · AWS IAM user has `AdministratorAccess` (not least-privilege) · long-lived AWS access keys on disk.

## Roadmap — three layers
Order: **Layer 2 first** (credits expire 2027-01-31; the Beelink has no deadline). Layer 1 grows with the course modules. Layer 3 is threaded through both.

**Layer 2 · Secure AWS deploy, broken & fixed — NOW**
- L2.0 Region move to Sydney + ADR-0005; split Terraform so the state bucket (`bootstrap/`) is never in the stack that gets destroyed.
- L2.1 EC2 + private RDS + ALB + SSM in Terraform.
- L2.2 Security baseline: CloudTrail, GuardDuty, CloudWatch alarms; scan with Prowler/ScoutSuite → findings into `threat-model.md`.
- L2.3 Misconfigure → catch → fix drills (public bucket, open SG, over-privileged role).

**Layer 1 · Detection lab on the Beelink — light now, grows with the course**
- L1.0 Isolated attack/victim network in OPNsense + a vulnerable box (Juice Shop / Metasploitable) — for the Ethical Hacking module.
- L1.1 Wazuh SIEM + Suricata IDS on OPNsense; attack the box, watch alerts fire — for Network Defence + SecOps modules.

**Layer 3 · DevSecOps in GitLab CI — threaded through**
- tfsec/Checkov on `aws-infra` (during L2.1); Gitleaks + Trivy on `ledger-api`; a policy gate that fails on a critical finding.

**Parked:** Prometheus/Grafana (Wazuh + CloudWatch cover it); more k8s/Argo work.

## Study alongside (not build)
- **Course** (order): Ethical Hacking (**current**) → Network Defence → Web Apps → SecOps & Incident Awareness.
- **TryHackMe** Cyber Security 101 → SOC Level 1 → Defending AWS. **OverTheWire Bandit** for Linux reps.
- Course ↔ project: Ethical Hacking → attack L1 lab · Network Defence → OPNsense/Suricata · Web Apps → OWASP-test `ledger-api` + Juice Shop · SecOps → triage Wazuh/GuardDuty alerts.
- Rule: only attack systems I own or have written permission to test.

## Learning log
- **2026-10-02** — Terraform state = Terraform's "save file" (maps config → real resources); moved local → S3 so it's shared, encrypted, locked. Set cost guardrails *before* building.
- **2026-10-19** — Encryption protects state from a breach (unreadable if stolen); locking protects it from two concurrent applies corrupting it. Different threats.
- **2026-11-07** — Pivot to cloud security. ADRs are immutable: a changed decision (region) is a new ADR that supersedes the old one.

## Starting a new chat
Open `~/homelab` as the folder. First message: _"Read CLAUDE.md and platform/STATE.md. We're on <stage>."_ Update this file at the end of each stage.
