# Business Flow 1 — Infrastructure Asset Management

**Status:** Consolidated business baseline for review and implementation planning  
**Domain:** Asset & Topology  
**Primary actor:** IT Admin

## 1. Purpose

Manage the identity, physical placement, and lifecycle of the infrastructure assets required by the AR-IMMS mini data center proof of concept. The flow establishes a reliable physical context that other flows can reference when registering agents, monitoring nodes, investigating incidents, and performing field maintenance.

This flow defines business behavior and ownership. It does not prescribe database tables, API paths, or a particular digital-twin technology.

## 2. Outcomes

- The seeded default Data Center anchors a usable location hierarchy: **Data Center → Site/Building → Room → Rack**. IT Admins can retrieve the Data Center and manage Sites and Rooms beneath it.
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
- Assign a unique QR identifier to a Rack and generate individual or bulk printable labels in both PDF and SVG.
- Track asset lifecycle separately from placement, Agent binding, and operational health.
- Retain a history of material asset, placement, and marker changes.
- Provide asset identity and confirmed placement for other AR-IMMS flows to reference.

### Out of scope

- Workload and Service registry, container/process inventory, and service dependency topology.
- Telemetry collection, health evaluation, alerting, and Incident lifecycle.
- Agent enrollment, credentials, heartbeat, configuration, unlink, and uninstall. These belong to Agent Management / Flow 2.
- Maintenance and Inspection Activity workflows, checklists, findings, corrective actions, and ticket handling. Asset maintenance-state coordination follows BR-14.
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
| **Maintenance**           | Owns Maintenance Activities and requests Asset maintenance-state changes through an agreed public contract.                                           |
| **Ticket Management**     | Owns Ticket assignment and provides the current-assignee check used for manual maintenance-state authorization.                                       |
| **Agent Management**      | Owns Agent identity and Agent–Node binding; the retirement check uses an agreed public-contract approach.                                             |
| **Other AR-IMMS domains** | Reference asset IDs and confirmed topology; they do not directly modify Asset & Topology data.                                                        |

The Telemetry Agent is not a participant in Flow 1. Its registration and lifecycle are handled by Flow 2.

## 5. Core terms

| Term                | Meaning in Flow 1                                                                                          |
| ------------------- | ---------------------------------------------------------------------------------------------------------- |
| **Site**            | A top-level managed location in the PoC.                                                                   |
| **Data Center**     | The default facility root provisioned through seed/bootstrap; read-only in Flow 1.                         |
| **Room**            | A location within a Site.                                                                                  |
| **Rack**            | A managed rack within a Room, with capacity and placement positions.                                       |
| **Node**            | A managed compute asset. A laptop or PC may represent a Node in the PoC.                                   |
| **Placement**       | The current confirmed relationship recorded through Asset Location between a Node and a Rack position.     |
| **Rack QR**         | A unique identifier used to look up and verify a Rack. It is not the Node ID or Agent identity.            |
| **Asset lifecycle** | One of `ACTIVE`, `MAINTENANCE`, or terminal `RETIRED`; separate from placement, Agent binding, and health. |
| **Agent binding**   | The Agent–Node relationship owned by Agent Management.                                                     |

## 6. Business rules

### BR-01 — Authorized management

Only an authorized IT Admin may create or change Sites, Rooms, Racks, placement, Rack QR mapping, and Asset lifecycle. The default Data Center is provisioned through seed/bootstrap and can be retrieved, but not created or updated through Flow 1.

### BR-02 — Location hierarchy

The parent location must exist before a child location can be created. Sites belong to the seeded default Data Center, Rooms belong to Sites, and Racks belong to Rooms. The system must not accept a location assignment to a missing parent. Reject a Room size or grid update if it would invalidate any current Rack placement.

### BR-03 — Stable asset identity

Each Node has an AR-IMMS identity that remains stable when its hostname, IP address, Rack placement, QR label, or Agent binding changes. Those values must not be treated as interchangeable identifiers.

### BR-04 — Rack capacity and placement

