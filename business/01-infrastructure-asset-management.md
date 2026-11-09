# Business Flow 1 — Infrastructure Asset Management

**Status:** Revised business baseline — pending cross-domain confirmation for BR-14 and implementation verification  
**Domain:** Asset & Topology  
**Primary actor:** IT Admin

## 1. Purpose

Manage the identity, physical placement, and lifecycle of the infrastructure assets required by the AR-IMMS mini data center proof of concept. The flow establishes a reliable physical context that other flows can reference when registering agents, monitoring nodes, investigating incidents, and performing field maintenance.

This flow defines business behavior and ownership. It does not prescribe database tables, API paths, or a particular digital-twin technology.

## 2. Outcomes

- The seeded default Data Center anchors a usable location hierarchy: **Data Center → Site → Room → Rack**. IT Admins can retrieve the Data Center and manage Sites and Rooms beneath it.
- Each managed compute Node has a stable AR-IMMS identity and starts in lifecycle state `ACTIVE`, whether or not it has been placed in a Rack.
- Rack capacity and Node placement are represented consistently, including the Rack column/position and U position required by the PoC.
- Rack QR identification is unique and resolves to the correct Rack.
- Placement, marker, and lifecycle changes can be traced over time.
- Other domains can reference assets by their AR-IMMS IDs without taking ownership of asset identity or location.

## 3. Scope

### In scope

- Retrieve the seeded default Data Center; create and maintain Site, Room, Rack, and compute Node records beneath it. Flow 1 does not create or update the Data Center.
- Represent Rack capacity and its usable placement positions.
- Provision a Node in an unassigned state, assign it to a Rack position, move it, or unmap it.
- Manage system-generated QR mappings for Racks and Nodes and generate individual or bulk printable labels in both PDF and SVG.
- Track asset lifecycle separately from placement, Agent binding, and operational health.
- Retain a history of material asset, placement, and marker changes.
- Provide asset identity and confirmed placement for other AR-IMMS flows to reference.

### Out of scope

- Workload and Service registry, container/process inventory, and service dependency topology.
- Telemetry collection, health evaluation, alerting, and Incident lifecycle.
- Agent enrollment, credentials, heartbeat, configuration, unlink, and uninstall. These belong to Agent Management / Flow 2.
- Maintenance and Inspection Activity workflows, checklists, findings, corrective actions, and ticket handling. Asset maintenance-state coordination is proposed under BR-14 pending Flow 05 acceptance.
- PDU, power/cooling capacity, electrical topology, switches, routers, detailed network interfaces, storage arrays, and component-level inventory.
- Region/Availability Zone hierarchies, cloud accounts, tenants, billing, costs, warranty, procurement, and vendor lifecycle.
- A full 3D or physics model of a data center. Asset data may support AR presentation, while camera tracking and spatial alignment are handled by the AR client.

The supplied DC ERD is a reference for concepts, not a target schema. Only concepts with a confirmed Flow 1 use case are included here.

## 4. Participants

| Participant               | Responsibility in this flow                                                                                                                           |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| **IT Admin**              | Retrieves the default Data Center; creates and maintains Sites and Rooms beneath it and manages Racks and Assets, including placement and retirement. |
| **Technician**            | May physically print/apply a Rack label and scan it during field work. Asset record changes remain subject to the authorized management flow.         |
| **AR-IMMS Core**          | Validates business rules, maintains canonical asset/location records, and records changes.                                                            |
| **Maintenance**           | Owns Maintenance Activities; proposed coordination with Asset maintenance state is documented in BR-14, pending Flow 05 agreement.                    |
| **Ticket Management**     | Owns Ticket assignment; proposed assignee authorization for manual maintenance-state transitions is subject to BR-14 approval.                          |
| **Agent Management**      | Owns Agent identity and Agent–Node binding; the retirement check uses an agreed public-contract approach.                                             |
| **Other AR-IMMS domains** | Reference asset IDs and confirmed topology; they do not directly modify Asset & Topology data.                                                        |

