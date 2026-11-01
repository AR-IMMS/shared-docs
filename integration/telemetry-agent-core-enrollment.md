# Telemetry Agent ↔ Core Enrollment Integration

## Readiness boundary

Core implements bootstrap-token issuance, Agent enrollment and recovery, and Admin binding revocation. The `agentBearer` scheme is reserved for future Agent-authenticated Core operations; no production operation in the current contract accepts an Agent credential.

The enrollment response includes a configured Control Plane URL and a 30-second polling interval. These are configuration values only. Core does not implement the referenced Control Plane, heartbeat, telemetry, configuration synchronization, Collector activation, or uninstall APIs.

**Backend Enrollment Ready** means the enrollment lifecycle and its published contracts are usable. **Agent Operational Ready** requires authenticated Agent operations and Control Plane functionality and is not met by this increment.

## API inventory

All Core endpoints use HTTPS. Responses containing bootstrap tokens or Agent credentials include `Cache-Control: no-store`.

| Method and path | Caller authentication | Result |
| --- | --- | --- |
| `POST /api/v1/nodes/{nodeId}/agent-bootstrap-tokens` | Human IAM; requires `agent:manage` | `201` with a Node-bound bootstrap token shown once |
| `POST /api/v1/agent-enrollments` | Bootstrap token in request body; no bearer header | `201` for a new enrollment; `200` for matching recovery |
| `POST /api/v1/agent-bindings/{bindingId}/revoke` | Human IAM; requires `agent:manage` | `204` for first or repeated revocation |

The opaque Agent credential is for explicitly Agent-authenticated Core APIs only. No such production API is currently published. It is not a Human/Admin credential or an OTLP Gateway credential.

When a future route opts into Agent authentication, Core hashes the bearer secret and resolves trusted `installationId`, `bindingId`, and `nodeId` from persisted records. Authorization requires an active credential, active binding, and consistent installation association. An installation ID by itself never grants access.

Bootstrap issuance requires an existing Node with active lifecycle, active placement, and no active Agent binding. Core resolves the Node from the bootstrap token; Agent-provided metadata does not select or authorize a Node.

## Enrollment sequence

```mermaid
sequenceDiagram
    autonumber
    actor Admin
    participant Core as Core Backend
    participant Asset as Asset & Topology
    participant DB as MongoDB replica set
    actor Agent as Telemetry Agent
    participant Store as Agent secure local storage

    Admin->>Core: POST bootstrap token (Human IAM, agent:manage)
    Core->>Asset: Check Node eligibility
    Asset-->>Core: Active Node, active placement, no active Agent binding
    Core->>DB: Store token hash and 10-minute expiry; append audit fact
    Core-->>Admin: 201 token (plaintext once, no-store)
    Admin-->>Agent: Deliver token through authorized channel
    Agent->>Store: Reuse installation ID; persist attempt ID and immutable request
    Agent->>Core: POST /api/v1/agent-enrollments
    Core->>Asset: Recheck Node eligibility
    Core->>DB: Transaction: consume token, update/create installation, binding, credential hash, attempt, escrow, audit fact
    DB-->>Core: Commit
    Core-->>Agent: 201 enrollment result and bearer credential
    Agent->>Store: Persist result and credential securely
    Note over Agent,Core: The returned Control Plane URL is not an implemented API
```

If the `201` response is lost, the Agent retries the exact persisted request with the same attempt ID and bootstrap token. A matching retry inside 24 hours returns the original enrollment result with `200`. Recovery is rejected if the binding or credential has been revoked.

## Request and response examples

Issue a token using an authorized human IAM bearer:

```http
POST /api/v1/nodes/7b120dad-8bf3-4bac-bc0c-bc71f95d4a21/agent-bootstrap-tokens HTTP/1.1
Authorization: Bearer <human-access-token>
```

```json
{
  "data": {
    "token_id": "bt_8c08c480-492d-4b7c-b6ab-3871949b0afe",
    "bootstrap_token": "arimms_bt_<secret>",
    "node_id": "7b120dad-8bf3-4bac-bc0c-bc71f95d4a21",
    "expires_at": "2026-11-01T12:10:00.000Z"
  },
  "meta": { "request_id": "<request-id>" }
}
```

Enroll without an Agent bearer credential:

```http
POST /api/v1/agent-enrollments HTTP/1.1
Content-Type: application/json
```

