# Database lifecycle, readiness and explicit administrative choice

**Status:** shared architecture / operations reference  
**Origin:** pattern validated in Claviger and generalized for the shared ecosystem

This reference applies when an application database or other authoritative persisted store has a lifecycle that may require initialization, migration, recovery or ownership validation.

The core rule is:

> **Observe state, derive safe capabilities, let an authorized actor choose the operation.**

The application must not collapse those three concerns into one automatic startup decision.

## 1. Separate three concepts

A robust runtime keeps these dimensions distinct:

~~~text
1. observed technical state
   "what is true right now?"

2. operational/readiness mode
   "what is the application currently allowed to do?"

3. exposed administrative actions
   "which explicit operations may an authorized actor request?"
~~~

Do not encode an administrative decision inside the technical state.

For example:

~~~text
DatabaseState = Missing
~~~

must not mean:

~~~text
therefore initialize automatically
~~~

A missing database may represent a first installation, a missing mount/volume, a move in progress, an interrupted restore, operator error or another incident.

## 2. Shared database-state vocabulary

Claviger currently validates the following technical states:

~~~text
Missing
Uninitialized
Ready
MigrationRequired
TooNew
Unavailable
~~~

These names form a useful shared baseline.

### Missing

The configured database file/store does not exist.

This is an observation only.

Possible project-specific actions may include initialization, restoring a known snapshot, retrying after infrastructure repair or doing nothing while maintenance is in progress.

### Uninitialized

The database/store exists but the expected application schema has not been initialized.

This is distinct from a missing database.

### Ready

The technical database/schema state matches the version understood by the application.

**Ready database does not automatically mean ready application.**

Additional readiness dimensions may still block normal operation, such as:

- ownership/binding to the current application instance;
- incomplete administrative configuration;
- unavailable storage;
- missing secrets/credentials;
- domain-specific consistency checks.

### MigrationRequired

The database is valid and older than the schema version expected by the application.

The runtime may report the pending migration state, but migration remains an explicit administrative operation unless a project has separately accepted a stronger autonomous-safety policy.

### TooNew

The database/schema is newer than the application knows how to handle.

Typical cause: an older application version was started against a database already upgraded by a newer version.

Fail closed.

Do not attempt a down migration automatically.

### Unavailable

The database exists or is configured, but cannot currently be opened/read reliably.

Examples include locking beyond policy, permissions, filesystem/storage failure or provider errors.

Do not reinterpret this as Missing.

## 3. Project-specific extensions

Projects may add states only when they represent a genuinely different observation.

Example for systems with stronger integrity requirements:

~~~text
Invalid
~~~

may mean the database is reachable but failed an integrity or consistency check.

Avoid states that are actually proposed actions, such as:

~~~text
RestoreRequired
InitializeRequired
MigrationFailed
~~~

Those are decisions or operation results, not intrinsic database state.

## 4. Structured status result

The inspection layer should return a structured result instead of a bool.

A typical shape is:

~~~text
DatabaseStatus
├── State
├── CurrentVersion?
├── TargetVersion
└── optional diagnostics / reason
~~~

The inspection operation is read-only.

It may inspect:

- existence;
- reachability;
- current schema/migration version;
- expected version;
- provider/schema compatibility;
- project-specific integrity checks.

It must not initialize, migrate, restore, bind or repair the database as a side effect.

## 5. Database readiness is not application readiness

Claviger demonstrates an important separation:

~~~text
database = Ready
+
database ownership = valid
+
guild ADMIN configuration = complete
→ guild/runtime features may be Ready
~~~

A ready database may still produce a restricted application.

General rule:

> **Each readiness dimension remains explicit until a real use case combines them.**

Do not overload one database state enum with storage readiness, account ownership, per-tenant configuration or other unrelated concerns.

## 6. Ownership / authority binding

Some applications need to prove that a persisted store belongs to the current logical application instance.

Claviger uses explicit database ownership binding and fails closed when another Discord application owns the database.

Generalized rule:

- unbound authority may require an explicit bind/adopt operation;
- same-owner validation may be idempotent;
- ownership mismatch must not be silently overwritten;
- ownership is a separate readiness dimension from schema readiness.

Only use this pattern where a real application-identity collision risk exists.

## 7. Action availability follows state and policy

The UI/CLI/agent should expose only actions that are meaningful and allowed for the current state.

Claviger validates a simple matrix:

~~~text
Missing / Uninitialized
→ status
→ initialize

