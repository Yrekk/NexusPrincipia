# Architecture and operations references

This folder contains architecture/operations material that is either:

- specific to the future NexusPrincipia backend; or
- intentionally shared across applications because NexusPrincipia will become their administration/control plane.

## Current shared reference

- [Debug & observability](debug-observability.md) — structured logs, filtering, correlation, runtime Debug sessions, expiry, auditing and Admin integration.

Application-specific implementation details remain in each application repository. Shared behavior should not be copied into several repos.