The Telemetry Agent is not a participant in Flow 1. Its registration and lifecycle are handled by Flow 2.

## 5. Core terms

| Term                | Meaning in Flow 1                                                                                                                                                            |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Site**            | A top-level managed location in the PoC.                                                                                                                                     |
| **Data Center**     | The default facility root provisioned through seed/bootstrap; read-only in Flow 1.                                                                                           |
| **Room**            | A location within a Site.                                                                                                                                                    |
| **Rack**            | A managed rack within a Room, with capacity and placement positions.                                                                                                         |
| **Node**            | A managed compute asset. A laptop or PC may represent a Node in the PoC.                                                                                                     |
| **Placement**       | The current confirmed relationship recorded through Asset Location between a Node and a Rack position.                                                                       |
| **QR mapping**      | A system-generated opaque value mapped to one Rack or Node. It is not the canonical target ID or Agent identity. Values remain unique across active and historical mappings. |
| **Asset lifecycle** | One of `ACTIVE`, `MAINTENANCE`, or terminal `RETIRED`; separate from placement, Agent binding, and health.                                                                   |
| **Agent binding**   | The Agent–Node relationship owned by Agent Management.                                                                                                                       |

## 6. Business rules

### BR-01 — Authorized management

Only an authorized IT Admin may manage Sites, Rooms, Racks, placement, QR mappings, and normal Asset lifecycle operations. The proposed limited maintenance-state transition for an assigned Technician or Maintenance workflow is described separately in BR-14; it does not grant general asset-management permissions. The default Data Center is provisioned through seed/bootstrap and can be retrieved, but not created or updated through Flow 1. QR mapping changes request a new system-generated value; Admins do not supply QR values.

### BR-02 — Location hierarchy

The parent location must exist before a child location can be created. Sites belong to the seeded default Data Center, Rooms belong to Sites, and Racks belong to Rooms. The system must not accept a location assignment to a missing or retired parent. Reject a Room size or grid update if it would invalidate any current Rack placement or walkway cell. Grid cells are complete cells only; a remainder smaller than `grid_snap_mm` does not form a cell. An IT Admin may retire a Room only when no Rack is currently placed there; retirement preserves the Room record and map for history, removes it from the normal Room listing, and blocks edits and future Rack placements. Direct Room retrieval reports its `RETIRED` status.

### BR-03 — Stable asset identity

Each Node has an AR-IMMS identity that remains stable when its hostname, IP address, Rack placement, QR label, or Agent binding changes. Those values must not be treated as interchangeable identifiers.

### BR-04 — Rack capacity and placement

A Node may be assigned only to an existing Rack with available capacity and a free, valid position. The system rejects a placement that exceeds Rack capacity or conflicts with an occupied position. In AT-02, each Rack occupies one logical Room-map cell addressed by integer column and row indices; rotation is retained as location metadata and does not change cell occupancy. Physical Rack width and depth are not modeled in AT-02. The exact physical footprint rules for multi-U equipment are not defined by this business flow. An IT Admin may mark valid unoccupied grid cells as walkways; a Rack cannot be placed on a walkway. Room capacity is the number of complete grid cells minus walkway cells. The Room map returns walkways and Rack locations as separate data. Concurrent edits must not silently overwrite Room layout changes. A layout-changing request must be based on the latest confirmed layout revision; otherwise the system rejects it and the user reloads the layout before retrying. The existing HTTP precondition mechanism (`ETag` / `If-Match`, including source and destination revisions for a cross-Room move) is documented by the API contract rather than defined as a business concept here.

### BR-05 — Unassigned Nodes are valid

A Node may exist without a Rack placement. Creating or unmapping a Node does not delete or retire it.

### BR-06 — Placement changes are explicit

Assign, move, and unmap are separate asset-management operations. Moving a Node records the previous and new placement. Unmapping releases its Rack position and retains the Node record and its history.

### BR-07 — QR identification

Each Rack or Node has at most one active QR mapping. The system generates opaque values for both target types; Node creation and its initial mapping are atomic. QR values are unique across active and inactive history and cannot be reassigned. Admins request generated values rather than supplying QR text. Resolution accepts active mappings only and returns current data from the existing Asset or Rack query; mapping history does not snapshot target details.