```json
{
  "bootstrap_token": "arimms_bt_<secret>",
  "enrollment_attempt_id": "d99c38f2-7668-4814-9e70-6416014f83fe",
  "installation": { "id": "5baf0cba-cf5a-4d39-a3d4-e76263b70810" },
  "agent": { "version": "0.1.4" },
  "environment": { "os": "linux", "architecture": "amd64", "hostname": "node-01" },
  "metadata": { "deployment_kind": "vm" }
}
```

Both success statuses use the same response envelope and result shape; only the status distinguishes new commit from recovery:

```json
{
  "data": {
    "enrollment_id": "e4fdf407-96b1-4ff1-b659-c9cf4470b5f9",
    "installation_id": "5baf0cba-cf5a-4d39-a3d4-e76263b70810",
    "binding": {
      "id": "e9a95012-1c46-4e85-97a0-a6b909b5c935",
      "node_id": "7b120dad-8bf3-4bac-bc0c-bc71f95d4a21",
      "status": "active"
    },
    "credential": { "type": "bearer", "access_token": "<secret>" },
    "control_plane": {
      "base_url": "https://core.example.com/api/v1/agent-control",
      "poll_interval_seconds": 30
    }
  },
  "meta": { "request_id": "<request-id>" }
}
```

The `control_plane.base_url` example does not mean `/api/v1/agent-control` is available. Core returns the configured `AGENT_CONTROL_PLANE_BASE_URL` value unchanged.

Revoke a binding with an authorized human IAM bearer and `agent:manage`:

```http
POST /api/v1/agent-bindings/e9a95012-1c46-4e85-97a0-a6b909b5c935/revoke HTTP/1.1
Authorization: Bearer <human-access-token>
```

The first and repeated successful calls return `204 No Content`; no response body is sent.

## Installation and attempt identity

- `installation.id` is a stable UUID retained across Agent restart, upgrade, and authorized re-enrollment. It is identity data, not an authentication secret.
- `enrollment_attempt_id` is a UUID generated once per enrollment operation. Persist it and the immutable request before sending. Reuse both after timeout, restart, or an uncertain server outcome.
- A separate enrollment, including re-registration after revocation, uses a new attempt ID and a new bootstrap token. It may reuse the stable installation ID.
- On successful re-registration, Core updates the existing installation's Agent version, environment, and metadata, creates a new binding and credential, and retains the revoked binding history.

## Credential storage and authenticated requests

Core persists only SHA-256 hashes of bootstrap tokens and Agent credentials. Recovery results are encrypted with AES-256-GCM for 24 hours. Audit facts do not contain either plaintext secret.

The Agent must store its credential in OS-protected secret storage and send it only as `Authorization: Bearer <credential>` to a future endpoint explicitly documented as Agent-authenticated. Do not send it to human/Admin APIs or an OTLP Gateway. I4 defines no credential expiry or renewal endpoint; credential revocation follows binding revocation.

## Lost-response retry and recovery

The request fingerprint covers canonical enrollment fields and the bootstrap-token hash. The attempt ID alone does not authorize recovery. To recover, resend the original bootstrap token, attempt ID, and unchanged request.

- `201 Created`: a new enrollment committed. Persist the returned installation, binding, and credential.
- `200 OK`: a matching committed result was recovered. Persist the returned credential as for `201`.
- `409 ENROLLMENT_ATTEMPT_CONFLICT`: the attempt ID was reused with different request data. Do not alter and replay the attempt; restore the original request or begin a separately authorized operation.
- `410 ENROLLMENT_RECOVERY_EXPIRED`: the recovery escrow is absent or expired. Core retains the binding and credential hash and will not create a second enrollment for that attempt.
- `410 ENROLLMENT_RECOVERY_REVOKED`: the original binding or credential is revoked. Core will not return the escrowed credential.
- `503 ENROLLMENT_UNAVAILABLE` or a lost connection: the commit outcome may be uncertain. Retry the exact same request and attempt ID.

Use bounded client backoff for retryable network and service failures. No fixed retry delay is published by Core. A deployment edge may optionally return `429`; respect `Retry-After` when present. Core does not implement rate limiting for these operations.

## Error catalog and Agent/Admin actions

Responses use `application/problem+json`, the stable `code`, and `request_id` correlation. Do not retry unchanged requests for permanent client or business errors.

