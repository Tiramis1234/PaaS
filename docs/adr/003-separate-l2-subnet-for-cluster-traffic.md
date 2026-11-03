# ADR 003: Separate L2 subnet for cluster traffic

- **Status:** Accepted
- **Recorded:** 2026-11-03

## Context

All nodes share one unmanaged switch, so VLANs are not available. There is no access to the router, which means DHCP gives no guarantee of stable addresses and no reservations can be made. Cluster members need addresses that do not change.

## Decision

Add a second, statically addressed subnet, `10.10.10.0/24`, on the same interfaces and use it for cluster traffic. DHCP stays in place for internet access only.

Nodes use `10.10.10.50`, `10.10.10.51` and `10.10.10.52`.

## Alternatives considered

- **Static addresses in the router's network (`192.168.1.0/24`):** rejected. The DHCP pool is unknown and cannot be reserved, so another device could receive the same address. A conflict on an address used by etcd could drop a node from the cluster, and the failure would be intermittent and hard to diagnose.

## Consequences

- Cluster addressing does not depend on the router or its settings.
- Each node has two addresses on one interface.
- The machine running Ansible needs its own address in this subnet and has to be connected to the same switch.
- The API VIP also needs a reserved address outside `.50–.52`.
- This is not network isolation: without VLANs, all traffic still shares the same switch.
