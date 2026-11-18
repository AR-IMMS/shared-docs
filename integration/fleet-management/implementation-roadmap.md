# Fleet Management implementation roadmap

**Status:** PROPOSED implementation sequence under ACCEPTED B+ ownership.
[Contracts are authoritative](../../contracts/fleet-management/README.md);
this roadmap does not approve their draft wire details.

## Small implementation checkpoints

| Order | Deliverable | Dependencies / decision gate | Acceptance |
|---|---|---|---|
| 1 | Enrollment and authenticated session vertical slice | Approve C01 exchange/recovery proof, Core service authentication and UID binding | One invitation exchange; same-attempt recovery; protected Agent credential; authenticated official Supervisor session; replay and wrong binding rejected |
| 2 | Durable configuration publication | Approve C02 intent schemas, revision/hash semantics and C04 composition boundary | CAS/idempotency; persist publication before send; update an already-connected Supervisor; actual Collector-to-OTLP delivery |
| 3 | Reconnect, rollback and credential enforcement | Checkpoints 1–2; approve revocation/rotation and failed-desired retry policy | Fleet restart preserves attribution; disconnect remains UNKNOWN until evidence; invalid config restores working collection; revoke closes sessions and rejects reconnect |
| 4 | Inventory and health projection | Approve C03 payload, scope, ordering and freshness | COMPLETE/PARTIAL/FAILED preserved; old snapshots cannot delete newer resources; stale evidence becomes UNKNOWN |
| 5 | Production interoperability and controlled cutover | Operational Fleet/Core; approved migration and configuration dependencies | Native Windows/Linux integration; legacy/current Core-backed behavior preserved; exclusive Collector ownership; rollback exercised |

Inventory/health development can proceed independently after its contract is
approved. Full reporting is not a configuration-cutover prerequisite: only
dependency endpoint observations actually required to build safe configuration
must be available.

## Keep implementation bounded

Use opamp-go for protocol transport and official Supervisor for Collector
execution. Add application/persistence boundaries for the responsibilities Fleet
actually owns. Do not begin with a generic plugin framework, registry CRUD suite,
custom discovery/scraping engine, or remote package manager.

Fleet implementation code, database layout and deployment details belong in the
Fleet repository. Persist desired state and publication mappings durably; process
events are wakeups, not the sole record of work. Select atomicity mechanisms
appropriate to the actual database deployment; do not assume standalone MongoDB
supports multi-document transactions or change streams.

## Cutover prerequisites

- Owner-approved enrollment/authentication, OpAMP binding and configuration contracts.
- Verified pinned local/remote/fallback composition, component IDs, secret handling
  and managed dependency endpoint readiness.
- Explicit migration for existing identity, protected credentials and configuration.
- Configuration update, reconnect, rollback, revocation and real OTLP E2Es pass.
- One persisted lifecycle owner; controlled rollback; no simultaneous legacy/native execution.

Keep Core-backed native configuration operational until these pass. Do not infer
Fleet authority from Supervisor.Enabled. Defer overlapping implementation removal
until migration, compatibility and recovery evidence justify it.

## Decision ownership

C01: Core/Fleet/Agent owners approve enrollment, credential scope and migration.
C02: Fleet/Core owners approve intent, policy and routing; Agent/Fleet owners
agree required local endpoint inputs. C03: Agent/Fleet/Core owners approve
observation semantics. C04: Fleet/Agent owners approve the supported integration
profile using pinned upstream evidence. Record decisions in those contracts.

**Mock Fleet: PASS** for provisional recorded tests; **production: NOT VERIFIED**.
Use the [verification matrix](verification-matrix.md) for evidence and release gates.