| Status and code | Meaning | Recommended action |
| --- | --- | --- |
| `400 VALIDATION_ERROR`, `400 BAD_REQUEST` | Invalid enrollment body or malformed path ID | Correct the request; do not retry unchanged. |
| `401 INVALID_BOOTSTRAP_TOKEN` | Token is unknown or invalid | Check secure token delivery; obtain a new token for a new operation. |
| `401 BOOTSTRAP_TOKEN_EXPIRED` | Bootstrap token is past its 10-minute validity | Obtain a new token and start a new attempt. |
| `401 BOOTSTRAP_TOKEN_CONSUMED` | Token was consumed by another operation | Do not start a different attempt with it; request a new token. |
| `401 UNAUTHORIZED` | Human/Admin authentication is absent or invalid | Refresh human IAM authentication. |
| `403 FORBIDDEN` | Human lacks `agent:manage` | Request the required permission. |
| `404 NOT_FOUND` | Node or binding does not exist | Verify the selected resource ID. |
| `409 NODE_NOT_ELIGIBLE`, `NODE_ALREADY_BOUND`, `INSTALLATION_ALREADY_BOUND` | Current Node or installation state conflicts with enrollment | Resolve the state in Core/Asset & Topology before starting a new authorized attempt. |
| `409 ENROLLMENT_ATTEMPT_CONFLICT` | Same attempt ID, changed request fingerprint | Preserve the original attempt request; do not mutate it. |
| `409 CONFLICT` | Bootstrap issuance conflict | Retry issuance as an Admin operation; a new issuance invalidates a prior unused token for the Node. |
| `410 ENROLLMENT_RECOVERY_EXPIRED` | 24-hour recovery escrow is no longer available | Start a new authorized enrollment with a new token and attempt if enrollment is still required. |
| `410 ENROLLMENT_RECOVERY_REVOKED` | Original enrollment was revoked | Do not use the old credential; re-enroll only through an authorized new operation. |
| `429 TOO_MANY_REQUESTS` | Optional deployment-edge response | Wait according to `Retry-After` if supplied, then retry within the client retry budget. |
| `500 INTERNAL_SERVER_ERROR` | Unexpected Core failure; may occur during bootstrap issuance | Do not expose details or reuse a possibly delivered secret. An Admin may issue a fresh token; issuance invalidates the previous unused token. |
| `503 ENROLLMENT_UNAVAILABLE` | Enrollment/recovery could not be completed or verified | Retry the exact same attempt and request because the commit outcome may be unknown. |
| `503 AGENT_REVOCATION_UNAVAILABLE` | Revocation transaction or audit write failed | Admin may retry the same idempotent revoke request. |
| `503 AGENT_AUTHENTICATION_UNAVAILABLE` | Future explicitly Agent-authenticated operation could not verify credentials | Retry that operation later; no such production operation currently exists. |

## Configuration and deployment prerequisites

- Serve the APIs over HTTPS. Human endpoints require functioning IAM authentication and `agent:manage` permission for bootstrap issuance and revocation.
- Configure `MONGODB_URI` for a replica set and `MONGODB_DB_NAME`. Enrollment and revocation use transactions; apply the registered MongoDB migrations before serving requests.
- Configure `AGENT_ENROLLMENT_RECOVERY_KEY` as a base64-encoded 32-byte key from a secret manager. All Core instances must share it, and it must remain stable until escrow records have expired.
- Configure `AGENT_CONTROL_PLANE_BASE_URL` as an HTTPS URL. Core returns it unchanged with `poll_interval_seconds: 30`. It is a future destination only; no operation is available there yet.
- Core redacts authorization headers from HTTP logs, excludes request bodies from its request serializer, and sets `Cache-Control: no-store` on responses containing secrets. Agents must also avoid logging tokens, credentials, or complete enrollment request bodies.
- MongoDB replica-set integration tests use the separate `MONGODB_TEST_URI` database whose name ends in `_test`.

## Integration checklist and known limitations

- [ ] Create an active, placed, unbound Node and confirm its identity with the Admin.
- [ ] Apply migrations and verify the Core HTTPS origin and required environment variables.
- [ ] Issue and deliver a bootstrap token through an authorized secret channel.
- [ ] Persist the installation ID, attempt ID, token, and immutable request securely before the first enrollment call.
- [ ] Verify both a new `201` enrollment and a simulated lost-response `200` recovery.
- [ ] Confirm the Agent stores the credential securely and does not use it for human or Gateway APIs.
- [ ] Verify Admin revocation returns `204` and that an authorized new enrollment can reuse the installation ID while preserving binding history.
- [ ] Treat the returned Control Plane URL as unavailable until a later contract publishes its operations.

There is no production Agent-authenticated operation to call after enrollment, no Control Plane, and no immediate notification to an offline Agent after revocation. Credentials have no expiry or renewal path in I1-I4. These limitations prevent an **Agent Operational Ready** verdict even when **Backend Enrollment Ready** is achieved.
