# ADR 001: Bare metal instead of Proxmox

- **Status:** Accepted
- **Recorded:** 2026-11-03

## Context

The cluster runs on three machines with 8–12 GB of RAM each. A hypervisor and a set of VMs would use part of that memory before any workload starts.

## Decision

Install the operating system and k3s directly on the hardware, without Proxmox.

## Alternatives considered

- **Proxmox with VMs as cluster nodes:** rejected because there is not enough RAM for a hypervisor and VMs.

## Consequences

- All of the memory goes to the system, k3s and workloads.
- There are no VM snapshots to roll back to, so a broken node has to be rebuilt with Ansible.
- Practising a kubeadm install for the CKA has to happen separately, for example on VMs or in an exam simulator.
