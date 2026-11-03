# ADR 006: etcd snapshots to Cloudflare R2

- **Status:** Accepted
- **Recorded:** 2026-11-03

## Context

Losing quorum or all nodes must not mean losing the cluster state, so backups have to exist outside the homelab. The storage should be S3-compatible and fit in a free tier.

## Decision

Take k3s etcd snapshots twice a day and send them to Cloudflare R2 over its S3-compatible API, with 14-day retention. Local copies stay on the nodes with matching retention.

## Consequences

- The cluster can be restored after a complete loss of the hardware.
- The free tier covers 10 GB and there is no hard spending limit, so a lifecycle rule and an alert on bucket size are required.
- The restore procedure has to be tested and kept as a runbook.

## Open questions

- Snapshots contain cluster Secrets, which etcd does not encrypt by default. Decide whether to enable k3s secrets encryption, and how to keep the key available for a restore.
