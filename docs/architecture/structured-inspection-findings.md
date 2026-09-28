# Structured inspection findings

**Status:** shared architecture / contract reference  
**Scope:** all ecosystem applications that expose diagnostics, discovery, reconciliation or lifecycle inspection to Admin, CLI, API or IA/tool adapters

This reference standardizes how an application explains *why* it produced an inspection result.

The core rule is:

> **Machine-readable findings are part of the application contract; human-readable sentences belong to presentation adapters.**

## 1. Why free-form strings are not an application contract

Avoid application/domain results such as:

~~~text
Reasons:
- "No applied migration was found."
- "Two unrelated tables are present."
~~~

Those strings are useful to a human but poor as a stable contract:

- wording changes break consumers;
- localization becomes difficult;
- UI/CLI/IA adapters start parsing prose;
- tests assert sentences instead of meaning;
- two applications describe the same condition differently.

The shared model is instead:

~~~text
Finding
├── code
└── details
~~~

Example:

~~~json
{
  "code": "database.schema.application_history_absent",
  "details": {}
}
~~~

~~~json
{
  "code": "database.schema.non_application_objects_present",
  "details": {
    "object_count": 2
  }
}
~~~

## 2. Canonical finding shape

At the transport/control-plane boundary, one finding has this conceptual shape:

~~~text
InspectionFinding
├── code: stable namespaced string
└── details: string → JSON-safe value
~~~

Allowed detail values should remain transport-safe:

- string;
- integer;
- decimal number when required;
- boolean;
- null;
- arrays of those primitive values.

Do not put framework/provider objects, exceptions or arbitrary nested application models into `details`.

Applications may use strongly typed native models internally. The wire representation must still map cleanly to `code + details`.

## 3. Stable codes

Finding codes are API-like identifiers.

Use lowercase namespaced dot notation:

~~~text
database.resource.missing
database.integrity.failed
admin.category.multiple_candidates
workflow.channel.missing
~~~

Rules:

- codes describe meaning, not UI wording;
- changing a sentence does not change the code;
- renaming a code is a contract change;
- prefer one stable code + structured details over dynamically generated codes;
- project-specific codes use a project/domain namespace when no shared semantic exists.

## 4. Findings are evidence, not permissions

A finding does not itself grant or deny an operation.

Do not encode:

~~~text
finding.code = "initialize_allowed"
~~~

when the actual observation is:

~~~text
database.schema.application_history_absent
~~~

Allowed capabilities remain derived from:

~~~text
observed facts
+ authorized classification
+ authorization policy
+ runtime policy
~~~

This prevents diagnostic wording from becoming hidden business logic.

## 5. Presentation belongs to adapters

Adapters map codes and details to human text.

Example:

~~~text
code:
database.schema.non_application_objects_present

Admin UI FR:
"2 objets SQLite non attribués à l'application ont été détectés."

CLI EN:
"Detected 2 SQLite objects not owned by the application."

IA/tool:
explains the same code and details conversationally
~~~

The application contract does not need to carry every translated sentence.

A presentation adapter may expose a fallback human message externally for resilience, but that message is not authoritative and must not be parsed to recover semantics.

## 6. Findings remain separate from observed facts

Facts and findings serve different purposes.

~~~text
ObservedFacts
├── file_exists = true
├── accessible = true
├── user_object_count = 2
└── current_version = 0

Findings
├── database.schema.application_history_absent
└── database.schema.non_application_objects_present { object_count: 2 }
~~~

Facts are raw observations.

Findings are stable semantic evidence derived from those facts.

Classification then consumes both:

~~~text
facts
→ findings
→ candidate states
→ suggested state
→ authorized choice when ambiguity exists
~~~

Do not mutate facts to match the selected classification.

## 7. Shared database finding codes

Database-backed applications should reuse these codes where the semantics match.

### Resource

~~~text
database.resource.missing
database.resource.path_not_file
database.resource.unavailable
~~~

Common details:

~~~text
provider_error_code?
reason_kind?
~~~

### Integrity

~~~text
database.integrity.failed
~~~

### Schema / ownership evidence

~~~text
database.schema.uninitialized
database.schema.application_history_absent
database.schema.user_objects_absent
database.schema.non_application_objects_present
database.schema.current
database.schema.outdated
database.schema.newer_than_runtime
database.schema.history_inconsistent
database.schema.read_failed
~~~

Common details:

~~~text
current_version?
target_version?
pending_count?
unknown_count?
object_count?
identifier?
provider_error_code?
~~~

Applications do not have to emit every code. They reuse the codes that fit their persistence technology and add namespaced codes only for real project-specific semantics.

## 8. Database state vocabulary

The shared lifecycle classification remains:

~~~text
Missing
Uninitialized
Ready
MigrationRequired
TooNew
Unavailable
Invalid
~~~

A database inspection result should conceptually expose:

~~~text
DatabaseInspection
├── facts
├── findings
├── candidate_states
├── suggested_state
└── requires_authorized_classification
~~~

Do not include `Invalid` as a generic "I do not want to use this resource" escape hatch when the technical/application identity is already proven.

Example:

~~~text
valid application migration history
+ exact current schema
→ Ready
~~~

If an Admin wants another database, that is resource selection/configuration, not a false `Invalid` classification.

Use multiple candidates only when the same observed facts genuinely support multiple semantic interpretations.

## 9. Cross-project compatibility

The shared contract is semantic, not language-specific.

C# may use:

~~~text
record / enum / immutable collections
~~~

Python may use:

~~~text
dataclass / StrEnum / tuple / Mapping
~~~

The important compatibility point is the serialized/control-plane meaning:

~~~text
code
details
candidate states
suggested state
facts
~~~

Do not force projects into a common binary package merely to share this vocabulary.

## 10. When this should become a shared library or service

### Shared library

A shared package becomes justified only when several applications in the same runtime/language need identical executable validation or serialization logic and versioning the package is cheaper than keeping small native implementations.

Documentation + compatibility tests are sufficient before that point.

### Micro-service / Nexus runtime capability

Do **not** create a micro-service merely because several applications expose the same states.

A networked shared service becomes justified when at least one real runtime responsibility becomes centralized, for example:

- Nexus remotely orchestrates inspections across multiple applications;
- classification decisions are centrally persisted and audited;
- authorization for cross-application administrative operations is centralized;
- a common operation must execute independently of an application's process;
- several consumers require the same live state over a stable network API.

Until then:

~~~text
shared contract
+ local implementation
+ future Nexus aggregation
~~~

is simpler and safer than:

~~~text
application
→ network
→ generic database micro-service
→ local database
~~~

The latter would add availability, authentication, deployment and failure modes without providing authority that Nexus actually needs yet.

## 11. Testing

Tests should assert codes and details, not presentation sentences.

Prefer:

~~~text
finding.code == "database.schema.outdated"
finding.details["pending_count"] == 2
~~~

over:

~~~text
"2 migrations are pending" in message
~~~

Presentation adapters may have separate tests for rendered human text.

For shared semantics, each application should also test that:

- canonical states map consistently;
- canonical finding codes use the documented meaning;
- candidate states are constrained by facts;
- no mutation occurs during inspection;
- human/presentation formatting is downstream from structured results.
