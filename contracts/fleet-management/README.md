# AR-IMMS Fleet Management — Cross-System Contract Drafts

**Status:** Review draft (2026-11-18). **NOT a published/approved wire contract.**  
**Architecture decisions accepted:** pre-approved invitation; reuse persisted installation UUID as `agent_id`; Core submits typed monitoring intent; Agent reports evidence via authenticated HTTPS REST; Fleet Manager owns desired Collector configuration.  
**Baseline:** Telemetry Agent `chore/v0.1.4a-next` (`22b0075`), PR #40 `c9597df` from provided impact assessment; no claim that the new Fleet service or endpoints exist.

## Read order

1. [Enrollment, identity, credential lifecycle](enrollment-identity-v1.md) — Core↔Fleet and Agent↔Fleet.
2. [Configuration management](configuration-management-v1.md) — Core↔Fleet, Fleet-owned desired revisions.
3. [Inventory, health, environment evidence](inventory-health-v1.md) — Agent→Fleet HTTPS.
4. [OpAMP integration profile](opamp-integration-v1.md) — Fleet↔official Supervisor; reuse native OpAMP protocol.

## Non-negotiable component vocabulary

| Name | Meaning |
|---|---|
| **Fleet Management Server** | AR-IMMS **custom Go service** built with `opamp-go`, central VPS; fleet policy, registration, credentials, durable publications and status |
| **Official OpAMP Supervisor** | **OpenTelemetry upstream binary** pinned v0.160.0, on managed host; Collector process/config executor |
| **Telemetry Agent** | AR-IMMS Go host service; installation, secure local credentials, host evidence, Windows Exporter/LHM/Node Exporter, CLI and local Operations |
| **Core** | NestJS user-facing business API, IAM, Node/Asset authority, business audit |
| **Collector** | OTel data-plane executable |

**No Fleet Manager implementation inside Telemetry Agent.** Existing `internal/opampserver` is a transitional Core-backed adapter, *not* the new fleet service. `opamp-go` does not provide a turnkey fleet management database or policy engine. Fleet↔Supervisor uses **unmodified native OpAMP**; Collector→Gateway uses **OTLP**; existing Core→Agent typed Operations over NATS remain separate.

## Decision ledger

| Decision | State |
|---|---|
| Invitation is authorized/approved at Core creation | **ACCEPTED** |
| `agent_id` reuses persistent installation UUID | **ACCEPTED** |
| Core→Fleet configuration uses typed monitoring intent | **ACCEPTED** |
| Agent→Fleet observations use authenticated HTTPS REST | **ACCEPTED** |
| Fleet is authority for desired Collector configuration, revision/publication/observation | **ACCEPTED** |
| Exact REST paths, schemas, auth mechanism, error codes, expiry/recovery windows | **PROPOSED / OPEN** |
| Mapping `agent_id` ↔ official Supervisor `instance_uid`, config merge precedence, profile policy | **OPEN** |
| Credentials/identity migration from Core-backed enrollment | **OPEN** |

## Shared minimum wire conventions (PROPOSED)

- All proposed REST API paths below are examples, **not deployed endpoints**. Require TLS for non-loopback transport; no silent redirects for credential-bearing calls.
- JSON uses `schema_version: 1`, explicit `request_id` (trace) and `operation_id`/`attempt_id` (idempotency) as relevant. Record authoritative *server-side* actor from verified service identity, not blindly from a JSON field.
- Success response is not proof of remote application. Use `202 Accepted` for asynchronous commands and a separate status query.
- Errors should follow RFC 9457 Problem Details (`application/problem+json`) with stable documented `code`, a nonsecret `detail`, `request_id`, and `Retry-After` when meaningful. **Exact codes are draft.**
- Durable idempotency: same key + same normalized fingerprint returns the same operation/result; same key + changed request returns conflict. No unbounded retention guarantee without decision.
- No sharing/reaching into another service's MongoDB; two logical databases may use one MongoDB deployment with separate DB users. Correlate Core business audit, Fleet execution audit, and Agent local Operations journal by IDs.
- Secrets, raw machine fingerprint, private endpoint credentials and full config bytes must not go into ordinary audit or diagnostic output. Time fields UTC RFC 3339.
- **Source-of-truth separation:** `desired`, `published`, `OpAMP-reported`, `effective`, `Collector ready`, and `OTLP delivery` are different evidence axes.

## Acceptance hierarchy

1. Architecture decisions above are accepted.
2. Proposed payloads are review-ready examples; **do not implement without field-level approval**.
3. An E2E against a mock Fleet/Core validates client behavior only, not production interoperability.
4. See the **Open decisions** section in each file before implementing that boundary.

## Intended repository workflow

These files are ready to be proposed in the `docs/shared` Git submodule. **Do not modify the gitlink, publish, or overwrite existing shared contracts automatically.** Core, Fleet and Agent owners should approve revisions and then publish the final wire schemas/OpenAPI. Links to old local Core-backed OpAMP contract remain historical implementation evidence.