### BR-08 — QR and spatial tracking are distinct

The QR is used to identify or look up a Rack or Node. It does not by itself define 3D geometry, a persistent spatial anchor, or precise navigation. AR alignment behavior belongs to the AR client.

### BR-09 — Lifecycle is separate from placement and health

Asset lifecycle states are `ACTIVE`, `MAINTENANCE`, and terminal `RETIRED`. New Assets start `ACTIVE` and unassigned. `ACTIVE` describes lifecycle only; it does not imply placement, Agent binding, or healthy operation. Placement is represented by Asset Location, and Agent binding is owned by Agent Management. Unmapping does not change lifecycle state.

### BR-10 — Retirement constraints

A Rack cannot be retired while Nodes remain placed in it. Retire an Asset only from `ACTIVE`, after maintenance has ended, it has been unmapped, and Agent Management confirms its binding has been removed. `RETIRED` cannot be reversed. Preserve the retired record and its references.

### BR-11 — Agent Management boundary

Flow 1 owns the Node identity and physical placement. Agent Management owns bootstrap tokens, Agent identity, Agent–Node binding, credentials, heartbeat/control state, unlink, and uninstall. Agent enrollment requires an eligible, assigned Node. Moving or unmapping a Node does not silently unlink or delete its Agent.

### BR-12 — Other domains reference assets

Monitoring, Incident, Maintenance, Inspection, and AR features reference the canonical Asset ID. They do not write directly into Asset & Topology storage or redefine the Node's physical placement.

### BR-13 — Change history

Material changes to locations, asset lifecycle, placement, or Rack/Node QR mapping record actor, source, and timestamp through the existing Audit mechanism. The history must be sufficient to determine what changed and the current confirmed placement; do not create a separate Audit store.

### BR-14 — Maintenance state coordination (PROPOSED; pending Flow 05)

**Decision status:** Proposed coordination rule, **not yet an accepted cross-domain contract**. Flow 05 must confirm the Maintenance Activity lifecycle, how overlapping activities are handled, and who may request manual transitions before implementation is treated as required.

Under the proposal, a Maintenance Activity entering `EXECUTE` requests that its linked Asset enter `MAINTENANCE`. When activities `RESUME` or `ABORT`, the Asset returns to `ACTIVE` only if no other Activity is executing for it. Maintenance requests state transitions through the Asset & Topology public boundary, with actor and source Activity recorded. The state change and Audit Fact must be consistent and retry-safe; failed coordination must remain recoverable. Ticket status transitions alone do not change Asset state.

The IT Admin remains authorized for manual lifecycle management. A proposed exception permits the *current Ticket assignee* to request only maintenance-state transitions for Assets linked to that Ticket, subject to authorization checks by Ticket Management and the guard against returning to `ACTIVE` while work is executing. This does not permit the assignee to create, move, unmap, retire or edit other Asset data. These details remain subject to Flow 05 confirmation.

## 7. Functional requirement table

