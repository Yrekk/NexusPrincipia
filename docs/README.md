# Shared documentation

This directory contains both NexusPrincipia-specific architecture and cross-project engineering references.

## Source-of-truth rule

A rule that applies to several applications should normally live here once.

A project repository should then keep a short local document that links here and records only project-specific additions or explicit exceptions.

Do not maintain copied versions of the same generic development document in several repositories.

## Development references

- [AI-assisted development operating model](development/ai-development-operating-model.md)
- [Project bootstrap](development/project-bootstrap.md)
- [Session continuity](development/session-continuity.md)
- [Documentation conventions](development/documentation-conventions.md)
- [Entrypoints and reusable administrative operations](development/entrypoints-and-reusable-operations.md)
- [Language-specific conventions](development/languages/)

## Architecture / operations references

- [Debug & observability](architecture/debug-observability.md)
- [Database lifecycle, readiness and explicit administrative choice](architecture/database-lifecycle-readiness.md)
