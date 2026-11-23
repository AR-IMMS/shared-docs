# Agent Registration v1 — Shared Review Proposal

**Status: PROPOSED, not approved.** Architecture ownership is accepted; wire details need Core, Fleet, and Agent agreement. This covers enrollment and credential handoff. It does not claim telemetry delivery.

## Ownership

- Core authenticates the human, authorizes the Node, and requests an invitation.
- Fleet owns invitation, Agent enrollment/recovery, credentials/revocation, and OpAMP session authorization.
- Agent owns stable installation identity, protected local state, and provisioning Fleet's OpAMP credential to official Supervisor.
- Agent installation ID (agent_id) and Supervisor protocol ID (instance_uid) are distinct. Host identity is evidence, not authentication.

## Verified repository snapshot — 2026-11-23

| Repository | Branch / implementation commit | Current implementation |
|---|---|---|
| Fleet Manager | feat/agent-registration — 12fa4f981f15e2dd3444b08d3833e617c6f13392 | Invitation, exchange, recovery, status, cancellation, version-fenced two-phase gate compensation, per-Agent credential revocation, and secret-free installation lookup routes are mounted. Enrollment and OpAMP default disabled. Mongo idempotency, encrypted result escrow, credential verifier, and UID binding exist. Credential rotation is not implemented. Routes remain provisional. |
| Core backend | feat/fleet-core-integration-design — 0c997fb77797a5a6d331fb95c21ae9fbecc69844 | Human-authorized invitation/sync controllers and mutual-TLS Fleet client exist. Invitation, operation-status, installation lookup, cancellation, and two-phase gate adapter paths match Fleet source. Core still authorizes using its existing human and Node rules; Core-backed enrollment remains the default. |
| Telemetry Agent | chore/v0.1.4a-next — db6559e80d8c00461c87a94f40150c26a58b2858 | Existing Core enrollment remains the default. An explicit Fleet-origin CLI path now supports invitation exchange/recovery, protected credential storage, and optional Supervisor provisioning. It is opt-in and does not replace the Core path. |

The reports reconcile: Fleet routes exist in source but enrollment and OpAMP remain disabled by default. Core and Fleet now have source-level parity for invitation cancellation and versioned two-phase gate compensation. Agent has an explicit Fleet-origin CLI path; the existing Core enrollment remains the default. Local real-service UAT verified invitation, exchange, Core binding sync, Supervisor session, and OTLP metrics. The 2026-11-23 follow-up verified cancellation, gate compensation, and revocation with Fleet HTTP/Mongo tests. Current Prometheus UAT metrics have no agent_id or installation_id labels; the Gateway has no logs or traces pipeline. These results do not approve the proposed wire contract.

## Proposed flow

1. Human → Core: authenticate, authorize Node, request invitation.
2. Core → Fleet over mutual TLS: create using stable operation and idempotency identities.
3. Agent → Fleet over HTTPS: exchange with persisted agent_id, immutable attempt ID, and independent recovery proof.
4. Fleet atomically consumes invitation, binds installation to Node, and returns one OpAMP-only credential.
5. Agent protects credential and provisions it to Supervisor. Supervisor → Fleet over WSS: authenticate first, then Fleet pins actual instance_uid to agent_id.
6. Verify Supervisor session, Collector readiness, and OTLP receipt separately. Registration alone proves none of these.

## Proposed minimum wire contract

See [OpenAPI proposal](../openapi/agent-registration-v1-proposal.openapi.yaml). Paths are relative to Fleet service root.

| Operation | Proposed path | Caller |
|---|---|---|
| Create/replay invitation | POST /internal/v1/enrollment/invitations | Core mTLS |
| Read operation | GET /internal/v1/enrollment/operations/{operation_id} | Core mTLS; secret-free |
| Cancel pending invitation | POST /internal/v1/enrollment/invitations/{invitation_id}/cancel | Core mTLS |
| Close Node gate | POST /internal/v1/nodes/{node_id}/enrollment-gate/close | Core mTLS; returns version and prior binding/state |
| Complete or compensate gate | PUT /internal/v1/nodes/{node_id}/enrollment-gate | Core mTLS; exact close operation and version required |
| Exchange | POST /v1/enrollment/exchanges | Agent HTTPS |
| Recover uncertain exchange | POST /v1/enrollment/recoveries | Agent HTTPS |
| Read installation | GET /internal/v1/installations/{agent_id} | Core mTLS; secret-free |
| Read active installation | GET /internal/v1/nodes/{node_id}/active-installation | Core mTLS; secret-free |
| Revoke credentials | POST /internal/v1/agents/{agent_id}/revoke | Core mTLS; idempotently fences active credentials |

### Wire, identity and security rules