| ID            | Business requirement                                                                                                               | Acceptance criteria                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **F1-REQ-01** | Asset and location management must be restricted to an authorized IT Admin, except for the proposed limited maintenance-state coordination in BR-14. | Unauthorized users cannot create or change locations, placements, QR mappings or ordinary asset lifecycle. Any proposed maintenance-state exception must pass its own authorization checks and cannot grant general asset management.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| **F1-REQ-02** | The system must maintain the location hierarchy **seeded Data Center → Site → Room → Rack**.                                       | The default Data Center is retrievable but read-only through Flow 1; a Room cannot be created without an existing Site; a Rack cannot be created without an existing active Room. Room map supports reserved walkway cells, rejects Rack placement on walkways, and reports capacity excluding walkways. Stale layout revisions conflict; Room size/grid changes that invalidate Rack or walkway cells are rejected. An IT Admin may retire a Room only when no Rack is currently placed there; retired Rooms are preserved for history, omitted from normal lists, and cannot be edited or receive Rack placements. |
| **F1-REQ-03** | Each managed Node must have a stable AR-IMMS identity.                                                                             | Changing a Node’s hostname, IP address, Rack placement, QR mapping, or Agent binding does not change its AR-IMMS identity.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| **F1-REQ-04** | A Node may exist without being assigned to a Rack.                                                                                 | A newly created or unmapped Node remains available in the registry as unassigned; it is not deleted or retired.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| **F1-REQ-05** | The system must represent Rack capacity and valid placement positions, including the required Column/Position and U position.      | A Rack occupies one logical Room-map cell addressed by integer column and row indices in AT-02; a Node placement that exceeds Rack capacity or conflicts with an occupied U position is rejected. The original placement remains unchanged when a move is rejected.                                                                                                                                                                                                                                                                                                                                                  |
| **F1-REQ-06** | Assigning, moving, and unmapping a Node must be separate operations.                                                               | Assign and move establish a valid Rack placement; unmap releases the position while retaining the Node record, identity, and history; if the Node has an active Agent binding, the binding and monitoring are preserved and the Node appears unassigned.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| **F1-REQ-07** | Racks and Nodes use system-generated QR mappings for lookup and field verification.                                                | QR values are globally unique across active and inactive history; each target has at most one active mapping. Node creation and its initial mapping are atomic. Resolution returns current target data through its Asset or Rack query; it does not return a frozen snapshot.                                                                                                                                                                                                                                                                                                                                        |
| **F1-REQ-08** | IT Admin must be able to generate individual or bulk printable Rack and Node QR labels in both PDF and SVG.                        | Each label contains the target's QR and readable identity/location information and can be physically applied and scanned.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| **F1-REQ-09** | Asset lifecycle is `ACTIVE`, `MAINTENANCE`, or terminal `RETIRED`, separate from placement, Agent binding, and operational health. | New Assets start `ACTIVE` and unassigned. Placement is recorded through Asset Location; Agent binding belongs to Agent Management. `ACTIVE` does not imply placement, Agent binding, or healthy operation.                                                                                                                                                                                                                                                                                                                                                                                                           |
| **F1-REQ-10** | An Asset or Rack may be retired only when its retirement preconditions are met.                                                    | Retire an Asset only from `ACTIVE`, after maintenance ends, it is unmapped, and Agent binding is removed. `RETIRED` cannot be reversed. A Rack with placed Nodes cannot be retired. Preserve history.                                                                                                                                                                                                                                                                                                                                                                                                                |
| **F1-REQ-11** | Material changes to locations, lifecycle, placement, and Rack/Node QR mappings must be traceable.                                  | The existing Audit mechanism records actor, source, and timestamp; no separate Audit store is introduced.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| **F1-REQ-12** | Other AR-IMMS flows must reference assets using the canonical Asset ID.                                                            | Other domains can retrieve confirmed asset identity and physical placement; they do not directly modify Asset & Topology records.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| **F1-REQ-13** | Asset and placement data must support the AR Digital Twin view.                                                                    | The view can represent Site/Room/Rack/Node placement and use Rack/Node QR mappings to resolve current target data. Asset & Topology does not own 3D geometry, camera tracking, or spatial anchors.                                                                                                                                                                                                                                                                                                                                                                                                                   |
| **F1-REQ-14** | Flow 1 must remain limited to confirmed asset-management needs.                                                                    | Workload/Service registry, telemetry, monitoring health, Incident lifecycle, Maintenance Activity workflows, and extended DCIM capabilities remain outside Flow 1. Asset maintenance-state coordination is proposed under BR-14 pending Flow 05 acceptance.                                                                                                                                                                                                                                                                                                                                                                                               |

> **Scope note:** F1-REQ-08 applies to **both Rack and Node labels**. Each label includes its QR value and readable target identity/location (when assigned). IP addresses and other operational details are not required. Multi-U equipment footprint rules remain deferred.

## 8. Main flow — provision and place a Node

