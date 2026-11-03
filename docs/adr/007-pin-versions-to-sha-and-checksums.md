# ADR 007: Pin versions to SHA digests and checksums

- **Status:** Accepted
- **Recorded:** 2026-11-03

## Context

The Trivy supply chain incident of March 2026 showed that even a security tool can become an attack path. A mutable tag or an unpinned installer can change what runs without any change in this repository.

## Decision

Pin tools to checksums, and pin CI actions and container images to SHA digests.

## Consequences

- Builds and installs are reproducible, and an upstream change does not reach the project unnoticed.
- Updates become a deliberate, manual step, so pinned versions need regular review to avoid falling behind on security fixes.