A Node may be assigned only to an existing Rack with available capacity and a free, valid position. The system rejects a placement that exceeds Rack capacity or conflicts with an occupied position. Rack map coordinates use the lower-left anchor of the unrotated Rack footprint. The exact physical footprint rules for multi-U equipment are not defined by this business flow.

### BR-05 — Unassigned Nodes are valid

A Node may exist without a Rack placement. Creating or unmapping a Node does not delete or retire it.

### BR-06 — Placement changes are explicit

Assign, move, and unmap are separate asset-management operations. Moving a Node records the previous and new placement. Unmapping releases its Rack position and retains the Node record and its history.

### BR-07 — Rack identification

Each Rack has at most one active QR mapping, and an active QR value identifies no more than one Rack. A Rack QR remains associated with the Rack when a Node is moved or unmapped.

### BR-08 — QR and spatial tracking are distinct

The QR is used to identify or look up a Rack. It does not by itself define 3D geometry, a persistent spatial anchor, or precise navigation. AR alignment behavior belongs to the AR client.

### BR-09 — Lifecycle is separate from placement and health

Asset lifecycle states are `ACTIVE`, `MAINTENANCE`, and terminal `RETIRED`. New Assets start `ACTIVE` and unassigned. `ACTIVE` describes lifecycle only; it does not imply placement, Agent binding, or healthy operation. Placement is represented by Asset Location, and Agent binding is owned by Agent Management. Unmapping does not change lifecycle state.

### BR-10 — Retirement constraints

A Rack cannot be retired while Nodes remain placed in it. Retire an Asset only from `ACTIVE`, after maintenance has ended, it has been unmapped, and Agent Management confirms its binding has been removed. `RETIRED` cannot be reversed. Preserve the retired record and its references.

### BR-11 — Agent Management boundary

Flow 1 owns the Node identity and physical placement. Agent Management owns bootstrap tokens, Agent identity, Agent–Node binding, credentials, heartbeat/control state, unlink, and uninstall. Agent enrollment requires an eligible, assigned Node. Moving or unmapping a Node does not silently unlink or delete its Agent.

### BR-12 — Other domains reference assets

Monitoring, Incident, Maintenance, Inspection, and AR features reference the canonical Asset ID. They do not write directly into Asset & Topology storage or redefine the Node's physical placement.

### BR-13 — Change history

Material changes to locations, asset lifecycle, placement, or Rack QR mapping record actor, source, and timestamp through the existing Audit mechanism. The history must be sufficient to determine what changed and the current confirmed placement; do not create a separate Audit store.

### BR-14 — Maintenance state coordination

A Maintenance Activity entering `EXECUTE` changes its linked Asset to `MAINTENANCE`. `RESUME` or `ABORT` returns the Asset to `ACTIVE` only when no other Activity is executing for that Asset. A Ticket becoming `OPEN` or `CLOSED` alone does not change Asset state. IT Admin is the default actor for manual maintenance-state changes. The current assignee may change state only for an Asset linked to that assignee's currently assigned Ticket. Manual action cannot return an Asset to `ACTIVE` while any Activity is executing for it.

## 7. Functional requirement table

