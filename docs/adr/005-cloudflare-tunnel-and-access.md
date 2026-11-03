# ADR 005: Cloudflare Tunnel and Cloudflare Access

- **Status:** Accepted
- **Recorded:** 2026-11-03

## Context

The homelab has no public IP address and ports cannot be forwarded, because there is no access to the router. Applications still have to be reachable from the internet, and admin panels have to be protected.

## Decision

All inbound internet traffic goes through Cloudflare Tunnel to the cluster Gateway. Admin panels sit behind Cloudflare Access. The Kubernetes API is never exposed publicly.

## Consequences

- No inbound ports are needed on the local network.
- Public access depends on an external provider.
- Tunnel requests are limited to 100 MB, so the container registry and pushes from CI work only on the LAN.
