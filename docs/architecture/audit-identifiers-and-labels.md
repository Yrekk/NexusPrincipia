# Audit identifiers and human-readable labels

**Status:** shared architecture / operations reference  
**Scope:** persisted audit, control-plane history, structured logs and operational records across ecosystem applications

The core rule is:

> **Stable identifiers preserve technical identity; human-readable labels may be stored alongside them as diagnostic snapshots when that materially improves investigation.**

## 1. Stable identity remains authoritative

Identifiers are used for:

- identity;
- joins and references;
- authorization;
- deduplication;
- machine-readable correlation.

Example:

~~~text
ActorReference = "user:190992294"
~~~

The identifier remains the source of authority.

Do not use a display label as a key merely because it is easier to read.

## 2. Human-readable labels are valid redundant context

Operational records may also preserve a human-readable label:

~~~text
ActorReference = "user:190992294"
ActorLabel     = "Damien Ferrari"
~~~

This redundancy is intentional when it reduces incident/debug cost.

An operator should not need to remember or manually resolve every opaque identifier just to understand an audit trail.

Useful examples include:

- actor/user names;
- machine names;
- game profile names;
- guild/server names;
- contact labels;
- resource descriptions;
- application-instance labels.

## 3. Labels never replace identifiers

Valid:

~~~text
ActorReference = stable identity
ActorLabel     = human context
~~~

Invalid:

~~~text
ActorLabel used as authorization identity
ActorLabel used as unique key
ActorLabel used to join authoritative records
~~~

Labels are presentation/diagnostic context only.

## 4. Prefer snapshot semantics in audit history

For historical/audit records, the label may represent what was visible at the time of the event.

If a user or resource is renamed later, do not silently rewrite old audit history merely to show the newest label.

Example:

~~~text
2026-09-28
ActorReference = "user:190992294"
ActorLabel     = "Damien Ferrari"
~~~

If the account is later renamed to `Damien F.`, the old decision may keep the original label because it explains the historical event as it occurred.

## 5. Store labels only when they materially help

Do not duplicate every field everywhere.

A redundant label is justified when:

- identifiers are opaque to humans;
- records may be inspected manually;
- incident investigation would otherwise require extra joins/lookups;
- audit history should remain understandable after surrounding data changes;
- the storage cost is negligible relative to the operational value.

This is especially appropriate for low-volume control-plane and audit records.

## 6. Structured logs and control-plane records

The same pattern applies to both persisted history and structured logs.

Examples:

~~~text
MachineId    = "01..."
MachineLabel = "WALL-E"

ProfileId    = "..."
ProfileLabel = "Project Zomboid"

ActorReference = "admin:..."
ActorLabel     = "Damien Ferrari"
~~~

When a label is unavailable, do not fabricate one. The stable identifier remains sufficient for machine correctness.

## 7. Privacy and sensitive data

Human readability does not justify copying unnecessary sensitive data.

Labels should remain limited to operationally useful, non-secret context.

Do not duplicate:

- credentials;
- tokens;
- secrets;
- passwords;
- unrelated personal data.

## 8. Testing

When labels are persisted with authoritative identifiers, tests should prove:

- identity logic uses the stable identifier, not the label;
- changing a label does not change technical identity;
- historical labels may remain unchanged after later renames;
- missing labels do not break identity logic;
- labels are preserved when required for audit readability.

## 9. Project-specific use

Applications should reference this rule and document only their local field names and exceptions.

The general pattern is:

~~~text
StableIdentifier
+
optional/required HumanReadableLabelSnapshot
~~~

The exact identifier and label types remain project-specific.
