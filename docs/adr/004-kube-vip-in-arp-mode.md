# ADR 004: kube-vip in ARP mode

- **Status:** Accepted
- **Recorded:** 2026-11-03

## Context

The Kubernetes API needs one stable address that keeps working when a node fails. The network has an unmanaged switch and no access to the router, so BGP is not an option.

## Decision

Use kube-vip in ARP mode to provide a virtual IP for the API in the cluster subnet. Kubeconfig points at the VIP rather than at a specific node.

## Alternatives considered

- **kube-vip or another load balancer with BGP:** not possible without access to the router.

## Consequences

- The API stays reachable under one address after a node fails.
- At any moment, one node holds the VIP, so all API traffic enters through that node.
- Failover depends on the VIP moving to another node and the network learning its new location through ARP.
