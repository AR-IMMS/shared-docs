# Flow 02 — Register and Manage Collector Agent

> **Document type:** Shared business-flow baseline<br>
> **Status:** Draft for team review · 2026-10-30<br>
> **Ownership:** Cross-team business specification (`docs/shared`)<br>
> **Related contract:** `contracts/openapi/agent-enrollment.openapi.yaml` (OpenAPI v0.1.0)

## 1. Objective and boundaries

Enable authorized personnel to enroll an AR-IMMS Collector Agent running on a computer into an **existing Node**, establish a trusted Agent–Node association and later manage its lifecycle. The same registration process applies to physical hosts and virtual machines. A Node represents an asset already managed by Asset & Topology; registering an Agent does not create or place a Node.

**In scope (Flow 2):** bootstrap token issuance, Agent enrollment, binding/credential establishment, management visibility, unlink/revocation and local uninstall as lifecycle scenarios.

**Not part of registration success:** starting or configuring the OTel Collector, telemetry ingestion, exporter freshness, discovery of workloads, AI investigation, maintenance or automatic Host–VM relationship mapping. These are separate flows or later integrations.

**Implementation phasing:** The current server increment covers **bootstrap issuance and enrollment** first. Unlink, uninstall reporting, heartbeat and configuration sync are business-level follow-ons; their Agent-facing API contracts are **not yet finalized**.

## 2. Actors and preconditions

| Actor / system         | Role                                                                                                 |
| ---------------------- | ---------------------------------------------------------------------------------------------------- |
| IT Admin               | Select Node; issue bootstrap token; inspect or revoke Agent binding.                                 |
| Maintenance Technician | Install/configure Agent and initiate enrollment; uninstall locally when required.                    |
| Collector Agent        | Maintains installation identity; submits enrollment and retains credential securely.                 |
| Core Backend           | Authorizes Admin, checks Node eligibility, issues tokens, validates enrollment and manages bindings. |
| Asset & Topology       | Owns Node identity, lifecycle and placement.                                                         |

**Registration preconditions**

1. The target Node exists in Asset & Topology, is not retired and satisfies the current eligibility rule (baseline: assigned to a location).
2. The requesting Admin has the required IAM permission.
3. The Node has no active Agent binding, under the baseline **one active Agent installation per Node**.
4. The Technician has an installed Agent and a bootstrap token obtained through an authorized operational channel.

**Unlink precondition:** active binding exists and Admin is authorized.<br>
**Uninstall precondition:** Agent is installed locally; network connectivity is **not guaranteed**.

## 3. Main flow — Bootstrap and enrollment

```mermaid
sequenceDiagram
    autonumber
    actor Admin
    participant Core as Core Backend
    participant Asset as Asset & Topology
    participant DB as MongoDB
    actor Technician
    participant Agent as Collector Agent
    participant Local as Local secure storage

    Admin->>Core: Request bootstrap token for selected Node
    Core->>Asset: Verify Node eligibility
    Asset-->>Core: Eligible
    Core->>DB: Store token verifier and expiry (10 min)
    Core-->>Admin: Return bootstrap token once
    Admin-->>Technician: Deliver token using authorized channel
    Technician->>Agent: Initiate enrollment with token
    Agent->>Local: Reuse durable installation ID
    Agent->>Local: Persist attempt ID + request state
    Agent->>Core: POST /api/v1/agent-enrollments
    Core->>DB: Validate, consume token and create binding atomically
    DB-->>Core: Enrollment committed
    alt Response received
        Core-->>Agent: 201 Binding + credential + Core control URL
        Agent->>Local: Securely persist credential and binding
    else Response lost
        Agent->>Core: Retry same attempt ID, token and immutable request
        Core-->>Agent: 200 Original result and credential within 24 hours
        Agent->>Local: Securely persist credential and binding
    end
    Note over Agent,Core: Registered; Collector activation happens later
```

**Business steps**