MigrationRequired
→ status
→ migrate

Ready but unbound
→ status
→ bind

Ready and correctly bound
→ status

TooNew / Unavailable
→ status only
~~~

This matrix is not a universal list of buttons. It demonstrates the rule:

> **unsafe or irrelevant mutating operations should not be offered merely because an implementation exists.**

A project with snapshots may expose more than one valid action.

Example:

~~~text
Missing
→ status
→ initialize
→ list/restore snapshot
→ retry inspection
~~~

The runtime still does not choose between them automatically.

## 8. Recommended action is guidance, not execution

A status response may contain guidance:

~~~text
State: MigrationRequired
Recommendation: review and run the migration operation
~~~

but guidance must not itself trigger mutation.

This distinction is useful for:

- Admin UI;
- CLI;
- human operators;
- future IA/tools.

## 9. Startup and degraded/minimal operation

Startup may inspect database status and derive the maximum safe runtime capability.

Examples:

~~~text
Ready
→ normal runtime may continue if other readiness checks pass

MigrationRequired
→ normal authority blocked
→ maintenance/admin capabilities remain available

TooNew
→ fail closed
→ diagnostic/status surface only

Unavailable / Invalid
→ restricted recovery or diagnostic mode
~~~

The exact mode names are project-specific.

Common useful vocabulary includes:

~~~text
Normal
Maintenance
RestrictedRecovery
~~~

The important rule is that **mode describes allowed runtime capability**, not what operation should be executed next.

## 10. Initialization, migration and restore are different operations

Even when they touch the same database, keep these capabilities separate:

~~~text
InitializeDatabase
ApplyPendingMigrations
RestoreDatabaseSnapshot
~~~

They have different preconditions, failure modes and operator intent.

Do not create one broad Maintenance capability that can silently switch behavior based on what it finds.

A migration path must not create a missing database merely because the provider can do so.

A restore path must not silently pick the newest snapshot.

## 11. Safe-state transitions require reinspection

After any mutating administrative operation:

~~~text
inspect
→ explicit operation
→ execute
→ inspect again
→ validate expected final state
~~~

Do not assume success merely because the mutation method returned without throwing.

Claviger follows this pattern after initialization/migration by checking that the final database status is actually Ready.

Projects with stronger requirements may add domain/integrity validation before returning to normal authority.

## 12. Fail-closed rules

At minimum:

- do not mutate a TooNew database;
- do not reinterpret Unavailable as Missing;
- do not auto-create an authoritative database because the expected file is absent;
- do not continue normal authority when schema compatibility is unknown;
- do not overwrite an ownership mismatch;
- do not expose unsafe mutating actions in states where their preconditions are not satisfied.

Diagnostic availability may remain while normal application authority is disabled.

## 13. Testing strategy: state × action matrix

Do not test only the happy path.

Tests should cover:

~~~text
observed state
×
exposed/allowed action
×
forbidden side effect
×
final reinspection
~~~

Examples:

- Missing reports Missing without creating a database.
- MigrationRequired does not invoke initialization.
- Initialize does not run against an already-migrated database.
- Migrate does not run against Missing/Uninitialized.
- TooNew exposes no mutating lifecycle operation.
- Unavailable exposes no unsafe mutation.
- successful initialize/migrate is followed by a final status check.
- ownership mismatch fails closed where ownership binding exists.

A capability-availability test is valuable because it catches accidental exposure of dangerous commands even when the underlying service itself is safe.

## 14. Admin, CLI and IA/tools use the same operations

The future Admin interface, CLI and IA agent are adapters over the same application-level capabilities.

~~~text
Admin ──────┐
CLI ────────┤
IA / tool ──┤
startup ────┘
      ↓
inspection + explicit application use cases
      ↓
persistence / infrastructure
~~~

An IA does not receive a hidden shortcut.

If a future project authorizes autonomous operations, that autonomy must still use the same guarded use cases, authorization, validation, audit and recovery controls.

## 15. What is generic vs project-specific

Shared here:

- separation of observation, readiness and action;
- explicit state vocabulary baseline;
- fail-closed posture;
- explicit user/operator action;
- action availability derived from state;
- final reinspection;
- adapter reuse;
- state × action testing.

Project-specific:

- exact additional states;
- exact operational mode names;
- snapshot/recovery policy;
- authorization model;
- ownership/binding semantics;
- domain consistency checks;
- which actions are available for each state.

Application repositories should link to this reference and document only their additions/exceptions.
