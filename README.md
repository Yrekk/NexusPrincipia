# NexusPrincipia

NexusPrincipia is the backend/control plane for a future web administration interface for my homelab and self-hosted applications.

> **Status:** early design. The repository structure and implementation stack are intentionally still provisional.

## Purpose

NexusPrincipia is intended to provide one administration surface for current and future services such as GameSaveSync, Claviger and other homelab applications.

The backend will progressively provide the APIs and operational capabilities needed to:

- monitor application and infrastructure health;
- expose structured diagnostics and observability;
- coordinate controlled administrative operations;
- manage application configuration where appropriate;
- centralize operational views without moving each application's domain authority into NexusPrincipia;
- serve a future Web Admin frontend.

NexusPrincipia should act as a **control plane**, not as an unrestricted shortcut to host or Docker privileges. Applications remain responsible for their own domain rules and should expose explicit administrative capabilities that NexusPrincipia can consume.

## Design direction

Early cross-application principles include:

- structured and filterable logs;
- correlation IDs for end-to-end investigations;
- runtime Debug sessions with automatic expiry;
- scheduled or explicitly persistent Debug when required;
- secret redaction even at Debug/Trace level;
- auditable administrative actions;
- local rotating logs as a fallback when the Admin interface is unavailable;
- controlled orchestration instead of giving the Web Admin unrestricted Docker access.

## Documentation

The documentation layout is temporary and will be reorganized as the backend architecture becomes concrete.

- [Debug & observability reference](docs/architecture/debug-observability.md) — cross-application reference for runtime logging, Debug sessions and future Admin diagnostics.

## Current scope

For now, this repository is primarily a design anchor for the future backend. Implementation will be introduced incrementally when the first concrete Admin use cases are ready, rather than pre-building speculative infrastructure.
