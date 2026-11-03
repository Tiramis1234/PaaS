# ADR 008: Laptop as a mainly control plane node, without a battery

- **Status:** Accepted
- **Recorded:** 2026-11-03

## Context

The third etcd member is a laptop with 8 GB of DDR3 RAM and a 4th generation i7. It runs around the clock, and a battery kept in a machine running 24/7 risks swelling.

## Decision

Run the laptop as a k3s server with a taint, so it mainly holds the control plane and heavier workloads go to the ThinkStations. Remove the battery, prevent sleep when the lid closes, disable Wi-Fi and use only the wired connection.

## Consequences

- The laptop provides the third etcd member without carrying heavy workloads.
- With no battery, a power cut stops the laptop along with the other nodes, so all three go down together. The nodes need to start automatically when power returns, where the hardware supports it, and a cold start of the cluster has to be tested.
- Temperature needs monitoring, because the machine runs continuously in a laptop chassis.
