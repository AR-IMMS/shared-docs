# Telemetry Agent handoff to Fleet Management

**Status:** Current Agent evidence plus ACCEPTED target boundaries.
[Canonical contracts](../../contracts/fleet-management/README.md).
[Architecture summary](../../architecture/fleet-management/overview.md).

## Current integration baseline

Agent PR #40 merged as 15cbe300ef64fa96d1eeacc53800957f1a6f7f61.
The existing internal/opampserver adapter is transitional Core-backed integration,
not the production Fleet Management Server.

**Mock Fleet: PASS; production Fleet/NestJS integration: NOT VERIFIED.**
Test-only internal/testsupport/mockfleet uses provisional examples authorized for
fixtures. It must not enter production runtime or define the final API.
See the [verification matrix](verification-matrix.md) for runs and limitations.

## What Fleet can rely on as integration intent

| Agent responsibility | Fleet interaction / preservation requirement |
|---|---|
| Persisted installation UUID and environment evidence | Reuse agent_id; treat host_id and environment as evidence, not authentication or confirmed topology |
| Enrollment application and OS-protected credential storage | Approved exchange/recovery contract supplies credential and Supervisor endpoint; preserve interrupted-attempt recovery |
| Official Supervisor provisioning | Pinned Supervisor v0.160.0 connects using native OpAMP; preserve authenticated binding and secret protection |
| OS integration and managed exporters | Windows Exporter, Libre Hardware Monitor and Node Exporter remain Agent-owned; Fleet consumes approved endpoint observations |
| Local inventory and health | Remain usable offline; Fleet stores scoped observation projections with freshness |
| CLI and SQLite | Preserve command/flag/output/exit compatibility, migration history and durable local safety records |
| Infrastructure Operations | Existing Core-authorized NATS channel and Agent safety/audit/idempotency remain separate from Fleet and OpAMP restart |

Collector Contrib and official Supervisor are pinned at v0.160.0 for the recorded
integration. Verify pinned capability/config behavior rather than assume upstream
specification support is enabled. Supervisor package updates and restart commands
are outside the baseline integration scope.

## Production connection prerequisites

Implement approved C01–C04 contracts in the separate Fleet service, then configure
Agent endpoints through the approved client boundary. A provisional mock's URL
or schema is not a production interoperability promise. Exact production routes,
auth scopes, secure rotation delivery, UID mapping and retention remain OPEN
where their contracts say so.

Keep legacy Agent-managed Collector, current Core-backed Supervisor mode and
future Fleet-backed Supervisor mode distinguishable. Supervisor.Enabled is the
lifecycle selector, not Fleet configuration authority. Current Agent writers stay
operational until composition and migration cutover gates pass.

Fleet supplies desired policy and publication/history; Supervisor executes
Collector config/lifecycle. Agent supplies secure bootstrap, local dependency
inputs and fallback provisioning. Agree component ownership, Gateway routing,
credential references and revision/hash attribution before cutover. Complete
inventory/health reporting can proceed separately from essential config inputs.

## Reuse and verification

Fleet developers can inspect Agent test fixtures and CI artifacts at the recorded
commit to reproduce enrollment and Supervisor remote-config/rollback/revocation
scenarios. Port their acceptance intent to the real service; do not copy the
in-memory fixture into production or claim durable Fleet restart behavior from it.

Validate all six flows against real Fleet/Core, official Supervisor, Collector
and an isolated OTLP sink. Preserve Windows/Linux service install/upgrade/remove,
OS exporter lifecycle, explicit existing-installation migration and safe rollback.
Implementation details and service persistence belong in the Fleet repository.