1. Seed/bootstrap provisions the default Data Center; the IT Admin retrieves it.
2. The IT Admin creates a Site under the default Data Center if it does not already exist.
3. The IT Admin creates a Room under that Site and may mark valid, unoccupied grid cells as walkways.
4. The IT Admin creates a Rack under that Room, records its capacity, and selects its initial one-cell map location by integer column and row indices. Rack rotation is retained as location metadata; Node Column/Position and U placement labels remain separate from Room-map coordinates. The Room map shows walkways and Rack locations separately and reports the remaining cell capacity.
5. The IT Admin creates a Node record. AR-IMMS assigns its stable identity; the Asset starts `ACTIVE` and unassigned.
6. The IT Admin selects the Node, Rack, and target position.
7. AR-IMMS validates that the Node and Rack are eligible, the Rack has capacity, and the position is free.
8. If validation succeeds, AR-IMMS records the placement as an Asset Location and retains its history. The Node is now available as a physical asset reference to other flows.
9. Node creation generates its initial QR mapping atomically. The IT Admin may request a newly generated QR mapping for a Rack or replace/deactivate a Rack or Node mapping. The Admin can generate a single or bulk label for printing. A Technician may physically apply and scan the label to resolve the target's current details.
10. If Agent collection is required, the authorized user continues with Flow 2. Flow 1 does not perform Agent enrollment.

## 9. Alternative and lifecycle flows

### 9.1 Move a Node

1. The IT Admin selects an assigned Node and a destination Rack position.
2. AR-IMMS validates the destination Rack and position.
3. If valid, AR-IMMS replaces the confirmed placement, releases the previous position, and records the change.
4. The Node ID remains unchanged. The Agent binding, if any, remains owned by Agent Management and is not silently removed.

A Node in `ACTIVE` or `MAINTENANCE` may be moved. Placement changes do not change lifecycle state or Agent binding; a `RETIRED` Node cannot be placed or moved. If a field task has already verified the Node at its previous location, the field workflow should require a fresh location/QR confirmation before proceeding; the exact rule belongs to Flow 05.

If the destination Rack has insufficient capacity or the position is occupied, the move is rejected and the original placement remains unchanged.

If the requested Rack U is already the Node's current position, the move is a no-op and creates no new placement history or Audit Fact.

### 9.2 Unmap a Node

1. The IT Admin requests that a Node be removed from its Rack position.
2. AR-IMMS releases the position and records the unmapping.
3. The Node remains in the registry as unassigned. Its identity, history, and QR mapping records are retained. Agent unlink and retirement are separate operations.

An `ACTIVE` or `MAINTENANCE` Node may be unmapped. **If the Node is already enrolled, unmapping preserves its Agent binding and monitoring**; the Node appears as **unassigned** in asset/location views. Agent connectivity and telemetry coverage continue to be evaluated independently of placement. Unmapping does not end a Maintenance Activity or change lifecycle state. A `RETIRED` Node cannot be unmapped because retirement requires it to be unassigned. Enrollment of an unassigned Node remains disallowed until it is placed again (Flow 02).

### 9.3 Retire an asset

1. The IT Admin requests retirement of a Node or Rack.
2. For a Rack, AR-IMMS verifies that it contains no placed Nodes.
3. For an Asset, retirement is allowed only from `ACTIVE`, after maintenance has ended, the Asset is unmapped, and Agent Management confirms its binding has been removed.
4. If the preconditions pass, AR-IMMS marks the Asset `RETIRED` permanently and preserves its history and references. Otherwise, the request is rejected with the unmet precondition.

### 9.4 Manage QR mappings

The IT Admin may request a generated QR mapping for a Rack or Node, replace its active mapping with a new generated value, or deactivate it. A QR lookup resolves only active mappings and reads the target's current data through its existing Asset or Rack query. Mapping history retains prior values and target references without storing snapshots. A QR change does not change a Rack ID or Node ID. An inactive or historical QR value cannot be reassigned.

### 9.5 Retire a Room

