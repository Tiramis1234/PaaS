# ADR 009: First Ansible run on a freshly installed node

- **Status:** Accepted
- **Recorded:** 2026-11-03

## Context

The inventory describes the target state: static addresses `10.10.10.50–52` and a shared `admin` account. A freshly installed Debian node has neither. It can only be reached at its DHCP address, using the `debian` account created during installation.

The same problem returns whenever a single node is reinstalled while the other two are already in the target state.

## Decision

The first run on a new node is an explicit, manual step:

- The repository stores only target values (`10.10.10.x` and `admin`).
- The DHCP address and the `debian` account are passed once at run time, with no file changes.
- The run is limited to that one node.
- Later runs connect using the values from the inventory.

## Alternatives considered

- **A handler triggered when connecting as `admin` fails:** rejected because it cannot work. Handlers respond to `notify` from a task that reports `changed`, and an `UNREACHABLE` host is skipped for the rest of the play.
- **Detecting in the playbook whether `admin` can log in:** rejected for now. It works, but adds logic that is rarely needed in a three-node cluster. It can be revisited if the manual step becomes a burden.
- **Creating `admin` during OS installation:** rejected because it moves part of the configuration outside Ansible, and the DHCP address would still have to be passed once.

## Consequences

- The inventory in the repository always matches the target state and contains no changing DHCP addresses.
- The procedure includes a manual step, so it has to be documented in a runbook, including how to find the DHCP address.
- After the first run, the `debian` account has to be removed or locked so it does not remain an extra way into the machine.
