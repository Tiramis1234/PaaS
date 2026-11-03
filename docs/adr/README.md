# Architecture Decision Records

Each record explains one significant decision in this project: its context, the choice made, the alternatives considered and the consequences.

| ADR | Decision | Main reason |
|---|---|---|
| [001](001-bare-metal-instead-of-proxmox.md) | Bare metal instead of Proxmox | 8–12 GB of RAM per node, no hypervisor overhead |
| [002](002-k3s-with-embedded-etcd.md) | k3s with embedded etcd on three servers | Quorum with three members, no single point of failure, CKA relevance |
| [003](003-separate-l2-subnet-for-cluster-traffic.md) | Separate L2 subnet for cluster traffic | No router access, DHCP does not guarantee stable addresses |
| [004](004-kube-vip-in-arp-mode.md) | kube-vip in ARP mode | Unmanaged switch, no BGP |
| [005](005-cloudflare-tunnel-and-access.md) | Cloudflare Tunnel and Cloudflare Access | No public IP or port forwarding |
| [006](006-etcd-snapshots-to-cloudflare-r2.md) | etcd snapshots to Cloudflare R2, 14 days | S3-compatible, 10 GB free tier |
| [007](007-pin-versions-to-sha-and-checksums.md) | Versions pinned to SHA digests and checksums | Trivy supply chain incident, March 2026 |
| [008](008-laptop-as-control-plane-node-without-battery.md) | Laptop as a mainly control plane node, without a battery | 8 GB of RAM, risk of battery swelling when running 24/7 |
| [009](009-first-ansible-run-on-a-fresh-node.md) | First Ansible run is an explicit manual step | Target address and account do not exist on a fresh node |
