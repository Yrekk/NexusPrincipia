# Development standards

This folder is the shared source of truth for development practices used across the ecosystem.

## Documents

- [AI-assisted development operating model](ai-development-operating-model.md) — one functional tranche per dedicated session, with fluid internal checkpoints, continuous testing and a targeted code/architecture review before tranche acceptance.
- [Project bootstrap](project-bootstrap.md) — start non-trivial projects from an established Solution Technique and a dedicated master prompt.
- [Session continuity](session-continuity.md) — handoff rules and inter-session recovery.
- [Documentation conventions](documentation-conventions.md) — comments, local READMEs, ADRs and living documentation.
- [Entrypoints and reusable administrative operations](entrypoints-and-reusable-operations.md) — keep startup files as adapters/composition roots; reusable Admin/CLI/AI capabilities live in services/use cases.
- [Language-specific conventions](languages/) — stack-specific guidance.

Project repositories should link here and keep only their local additions or explicit exceptions.


## Default development cycle

Every functional tranche follows the shared loop:

~~~text
dedicated session
→ light scope
→ continuous implementation through internal checkpoints
→ technical validation and fixes
→ targeted code / architecture review
→ structural correction if needed
→ explicit tranche acceptance
→ documentation / handoff
→ new session for the next tranche
~~~

Internal checkpoints are not mini-tranches and do not require their own acceptance ceremony.
