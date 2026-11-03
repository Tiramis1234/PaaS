# ADR 002: k3s with embedded etcd on three servers

- **Status:** Accepted
- **Recorded:** 2026-11-03

## Context

The cluster should survive the loss of any single node and stay relevant to CKA preparation, which covers etcd.

## Decision

Run k3s with embedded etcd on all three machines, each as a server. The first server initialises the cluster and the others join it.

## Alternatives considered

- **Two nodes with an external PostgreSQL datastore:** dropped once the laptop became the third node, because three etcd members give quorum without a separate database.

## Consequences

- The cluster has no single point of failure in the control plane and tolerates the loss of one node.
- Losing two nodes at once loses quorum, so the API stops until the nodes return or etcd is restored from a snapshot.
- etcd is sensitive to slow disks, so fsync latency has to be measured and monitored on every node.
- Each node spends memory on the control plane, which matters on 8 GB machines without swap.
