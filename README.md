# NexusPrincipia

NexusPrincipia is the backend/control plane for a future web administration interface for my homelab and self-hosted applications.

It also hosts the **shared engineering references** used across the ecosystem so that GameSaveSync, Claviger and future applications do not each carry divergent copies of the same development, documentation and operational rules.

> **Status:** early design. The backend structure and implementation stack are still provisional; the shared engineering documentation is already intended to be reusable.

## Purpose

NexusPrincipia will progressively provide the APIs and operational capabilities needed to:

- monitor application and infrastructure health;
- expose structured diagnostics and observability;
- coordinate controlled administrative operations;
- manage application configuration where appropriate;
- centralize operational views without moving each application's domain authority into NexusPrincipia;
- serve a future Web Admin frontend.

NexusPrincipia should act as a **control plane**, not as an unrestricted shortcut to host or Docker privileges.

## Shared engineering source of truth

Cross-project conventions live here rather than being copied into every application repository.

Application repositories should:

1. link to the relevant NexusPrincipia document;
2. keep only project-specific additions or exceptions locally;
3. avoid maintaining copied versions of the same generic rule.

## Documentation

Start with [docs/README.md](docs/README.md).

### Development

- [AI-assisted development operating model](docs/development/ai-development-operating-model.md)
- [Project bootstrap: technical solution + master prompt](docs/development/project-bootstrap.md)
- [Session continuity and handoff](docs/development/session-continuity.md)
- [Documentation conventions](docs/development/documentation-conventions.md)
- [C# / .NET conventions](docs/development/languages/csharp-dotnet.md)
- [Python conventions](docs/development/languages/python.md)

### Operations / architecture

- [Debug & observability reference](docs/architecture/debug-observability.md)

## Current scope

For now, this repository is both:

- the design anchor for the future NexusPrincipia backend;
- the authoritative home for shared engineering conventions already used by active projects.

Backend implementation will be introduced incrementally when concrete Admin use cases are ready rather than by pre-building speculative infrastructure.
