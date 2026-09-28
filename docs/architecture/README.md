# Architecture and operations references

This folder contains architecture/operations material that is either:

- specific to the future NexusPrincipia backend; or
- intentionally shared across applications because NexusPrincipia will become their administration/control plane.

## Current shared references

- [Debug & observability](debug-observability.md) — structured logs, filtering, correlation, runtime Debug sessions, expiry, auditing and Admin integration.
- [Inspection, classification and authorized choice](inspection-classification-authority.md) — separate observed facts from semantic classification, constrain candidate states, explain suggestions, preserve Admin authority and derive capabilities safely.
- [Database lifecycle, readiness and explicit administrative choice](database-lifecycle-readiness.md) — observed DB state, operational readiness, safe action availability, fail-closed behavior and Admin/CLI/IA reuse.

Application-specific implementation details remain in each application repository. Shared behavior should not be copied into several repos.