- JSON schema_version is 1. X-Request-Id is correlation; operation_id is Core business-operation identity; Idempotency-Key is mutation replay identity.
- Invitation body: operation_id, node_id, requested_ttl_seconds. Require Idempotency-Key header, scoped to authenticated Core identity and normalized body. Core authorizes human/Node; Fleet verifies configured Core client certificate. Forwarded actor ID is audit attribution only.
- Proposed TTL preserves current Fleet defaults: 30 minutes, maximum 24 hours. Same key+body replays same pending invitation secret; changed body conflicts. Operation status never returns a secret.
- Exchange body: enrollment_attempt_id, agent_id, invitation_token, recovery_proof. Agent durably stores the immutable attempt and proof before first send and retries exact values. Agent does not supply instance_uid.
- Proposed recovery window 24 hours, matching Fleet default. Fleet retains proof hash and authenticated ciphertext of original result. Recovery requires attempt ID, agent_id, same proof and never issues another credential. Expiry requires new Core-authorized invitation.
- Result: agent_id, node_id, credential_id, credential, credential_scope, opamp_endpoint, credential_expires_at. Proposed scope opamp.connect for that installation/WSS endpoint only; no Core, Agent REST, or OTLP access. Propose null expiry until revoke for v1; Fleet currently has no expiry, and rotation remains OPEN.
- Secret responses use Cache-Control: no-store. Never log invitation, proof, credential, or raw identity evidence.
- Fleet authenticates credential before binding Supervisor's actual instance_uid to agent_id. Reconnect requires same UID. Changed/already-owned UID is rejected. No automatic rebind; require separate authorization, audit, expected-current-UID.
- Proposed stable errors: INVALID_REQUEST (400), INVITATION_REJECTED (401 for invalid/expired/cancelled), IDEMPOTENCY_CONFLICT or ENROLLMENT_CONFLICT (409), RECOVERY_EXPIRED (410), RATE_LIMITED (429), TEMPORARY_UNAVAILABLE (503). Exact mapping OPEN; use shared Problem response.
- Retry network uncertainty, 429, and 5xx with bounded backoff and exact same operation/attempt. Do not follow redirects. Core replays same POST/key after response loss; Agent uses recovery after uncertain exchange, never a new attempt.

### Gate semantics

OPEN permits authorized invitation/exchange. CLOSED rejects new invitations and unconsumed exchanges. Gate change serializes with exchange: if exchange wins, close reports binding; if close wins, exchange fails. Closing does not revoke existing credentials or end OpAMP sessions. Same operation_id+state replays; changed state conflicts; next transition uses new operation_id. Proposed unseen-gate default OPEN matches Fleet today; Core closes before Node retirement and reopens after compensated failure. Fail-closed default is OPEN for review.

## Compatibility matrix

| Boundary | Current state | Required change |
|---|---|---|
| Core → Fleet invitation | Current Core and Fleet source use /internal/v1/enrollment/invitations, requested_ttl_seconds, X-ARIMMS-Operation-Id and Idempotency-Key. Fleet also requires the forwarded actor-attribution header and authenticates the Core certificate. Real local mTLS invitation creation and exact replay passed. | Keep the wire details PROPOSED until all owners approve the same OpenAPI SHA; preserve stable operation/key on retries. |
| Core → Fleet status/gate/cancel | Operation status and installation lookup paths are mounted in Fleet and used by Core. Core lifecycle adapters and Fleet use matching cancellation and two-phase close/complete paths in source. Fleet exposes per-Agent credential revocation, but Core has no Fleet revocation orchestration yet. | Agree on cancellation/gate semantics and add authorized Core revocation orchestration if revocation is accepted into v1; keep status and installation responses secret-free. |
| Agent → Fleet | Existing Core enrollment remains default; explicit Fleet-origin enrollment/recovery and protected credential storage are implemented on the Agent branch. Local real-service registration and Supervisor provisioning passed. | Keep Fleet mode explicit until contract agreement and production rollout; preserve Core mode during migration. |
| Fleet → Supervisor | Verifier, callback auth, Mongo UID binding, Application wiring exist; OPAMP_ENABLED defaults false. Exchange returns opamp.connect scope and null expiry. Fleet revoke is mounted and tested; rotation is not implemented or agreed. | Keep disabled until security and real Agent/Supervisor integration pass. |
| Registration → telemetry | Separate Agent Gateway config; enrollment has no OTLP receipt. | Verify backend delivery as a separate UAT gate. |

## Repository implementation and tests

1. **Shared:** approve choices, then adopt OpenAPI and update C01/C04.
2. **Fleet:** source implements mTLS-protected routes, invitation replay and encrypted recovery, cancellation, versioned gate compensation, and per-Agent revoke. Mongo/HTTP tests cover cancellation/replay, stale gate completion, binding snapshot, and revoke fencing; shared wire details remain proposed. Credential rotation is unimplemented pending an agreed recovery and overlap policy.
3. **Core:** retain human/Node authorization; align adapter/Problem mapping; same operation/key; test mTLS, uncertain response, no-store, gate compensation and denied auth.
4. **Agent:** add explicit Fleet target without silently replacing Core enrollment; persist attempt/token/proof, protect credential, provision only to Supervisor. Test restart recovery, retained/fresh Supervisor state, revoke and redaction.
5. **Integration/UAT:** Core + Mongo Fleet + production Agent + pinned Supervisor v0.160.0; verify exchange/recovery, OpAMP auth/UID, shutdown/reconnect, then separately verify OTLP at backend.

## Cross-repo decisions OPEN

1. Route roots, Core mTLS identity, actor audit header.
2. Idempotency scope/retention and token replay until invitation terminal state.
3. Agent proof persistence and 24-hour recovery.
4. OpAMP-only scope, null-expiry v1, and safe credential rotation/recovery delivery (not implemented).
5. Unseen-gate default and Core close/reopen ordering. The current Fleet implementation assumes OPEN for an unseen Node and version-fences compensation.
6. Problem-code mapping, Core authorization/orchestration for Fleet revocation, and whether revocation belongs in v1. Rotation and replacement-credential delivery remain unimplemented until recovery, overlap, and lost-response behavior are agreed.
7. Coexistence/migration from Core enrollment; no shared migration contract exists.
8. Whether OTLP is acceptance after registration; keep Collector readiness and OTLP receipt as separate evidence. Per-Agent resource attribution, and logs/traces storage/export paths, are not verified by the current UAT.
