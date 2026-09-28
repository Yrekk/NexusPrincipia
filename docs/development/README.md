# Development standards

This folder is the shared source of truth for development practices used across the ecosystem.

## Documents

- [AI-assisted development operating model](ai-development-operating-model.md) — tranche-by-tranche collaboration with a mandatory post-validation code/architecture review before acceptance; trade-off questions are used to make architectural choices explicit rather than asking the developer to reproduce the AI's reasoning.
- [Project bootstrap](project-bootstrap.md) — start non-trivial projects from an established Solution Technique and a dedicated master prompt.
- [Session continuity](session-continuity.md) — handoff rules and inter-session recovery.
- [Documentation conventions](documentation-conventions.md) — comments, local READMEs, ADRs and living documentation.
- [Entrypoints and reusable administrative operations](entrypoints-and-reusable-operations.md) — keep startup files as adapters/composition roots; reusable Admin/CLI/AI capabilities live in services/use cases.
- [Language-specific conventions](languages/) — stack-specific guidance.

Project repositories should link here and keep only their local additions or explicit exceptions.


## Default development cycle

Every coherent development follows the shared loop:

~~~text
scope
→ implementation
→ technical validation and fixes
→ shared code / architecture review
→ trade-off discussion
→ structural correction if needed
→ explicit acceptance
→ next tranche
~~~

Project repositories may add constraints, but should not skip the
review-before-acceptance step.
