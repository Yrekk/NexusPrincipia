# Inspection, classification and authorized choice

**Status:** shared architecture / operations reference  
**Scope:** all ecosystem projects where technical discovery can lead to more than one legitimate semantic interpretation or administrative consequence

This rule is intentionally broader than database lifecycle handling. It applies to databases, discovered resources, imports, ownership/adoption, devices, configuration recovery, storage targets and any other workflow where the system can observe facts but may not possess all of the operator's context.

The core rule is:

> **Observe facts, constrain compatible interpretations, recommend with reasons, let an authorized actor choose, then derive safe capabilities from both the facts and that choice.**

The inspector is never the administrative authority.

## 1. Separate facts from classification

A discovery/inspection layer first records technical facts.

Examples:

~~~text
file exists = true
provider readable = true
migration history = absent
user tables = present
configured target reachable = true
~~~

Those facts are observations. An administrator does not make them false by selecting another classification.

A semantic state such as:

~~~text
Uninitialized
Invalid
Adoptable
Foreign
Configured
~~~

is a classification built from those facts plus project policy and, sometimes, human context.

Keep those layers separate.

## 2. Four layers of a safe inspection result

When classification can affect authority or later mutations, prefer a structured result conceptually similar to:

~~~text
InspectionResult
├── ObservedFacts
├── CandidateStates
├── SuggestedState?
├── Findings / evidence
└── RequiresAuthorizedDecision
~~~

After an authorized choice:

~~~text
AuthorizedClassification
├── SelectedState
├── inspection identity / revision
├── actor
├── timestamp
└── optional rationale
~~~

The exact types and names are project-specific. The separation is not.

### ObservedFacts

Facts detected directly by the system.

They should be as precise and provider-neutral as practical for the use case.

### CandidateStates

The set of semantic classifications still compatible with the observed facts and current safety policy.

This set is also a guardrail.

Do not offer a state merely because it exists in an enum.

Example:

~~~text
facts:
- valid SQLite
- no application migration history

possible:
- Uninitialized
- Invalid

not offered:
- Ready
- MigrationRequired
- TooNew
~~~

The administrator may disagree with the suggestion, but cannot choose a classification that contradicts facts or bypasses a blocking safety invariant.

### Findings / evidence

Explanations follow [Structured inspection findings](structured-inspection-findings.md): stable machine-readable codes plus structured details. Free-form UI sentences are not the authoritative application contract.

### SuggestedState

The inspector may recommend the state that best fits the evidence.

A suggestion should include the reason:

~~~text
Suggested: Uninitialized

Because:
- storage is readable
- no application schema/history exists
- no conflicting application data was detected
~~~

The UI may preselect the suggestion for convenience, but **preselection is not confirmation**.

### SelectedState

When human/operator context can legitimately alter the interpretation, the authorized actor chooses one of the candidate states.

Example:

~~~text
Inspector suggests: Uninitialized

Admin knows:
- this is a development/test file
- the application must never adopt or mutate it

Admin selects: Invalid
~~~

That choice becomes the effective administrative classification for the inspected resource.

It does not rewrite the observed facts.

## 3. No fake freedom and no fake certainty

The system must avoid both extremes.

Do not create fake freedom:

~~~text
facts prove resource is unreachable
→ do not offer Ready
~~~

Do not create fake certainty:

~~~text
facts are compatible with Uninitialized or Invalid
→ do not silently collapse them into one authoritative state
~~~

If only one candidate remains, do not invent unsafe alternatives just to populate a dropdown.

A UI may present:

~~~text
candidate: Unavailable
[ acknowledge / keep restricted ]
[ cancel ]
~~~

rather than pretending that Ready is a valid choice.

## 4. Classification is not an operation

Selecting a state does not itself execute a mutation.

Keep the sequence explicit:

~~~text
inspect
→ show facts
→ show compatible candidate states
→ suggest + explain
→ authorized selection
→ derive available capabilities
→ authorized operation, if any
→ re-inspect
~~~

Examples:

~~~text
Selected = Uninitialized
≠ initialize immediately

Selected = Invalid
≠ delete/replace immediately

Selected = MigrationRequired
≠ migrate immediately
~~~

The classification determines which operations may become available. A separate explicit operation still performs the mutation.

## 5. Downstream capabilities use facts + selected classification + policy

Do not derive permissions from the selected label alone.

Conceptually:

~~~text
AllowedCapabilities =
    f(ObservedFacts, SelectedState, AuthorizationPolicy, RuntimePolicy)
~~~

This prevents an administrative classification from overriding a physical or safety constraint.

Example:

~~~text
Admin selects Invalid for an existing file
→ current file is not considered usable authority
→ system may report that no usable application database is available
→ initialization may be offered where safe
→ existing Invalid file is NOT silently overwritten
~~~

Replacement, deletion, move, adoption or reuse of an existing resource are separate operations with their own authorization.

## 6. Human context is a legitimate input

The inspector only knows what it can observe.

The administrator may know context that is intentionally not inferable:

- a database/file is only a development fixture;
- a discovered resource belongs to another application;
- an apparently unused directory is reserved for recovery;
- a device is being reinstalled and should not be enrolled as new;
- an import target is intentionally staged but not ready for adoption.

The architecture must permit that context to influence classification **without permitting it to falsify technical facts**.

## 7. Durable classifications are control-plane state

An authorized classification may need to survive process restarts. When it does, persist the decision **outside the resource being inspected**.