1. Admin selects a Node, verifies identifying details and requests a bootstrap token.
2. Core authorizes the Admin, checks Node eligibility and issues a high-entropy token **bound to that Node**, valid for **10 minutes**, intended for **one successful enrollment**; plaintext is displayed once.
3. Technician supplies the token to the installed Agent.
4. Agent reuses its persistent installation ID, creates a unique enrollment attempt ID, and sends its version and execution-environment metadata to Core.
5. Core validates token, Node eligibility, installation and binding uniqueness, then atomically consumes the token and records the installation/binding/credential issuance.
6. Core returns the registered Node reference, binding and Agent credential. Agent persists the result securely.
7. The Agent is now **registered**, but its telemetry configuration and Collector activation are handled separately.

### Shared enrollment schema (business view)

```json
{
  "bootstrap_token": "<secret>",
  "enrollment_attempt_id": "d99c38f2-7668-4814-9e70-6416014f83fe",
  "installation": { "id": "5baf0cba-cf5a-4d39-a3d4-e76263b70810" },
  "agent": { "version": "0.1.4" },
  "environment": {
    "os": "linux",
    "architecture": "amd64",
    "hostname": "node-01"
  },
  "metadata": {}
}
```

`metadata` is optional, extensible and untrusted. Host and VM installations use the **same schema**; host names, MAC addresses, host fingerprints and custom metadata must **not** authorize Node ownership. Core resolves Node from the token, not from Agent-supplied `node_id`.

## 4. Exceptions and alternate paths

| ID    | Situation                                                                     | Expected behavior                                                                                                                         |
| ----- | ----------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| EX-01 | Node absent, retired, ineligible or already actively bound                    | Do not issue an eligible token or create another active binding; return an appropriate business error.                                    |
| EX-02 | Bootstrap token invalid, expired or previously consumed for another operation | Reject enrollment; no new binding; Technician requests a new token as appropriate.                                                        |
| EX-03 | Two installations try to bind to the same Node                                | At most one succeeds; the other receives conflict.                                                                                        |
| EX-04 | One installation tries to hold multiple active Node bindings                  | Reject the conflicting enrollment.                                                                                                        |
| EX-05 | Network times out after Core commits enrollment                               | Agent retries the **same attempt ID, bootstrap token and unchanged request**. Core returns the original result for 24 hours; after that it returns `410 ENROLLMENT_RECOVERY_EXPIRED` without creating another binding. |
| EX-06 | Core temporary outage / optional deployment-edge rate limit                    | For an uncertain enrollment outcome, retry the same attempt ID, bootstrap token and immutable request using bounded client backoff. A deployment edge may return 429; respect `Retry-After` when present. Core does not implement rate limiting for these endpoints. |
| EX-07 | Local credential persistence fails                                            | Agent retains recoverable attempt state; do not claim registration has completed locally.                                                 |
| EX-08 | Agent metadata is incomplete                                                  | Required schema fields are validated; optional data may be omitted; metadata does not determine authorization.                            |

## 5. Lifecycle follow-on scenarios

### 5.1 Unlink / revoke (I4)

1. Admin with `agent:manage` requests `POST /api/v1/agent-bindings/{bindingId}/revoke`.
2. Core revokes the association and its active credentials atomically, records one audit fact, and retains installation and binding history. First and repeated calls return `204 No Content`.
3. Agent may be offline. It learns about revocation on a later authenticated interaction or another defined mechanism; **immediate local shutdown is not guaranteed**.
4. An unlinked Node retains its Asset & Topology placement; unlink is **not** asset `unassigned` or `retired`.
5. Re-enrollment is a new authorized operation with a new bootstrap token and attempt ID. It may reuse the stable installation ID, updates Agent version/environment/metadata, and creates a new binding without restoring the revoked binding.

### 5.2 Uninstall (subsequent increment)

1. Technician invokes local uninstall.
2. Agent attempts a final notification when possible, then removes Agent-owned components and protected local secrets under OS policy.
3. Offline uninstall must be possible; final notification is **best-effort**, not a prerequisite for removing software.
4. Core should not assume a missing uninstall report proves the Agent still runs. Any needed server-side revoke/cleanup follows the management policy.

## 6. Business rules / invariants