| ID            | Business requirement                                                                                                               | Acceptance criteria                                                                                                                                                                                                                             |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **F1-REQ-01** | Asset and location management must be restricted to an authorized IT Admin.                                                        | An unauthorized user cannot create or change locations, placements, Rack QR mappings, or asset lifecycle.                                                                                                                                       |
| **F1-REQ-02** | The system must maintain the location hierarchy **seeded Data Center → Site → Room → Rack**.                                       | The default Data Center is retrievable but read-only through Flow 1; a Room cannot be created without an existing Site; a Rack cannot be created without an existing Room. Room size/grid changes that invalidate Rack placements are rejected. |
| **F1-REQ-03** | Each managed Node must have a stable AR-IMMS identity.                                                                             | Changing a Node’s hostname, IP address, Rack placement, QR mapping, or Agent binding does not change its AR-IMMS identity.                                                                                                                      |
| **F1-REQ-04** | A Node may exist without being assigned to a Rack.                                                                                 | A newly created or unmapped Node remains available in the registry as unassigned; it is not deleted or retired.                                                                                                                                 |
| **F1-REQ-05** | The system must represent Rack capacity and valid placement positions, including the required Column/Position and U position.      | A placement that exceeds Rack capacity or conflicts with an occupied position is rejected. Rack map coordinates use the lower-left anchor of the unrotated footprint. The original placement remains unchanged when a move is rejected.         |
| **F1-REQ-06** | Assigning, moving, and unmapping a Node must be separate operations.                                                               | Assign and move establish a valid Rack placement; unmap releases the position while retaining the Node record, identity, and history.                                                                                                           |
| **F1-REQ-07** | Each Rack must have a unique QR identifier for lookup and field verification.                                                      | An active QR identifies only one Rack. Scanning or looking up the QR resolves to that Rack. Changing a QR does not change the Rack ID.                                                                                                          |
| **F1-REQ-08** | IT Admin must be able to generate individual or bulk printable Rack QR labels in both PDF and SVG.                                 | Each generated label contains the Rack QR and readable Rack identity/location information and can be physically applied and scanned.                                                                                                            |
| **F1-REQ-09** | Asset lifecycle is `ACTIVE`, `MAINTENANCE`, or terminal `RETIRED`, separate from placement, Agent binding, and operational health. | New Assets start `ACTIVE` and unassigned. Placement is recorded through Asset Location; Agent binding belongs to Agent Management. `ACTIVE` does not imply placement, Agent binding, or healthy operation.                                      |
| **F1-REQ-10** | An Asset or Rack may be retired only when its retirement preconditions are met.                                                    | Retire an Asset only from `ACTIVE`, after maintenance ends, it is unmapped, and Agent binding is removed. `RETIRED` cannot be reversed. A Rack with placed Nodes cannot be retired. Preserve history.                                           |
| **F1-REQ-11** | Material changes to locations, lifecycle, placement, and Rack QR mapping must be traceable.                                        | The existing Audit mechanism records actor, source, and timestamp; no separate Audit store is introduced.                                                                                                                                       |
| **F1-REQ-12** | Other AR-IMMS flows must reference assets using the canonical Asset ID.                                                            | Other domains can retrieve confirmed asset identity and physical placement; they do not directly modify Asset & Topology records.                                                                                                               |
| **F1-REQ-13** | Asset and placement data must support the AR Digital Twin view.                                                                    | The view can represent Site/Room/Rack/Node placement and use the Rack QR to verify a physical Rack. Asset & Topology does not own 3D geometry, camera tracking, or spatial anchors.                                                             |
| **F1-REQ-14** | Flow 1 must remain limited to confirmed asset-management needs.                                                                    | Workload/Service registry, telemetry, monitoring health, Incident lifecycle, Maintenance Activity workflows, and extended DCIM capabilities remain outside Flow 1. Asset maintenance-state coordination follows BR-14.                          |

> **Review note**: **F1-REQ-08** limits label content to Rack identity/location; fields such as IP address are not included in this baseline. Multi-U equipment footprint rules remain deferred.

## 8. Main flow — provision and place a Node

1. Seed/bootstrap provisions the default Data Center; the IT Admin retrieves it.
2. The IT Admin creates a Site under the default Data Center if it does not already exist.
3. The IT Admin creates a Room under that Site.
4. The IT Admin creates a Rack under that Room and records its capacity and required placement coordinates, including Column/Position and U position. Rack map coordinates use the lower-left anchor of the unrotated footprint.
5. The IT Admin creates a Node record. AR-IMMS assigns its stable identity; the Asset starts `ACTIVE` and unassigned.
6. The IT Admin selects the Node, Rack, and target position.
7. AR-IMMS validates that the Node and Rack are eligible, the Rack has capacity, and the position is free.
8. If validation succeeds, AR-IMMS records the placement as an Asset Location and retains its history. The Node is now available as a physical asset reference to other flows.
9. The IT Admin may assign a unique QR to the Rack and generate a single or bulk label for printing. A Technician may physically apply and scan the label to verify the Rack.
10. If Agent collection is required, the authorized user continues with Flow 2. Flow 1 does not perform Agent enrollment.

## 9. Alternative and lifecycle flows

### 9.1 Move a Node

