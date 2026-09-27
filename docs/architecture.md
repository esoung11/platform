# Architecture — homelab platform

_Last updated: 2026-09-27 (Stage 04: k3s cluster live; Argo CD + ledger-api in progress)_

## Overview
A self-hosted CI/CD + Kubernetes platform on a single Proxmox host (the Beelink).
Code is pushed to a self-hosted GitLab, built into a container image by a GitLab
Runner, and (target state) deployed to a k3s cluster by Argo CD via GitOps.

## Components
- **ThinkPad** — client only. Reaches the isolated lab network over an `sshuttle` tunnel; runs `git` and `kubectl`.
- **OPNsense** — firewall/router between the management network (vmbr0) and the isolated lab network (vmbr1, `10.10.20.0/24`); provides DHCP + outbound NAT.
- **GitLab CE + Container Registry** (`10.10.20.10`) — Git remote, CI/CD, image store.
- **GitLab Runner** (`10.10.20.11`) — executes pipeline jobs (builds images).
- **k3s cluster** — `k3s-server` (`10.10.20.20`, control plane + workloads) and `k3s-agent` (`10.10.20.21`, workloads).
- **Argo CD** (in progress) — runs in the cluster, watches the `platform` repo, and reconciles the cluster to match the manifests in Git.

## Diagram

```mermaid
flowchart TB
  subgraph TP["ThinkPad (client)"]
    git["git push"]
    kc["kubectl"]
    ss["sshuttle"]
  end
  subgraph BEE["Beelink — Proxmox host"]
    opn["OPNsense<br/>firewall + router"]
    subgraph LAB["Lab net 10.10.20.0/24 (isolated)"]
      gl["GitLab CE + Registry<br/>.10"]
      run["GitLab Runner<br/>.11"]
      subgraph K3S["k3s cluster"]
        srv["k3s-server .20<br/>control plane + workloads"]
        agt["k3s-agent .21<br/>workloads"]
        argo["Argo CD<br/>in cluster"]
      end
    end
  end
  git -->|push code| gl
  gl -->|trigger pipeline| run
  run -->|build + push image| gl
  argo -.->|watch manifests| gl
  argo -.->|apply| srv
  srv -->|pull image| gl
  kc -->|manage| srv
  ss -.->|tunnel in| LAB