This is especially important when the resource is ambiguous, rejected, foreign, invalid or not yet adopted. Writing the decision into that same resource would already treat it as trusted application authority.

Conceptually:

~~~text
AuthorizedClassification
├── resource identity
├── inspection identity / revision
├── relevant facts fingerprint or equivalent
├── candidate states at decision time
├── suggested state
├── selected state
├── actor
├── timestamp
└── optional rationale
~~~

The exact persistence mechanism is project-specific. It may initially be an application-owned control-plane store and may later move behind Nexus if Nexus acquires a real cross-application runtime responsibility.

The invariant is:

> **A durable administrative decision belongs to trusted control-plane state, not to the ambiguous resource whose meaning is being decided.**

Do not require a micro-service merely to satisfy this rule. Shared semantics + local trusted persistence remain valid until Nexus actually owns a centralized runtime responsibility.

### 7.1 Reusing a persisted classification

A persisted classification is authoritative only for the inspection context that was reviewed.

Before reusing it, the application must establish that the relevant context is still compatible. Depending on the project, this may include:

- the same resource identity;
- the same inspection/revision identity;
- the same relevant-facts fingerprint;
- a still-compatible candidate-state set;
- a selected state that is still permitted by current policy.

If material facts or classification policy change, the system must re-inspect and must not blindly reuse the old decision.

Possible invalidation signals include:

- resource content or metadata changed materially;
- migration/schema evidence changed;
- ownership/adoption evidence changed;
- the candidate-state set changed;
- the previously selected state is no longer compatible;
- the classification contract/policy changed in a way that affects the decision.

The mechanism is project-specific.

The invariant is:

> **A human choice is authoritative for the context that was reviewed, not for an arbitrarily changed resource or a materially changed classification contract.**

A stale decision may remain available for audit/history, but it must not silently restore authority.

### 7.2 Recording a decision against a fresh inspection

When the operator selects a classification from an inspection shown earlier, the application must not assume that the inspected context is still current.

Prefer an optimistic-concurrency style contract:

~~~text
inspection shown to actor
→ ExpectedInspectionRevision = ABC
→ actor selects candidate
→ application re-inspects
→ current revision still ABC ?
   ├── yes → persist AuthorizedClassification
   └── no  → reject stale decision and return fresh inspection
~~~

This protects against a time-of-check/time-of-use race where the resource changes between display and confirmation.

The invariant is:

> **A durable human classification is recorded only against the fresh inspection context that is actually being authorized.**

Adapters must pass the expected revision they presented to the actor. They must not bypass reinspection merely because the actor already confirmed an older screen.

## 8. Auditability

When a classification affects authority, mutation availability or recovery behavior, preserve enough information to explain the decision later.

Prefer recording:

- observed resource identity;
- relevant facts or an inspection reference;
- candidate states;
- suggested state;
- selected state;
- actor;
- time;
- optional reason when the actor overrides the suggestion.

For operationally important identifiers, it is valid and often useful to preserve a human-readable label snapshot next to the stable identifier:

~~~text
ActorReference = "user:190992294"
ActorLabel     = "Damien Ferrari"
~~~

The identifier remains the authority for identity, joins and authorization. The label is redundant diagnostic context only and must never become a key or source of authority.

Prefer snapshot semantics for audit records: if the display name changes later, historical records may retain the label that was visible when the event happened.

The same pattern may be used for resources, tenants, machines, profiles or other entities when it materially improves incident investigation.

This is especially important for destructive, recovery, ownership and adoption workflows.

## 9. Admin, CLI and IA/tools share the same decision contract

Different adapters must not invent different candidate sets.

~~~text
Inspector
   ↓
facts + candidates + suggestion + reasons
   ↓
Application use case / decision contract
   ├── Admin UI
   ├── CLI
   └── IA / tool
~~~

The future IA may:

- explain the evidence;
- recommend a candidate;
- highlight risks;
- prepare the operation.

It must not gain a hidden route that bypasses candidate-state restrictions or human authorization.

If a project later explicitly authorizes autonomous decisions for a bounded case, that policy still uses the same candidate set, validation, audit and operation boundaries.

## 10. Testing strategy

Test the whole decision surface, not only the inspector's preferred answer.

Useful matrix:

~~~text
observed facts
×
candidate states
×
suggested state
×
authorized selection
×
allowed capabilities
×
forbidden side effects
~~~

Tests should prove at least:

- suggestions are deterministic for identical facts;
- impossible/unsafe states are not offered;
- an authorized actor can select another compatible state;
- selecting another state does not alter the recorded facts;
- classification alone causes no mutation;
- downstream capabilities respect both facts and selection;
- materially changed facts require a fresh decision where relevant;
- every adapter receives the same candidate-state restrictions.

## 11. Relationship with project-specific state machines

This reference does not require every project to create generic types named
`CandidateState` or `AuthorizedClassification`.

Projects should keep their domain vocabulary.

What is shared is the architecture:

~~~text
facts
→ compatible interpretations
→ structured findings
→ suggestion with evidence
→ authorized choice when needed
→ constrained capabilities
→ explicit operation
→ reinspection
~~~

A project may have fully deterministic classifications where no contextual choice is legitimate. In that case the candidate set naturally contains one state.

Do not add an artificial human decision where it brings no information or safety benefit.

Use this pattern specifically where classification has meaningful consequences and operator context can legitimately distinguish otherwise compatible interpretations.