1. The IT Admin requests Room retirement.
2. AR-IMMS checks that no Rack is currently placed in the Room and atomically marks the Room `RETIRED` with an Audit Fact.
3. If a Rack is still placed there or a concurrent layout change wins, AR-IMMS rejects the request and leaves the Room active.
4. A retired Room is omitted from normal Room listings, remains directly readable with its map and status for history, and cannot be edited or receive Rack placements. Repeating retirement is a no-op.

## 10. Failure and exception handling

| Condition                                                                     | Expected behavior                                                                                     |
| ----------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| Parent Site or Room does not exist                                            | Reject child location creation.                                                                       |
| Rack does not exist or is not eligible                                        | Reject Node placement.                                                                                |
| Rack capacity is exceeded or target position is occupied                      | Reject placement/move; preserve the prior placement.                                                  |
| QR value already exists in active or inactive history                         | Reject the duplicate value; historical QR values cannot be reused.                                    |
| Rack still contains placed Nodes                                              | Reject Rack retirement.                                                                               |
| Asset is in maintenance, is placed, or still has an Agent binding             | Reject Asset retirement until maintenance ends, it is unmapped, and Agent Management confirms unlink. |
| Room size/grid update would invalidate a Rack placement                       | Reject the proposed Room update and preserve the existing Room and Rack layout.                       |
| Walkway cell is outside the full-cell grid, duplicated, or occupied by a Rack | Reject the complete walkway update; preserve the Room map.                                            |
| Rack target cell is outside the grid, occupied, or marked as walkway          | Reject the Rack create/move; preserve existing Rack locations and history.                            |
| Room layout revision is stale                                                 | Return a conflict and require the client to reload the Room map before retrying.                      |
| Room retirement is requested while a Rack is placed there                     | Return a conflict; keep the Room active and preserve all Room and Rack data.                          |
| Edit or Rack placement targets a retired Room                                 | Return a conflict; preserve its retired status and history.                                           |
| IT Admin unmaps a Node                                                        | Release the position; display the Node as unassigned. Preserve existing Agent binding and monitoring; do not delete, retire or unlink it. |

## 11. Resulting system responsibilities

- **Asset & Topology** is the source of truth for Site/Room/Rack/Node identity, confirmed physical placement, Rack/Node QR mappings, and asset lifecycle.
- **Agent Management** is the source of truth for Agent identity and Agent–Node binding. A Flow-level application coordinator uses public contracts across the two domains: initial enrollment requires a Node that is `ACTIVE` and placed, whereas subsequent unmapping **does not revoke an existing binding**. Retirement requires a confirmed unbound result and must fail closed when binding status is unavailable. Protection against concurrent binding and retirement remains a cross-domain implementation/verification task (AT-10).
- **Monitoring/Observability** is the source of truth for telemetry and observed runtime/service behavior.
- **Maintenance** owns Maintenance Activities. Asset-state coordination and **Ticket Management** assignee-based authorization are **proposed in BR-14**, pending Flow 05 acceptance and agreement on a public contract.
- Other domains use the public contracts of the domains that own the requested data.

## 12. Digital Twin and AR usage

Asset & Topology supplies the authoritative Site/Room/Rack/Node identity and confirmed physical placement used by the AR Digital Twin view. The view may represent the Rack and its placed Nodes and use the Rack QR to verify the physical Rack. Asset & Topology does not own 3D geometry, camera tracking, spatial anchors, telemetry, or live health. Those concerns remain with the AR client and their owning domains.

## 13. Deferred decisions

These details are intentionally not invented by Flow 1 and should be decided only when a concrete use case requires them:

- Exact asset metadata fields beyond the identity, hierarchy, capacity, and placement information required by the flow.
- The physical footprint rules for Nodes that occupy multiple U positions.
- Whether any additional asset classes are needed.
- Label fields beyond the Rack/Node QR and the readable target identity/location needed for field verification.
- Any additional topology such as power, network, cooling, cloud, or Workload/Service relationships.
- Flow 05 approval of BR-14 maintenance-state coordination, Ticket-assignee authorization, and field re-verification after location changes.
- Verification of cross-domain enrollment/retirement concurrency guard (AT-10).