1. The IT Admin selects an assigned Node and a destination Rack position.
2. AR-IMMS validates the destination Rack and position.
3. If valid, AR-IMMS replaces the confirmed placement, releases the previous position, and records the change.
4. The Node ID remains unchanged. The Agent binding, if any, remains owned by Agent Management and is not silently removed.

If the destination Rack has insufficient capacity or the position is occupied, the move is rejected and the original placement remains unchanged.

### 9.2 Unmap a Node

1. The IT Admin requests that a Node be removed from its Rack position.
2. AR-IMMS releases the position and records the unmapping.
3. The Node remains in the registry as unassigned. Its identity, history, and Rack QR records are retained. Agent unlink and retirement are separate operations.

### 9.3 Retire an asset

1. The IT Admin requests retirement of a Node or Rack.
2. For a Rack, AR-IMMS verifies that it contains no placed Nodes.
3. For an Asset, retirement is allowed only from `ACTIVE`, after maintenance has ended, the Asset is unmapped, and Agent Management confirms its binding has been removed.
4. If the preconditions pass, AR-IMMS marks the Asset `RETIRED` permanently and preserves its history and references. Otherwise, the request is rejected with the unmet precondition.

### 9.4 Manage a Rack QR

The IT Admin may create, replace, or deactivate a Rack QR independently of Node placement. A QR lookup returns the Rack it identifies. A QR change does not change the Rack ID or any Node ID.

## 10. Failure and exception handling

| Condition                                                         | Expected behavior                                                                                     |
| ----------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| Parent Site or Room does not exist                                | Reject child location creation.                                                                       |
| Rack does not exist or is not eligible                            | Reject Node placement.                                                                                |
| Rack capacity is exceeded or target position is occupied          | Reject placement/move; preserve the prior placement.                                                  |
| QR is already active on another Rack                              | Reject the duplicate mapping.                                                                         |
| Rack still contains placed Nodes                                  | Reject Rack retirement.                                                                               |
| Asset is in maintenance, is placed, or still has an Agent binding | Reject Asset retirement until maintenance ends, it is unmapped, and Agent Management confirms unlink. |
| Room size/grid update would invalidate a Rack placement           | Reject the proposed Room update and preserve the existing Room and Rack layout.                       |
| IT Admin unmaps a Node                                            | Release the position only; do not delete, retire, or unlink the Node.                                 |

## 11. Resulting system responsibilities

- **Asset & Topology** is the source of truth for Site/Room/Rack/Node identity, confirmed physical placement, Rack QR mapping, and asset lifecycle.
- **Agent Management** is the source of truth for Agent identity and Agent–Node binding. A Flow-level application coordinator uses the Asset & Topology and Agent Management public contracts for cross-domain operations: a Node must be `ACTIVE` and placed before binding, and retirement requires a confirmed unbound result from Agent Management. The coordinator fails closed if binding status is unavailable and does not replace either domain as the source of truth. AT-10 must define and verify protection against concurrent bind and retirement operations.
- **Monitoring/Observability** is the source of truth for telemetry and observed runtime/service behavior.
- **Maintenance** owns Maintenance Activities and requests Asset state changes through an agreed public contract; **Ticket Management** owns assignment and provides the current-assignee check for manual maintenance authorization.
- Other domains use the public contracts of the domains that own the requested data.

## 12. Digital Twin and AR usage

Asset & Topology supplies the authoritative Site/Room/Rack/Node identity and confirmed physical placement used by the AR Digital Twin view. The view may represent the Rack and its placed Nodes and use the Rack QR to verify the physical Rack. Asset & Topology does not own 3D geometry, camera tracking, spatial anchors, telemetry, or live health. Those concerns remain with the AR client and their owning domains.

## 13. Deferred decisions

These details are intentionally not invented by Flow 1 and should be decided only when a concrete use case requires them:

- Exact asset metadata fields beyond the identity, hierarchy, capacity, and placement information required by the flow.
- The physical footprint rules for Nodes that occupy multiple U positions.
- Whether any additional asset classes are needed.
- Label fields beyond the Rack QR and the readable Rack identity/location needed for field verification.
- Any additional topology such as power, network, cooling, cloud, or Workload/Service relationships.
