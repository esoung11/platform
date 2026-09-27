# 2. Use k3s for the Kubernetes distribution

Date: 2026-09-27
Status: Accepted

## Context
Stage 04 needs a real Kubernetes cluster on the Proxmox homelab (limited RAM,
single always-on host) to learn orchestration + GitOps and to run ledger-api.
Options: k3s, kubeadm (vanilla upstream), managed cloud Kubernetes.

## Decision
Use k3s across two Ubuntu 24.04 VMs — server/control-plane at 10.10.20.20,
agent/worker at 10.10.20.21. Managed cloud k8s is out of scope for the free
homelab stage; kubeadm rejected here as heavier and more manual.

## Consequences
- Fast path to a CNCF-conformant cluster; kubectl and manifests are identical to
  upstream k8s, so skills transfer.
- Low resource footprint fits the Beelink.
- Single server = no control-plane HA: if it dies, existing pods keep running but
  no new scheduling/self-healing until it returns (accepted for a lab; prod runs 3+).
- Opinionated k3s defaults (Traefik, ServiceLB, local-path, embedded datastore)
  differ from vanilla k8s — noted so they aren't mistaken for standard behaviour.