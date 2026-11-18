# Fleet Management verification matrix

**Status:** Acceptance checklist; does not approve draft API fields.
[C01–C04 are canonical](../../contracts/fleet-management/README.md).

## Evidence baseline

**Mock Fleet: PASS.** Telemetry Agent PR #40 head
a839893cb41cfcbbfb9163f4bbc3e764404ced73 passed Windows/Ubuntu
[CI 44388896953](https://github.com/AR-IMMS/telemetry-agent/actions/runs/44388896953).
Merged integration commit 15cbe300ef64fa96d1eeacc53800957f1a6f7f61 passed
[CI 44391516366](https://github.com/AR-IMMS/telemetry-agent/actions/runs/44391516366).

Recorded coverage includes protected enrollment application tests and managed
Supervisor/Collector fallback, remote configuration, rollback, reconnect and
revocation tests using provisional fixtures. These exercise Agent behavior and
pinned runtime integration, not a production Fleet implementation.

**Production Fleet/NestJS interoperability: NOT VERIFIED.**
Draft endpoint/schema conformance is not established by Mock Fleet PASS.
Hosted Windows Docker-dependent checks were skipped; Ubuntu ran live Docker,
JetStream and profile/OTLP E2Es. Do not convert those skips into Windows PASS.
PR-head race checks passed both OS; integration push race checks were skipped.

## Required production acceptance

| Flow / concern | Required observable result | Contract | Production evidence |
|---|---|---|---|
| Enrollment, approval and connection | Core-authorized invitation, atomic exchange/recovery, stable installation and authenticated UID binding; replay rejected | C01/C04 | NOT VERIFIED |
| Health, capabilities and inventory | Authenticated scoped evidence, provenance/freshness; partial/failed/old snapshots preserve newer records | C03/C04 | NOT VERIFIED |
| Configuration publication/application | Durable desired/publication mapping, connected-session update, CAS/idempotency, unchanged no-op; actual OTLP sink reception | C02/C04 | NOT VERIFIED |
| Connection loss during apply | Outcome stays UNKNOWN until observed; restart/reconnect resolves hash against durable history and latest desired | C02/C04 | NOT VERIFIED |
| Invalid configuration/rollback | Previous working hash attributed correctly; latest desired not falsely APPLIED; collection/delivery restored | C02/C04 | NOT VERIFIED |
| Online credential revocation | Persist revoke before disconnect; matching sessions close; reconnect/config delivery denied | C01/C04 | NOT VERIFIED |
| Composition and dependencies | Local/remote/fallback collision tests; protected secrets; reachable managed exporter endpoints; one receiver/process owner | C02/C04 | NOT VERIFIED |
| Migration and compatibility | Existing IDs, CLI, SQLite, OS exporters and Operations preserved; explicit rollback on Windows/Linux | C01–C04 | NOT VERIFIED |

## Recording rules

Record source commit, OS, pinned binary versions/digests, commands, executed tests,
sanitized logs/artifacts and expected negative security assertions. Use PASS,
FAIL, BLOCKED or NOT_TESTED per scenario/platform; record exact missing contract
when blocked. Historical evidence must name its commit and scope.

Keep accepted intent, published revision/hash, reported applied hash, effective
configuration, Collector readiness, collection and OTLP delivery separate.
APPLIED or readiness alone does not prove delivery. UNKNOWN/unattributed/stale
evidence must remain visible. Test credential/revocation races, duplicate
requests and recovery after persistence/network failures.

The detailed acceptance rules and OPEN decisions remain in C01–C04; resolve
them there before declaring production acceptance.