| Rule  | Statement                                                                                                  |
| ----- | ---------------------------------------------------------------------------------------------------------- |
| BR-01 | Node is created and positioned by Asset & Topology before enrollment.                                      |
| BR-02 | Only an authorized Admin may issue a Node-bound bootstrap token.                                           |
| BR-03 | Bootstrap token is high entropy, single-use, valid for 10 minutes and never exposed again after issuance.  |
| BR-04 | One Node and one installation each have at most one **active** binding.                                    |
| BR-05 | `installation.id` persists across restarts/upgrades and re-enrollment; it is not, by itself, proof of authorization. |
| BR-06 | Enrollment attempt ID is stable across retries of the same operation.                                      |
| BR-07 | Node identity in enrollment is resolved from token; host/VM metadata is informational.                     |
| BR-08 | Credential authorization is subject to an active registration binding; revoked registrations are rejected. |
| BR-09 | Registration, Agent connectivity and telemetry health are distinct states.                                 |
| BR-10 | Unlink does not alter the Node's physical/logical placement.                                               |

## 7. Acceptance scenarios

- **AC-01:** Authorized Admin can issue one 10-minute Node-bound bootstrap token for an eligible Node; it is returned in plaintext only once.
- **AC-02:** Agent submits the documented v1 payload and receives a binding referencing the correct Node and an Agent credential.
- **AC-03:** Expired, invalid or consumed token cannot establish another binding, even before database TTL cleanup runs.
- **AC-04:** Concurrent enrollment cannot create multiple active bindings for either Node or installation.
- **AC-05:** Retried requests preserve installation ID and attempt ID; duplicate committed requests are recognized without duplicating business records.
- **AC-05a:** A matching retry proves possession of the original bootstrap token and recovers the same credential for 24 hours; the attempt ID alone never authorizes recovery.
- **AC-06:** Agent credential is stored securely and never logged or persisted as plaintext operational state.
- **AC-07:** Enrollment success does not automatically mark the Collector operational or telemetry healthy.
- **AC-08:** Authorized unlink revokes permission while preserving the Node's placement and binding history.
- **AC-09:** Core accepts a credential only while both credential and binding are active; Agent credentials never authorize Human/Admin APIs.
- **AC-10:** Revoking a binding invalidates its active credentials, preserves history, and appends one audit fact; repeated revocation returns 204 without another fact.
- **AC-11:** Matching I3 recovery after revocation returns `410 ENROLLMENT_RECOVERY_REVOKED` and never reveals the credential.
- **AC-12:** Re-enrollment with a new token and attempt can reuse the installation ID, updates its current Agent metadata, and creates a distinct active binding.

## 8. Open decisions / change control

| Decision                                                              | Status                                                                          |
| --------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| Reissue policy when an unconsumed bootstrap token exists              | **Confirmed for I1:** new issuance atomically invalidates the previous unused token for that Node. |
| Node eligibility for bootstrap issuance and enrollment                 | **Confirmed for I1:** Node exists, lifecycle is active, placement is active, and no active Agent binding exists; QR mapping is not required. |
| Credential expiration, renewal and rotation                           | **Out of I4**; credentials have no expiry in this increment.                   |
| Lost-response credential recovery/security                            | **Approved:** AES-256-GCM encrypted result escrow for 24 hours; recovery requires the same attempt ID, original bootstrap token and immutable request fingerprint. |
| Agent binding revocation API                                           | **Implemented in I4 and published in OpenAPI:** `POST /api/v1/agent-bindings/{bindingId}/revoke`, `agent:manage`, idempotent 204. |
| Control Plane, heartbeat, uninstall-report and telemetry API shapes   | **Outside current OpenAPI**; design and implement in later increments.          |
| Administrator/Technician notification on registration failure         | **Deferred**; logging/audit first, do not promise proactive notification.       |

**Documentation policy:** This file describes shared business behavior, not MongoDB schemas, NestJS classes or internal server implementation. Technical design belongs in `docs/local` of the Core Backend repository.

## 9. Agent integration readiness boundary

Core currently publishes bootstrap-token issuance, initial/recovered enrollment, and Admin binding revocation. The `agentBearer` scheme is reserved for explicitly protected Agent-facing Core APIs; no production I1-I4 operation currently accepts an Agent credential. The enrollment response's Control Plane URL is configuration data only and does not make Control Plane operations available. Agent operational readiness therefore remains separate from Backend Enrollment readiness.
