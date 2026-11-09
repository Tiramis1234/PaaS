# ADR 010: Tailscale as the admin access path

- **Status:** Proposed
- **Recorded:** 2026-11-09

## Context

Tailscale is already installed on the nodes. The admin machine reaches them through Tailscale MagicDNS (for example `node-01` resolves to `node-01.taila6a531.ts.net`, `100.118.252.42`), so SSH and Ansible currently go over Tailscale instead of the LAN.

This was never recorded as a decision. It affects several existing ones:

- [ADR 003](003-separate-l2-subnet-for-cluster-traffic.md) assumes the machine running Ansible is connected to the same switch and has an address in `10.10.10.0/24`.
- [ADR 009](009-first-ansible-run-on-a-fresh-node.md) assumes a fresh node is reached at its DHCP address.
- The inventory is meant to use static cluster addresses, so it does not depend on DHCP.

Each node also gets more addresses: `tailscale0` (IPv4 in `100.64.0.0/10` and IPv6 in `fd7a:115c:a1e0::/48`). Without an entry in `/etc/hosts`, NSS `myhostname` resolves the node's own name to all of its addresses, including the Tailscale ones and public IPv6.

## Decision

Tailscale is used only as an admin access path: SSH and Ansible from outside the LAN.

Until the `10.10.10.0/24` subnet from ADR 003 exists, the inventory connects through Tailscale names. This is a temporary step. Once the static subnet works on all nodes, the inventory switches to `10.10.10.x`.

- Cluster traffic (etcd, kubelet, API, CNI) stays on `10.10.10.0/24`, as in ADR 003.
- k3s gets `node-ip` set explicitly. Node names in `/etc/hosts` resolve to cluster addresses.
- Tailscale is never part of the cluster's data path, and the Kubernetes API is not exposed through it.

## Alternatives considered

- **LAN-only admin access, no Tailscale:** the setup ADR 003 assumes. Rejected for convenience: the admin machine would have to be on the same switch for every Ansible run.
- **Tailscale for cluster traffic too:** rejected. It adds encryption overhead and an external control plane under etcd, and it mixes up the meaning of `node-ip`.
- **SSH through Cloudflare Tunnel and Access:** possible, but needs `cloudflared` on the admin machine and a working tunnel before the cluster exists. Tailscale works from the first boot.

## Consequences

- Nodes can be managed without being on the local network.
- Tailscale's coordination server is an external dependency for admin access. If it is unavailable, the LAN path (`10.10.10.x`) is still the fallback.
- The firewall from the hardening role must allow SSH on `tailscale0` and decide what else is allowed there.
- Cilium and Tailscale share the host's routing and packet filtering. Check that they do not conflict before week 3.
- ADR 003 and ADR 009 need a short note on how Tailscale changes their assumptions.

## Open questions

- After the switch to `10.10.10.x`, does Tailscale stay as a backup path for remote access, or is it removed?
- Is joining Tailscale part of the manual first run (ADR 009), or done by Ansible?
- Are Tailscale SSH and ACLs used, or plain OpenSSH over Tailscale?
