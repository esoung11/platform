# 5. Use Tailscale for remote access (over self-hosted WireGuard)

Date: 2026-09-29
Status: Accepted

## Context
The lab network (10.10.20.0/24) is isolated and was previously reachable only via an
sshuttle tunnel from a device on the home LAN — so the homelab was unreachable from
any other network. We want secure remote access from anywhere. The home connection
is double-NAT'd (behind the ISP router and the home router), so no inbound port can be
reliably forwarded. Options: self-hosted WireGuard, or Tailscale (a managed
WireGuard-based mesh with NAT traversal).

## Decision
Use Tailscale. The Beelink runs as a subnet router advertising 10.10.20.0/24; the
ThinkPad joins the tailnet with `--accept-routes`. This replaces sshuttle.

## Consequences
- Works from any network with no port-forwarding — Tailscale handles NAT traversal,
  which self-hosted WireGuard could not do behind double NAT.
- No management ports are exposed on the home IP (meets the "lock the front door" goal).
- Depends on Tailscale's coordination service (a third party); Headscale is the
  self-hosted alternative if full control is later required.
- Tailscale is now the front door to everything — should be hardened with ACLs and
  reviewed in the threat model.
- Still WireGuard under the hood, so the protocol knowledge transfers.
