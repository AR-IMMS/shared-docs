# AR-IMMS — Product Overview

> **Document type:** Product baseline (business level)  
> **Project:** AR-IMMS — Augmented Reality Infrastructure Monitoring and Maintenance System  
> **Scope:** Proof of Concept (PoC)  
> **Revision:** Approved — 2026-11-08  
> **Status note:** This document describes intended product behavior, **not** implementation completion. See repository-local implementation and verification reports for delivery status.

## 1. Document Purpose

This document defines the product problem, intended users, objectives, boundaries, major business capabilities, and success criteria of AR-IMMS. It is the entry point for business analysts, developers, testers, and reviewers.

It is **not** a detailed SRS, implementation design, API contract, or progress report. Detailed rules and workflows belong in `business/`; cross-system design belongs in `architecture/`, interface definitions in `contracts/`, integration behavior in `integration/`, and implementation details in each repository's `docs/local/`.

When requirements or scope change, update the relevant business-flow documentation and this product baseline deliberately; do not silently infer product commitments from partially implemented code.

## 2. Product Overview

**AR-IMMS** is a proof-of-concept platform that connects infrastructure asset context, operational monitoring, incident and ticket handling, and technician-led maintenance. It aims to demonstrate a traceable path from an observed condition to investigation, assigned work, on-site asset verification, execution, and recorded outcomes.

A registered infrastructure model provides the context for monitoring data. Telemetry Agents and application instrumentation supply selected operational signals. Operators review detected conditions and, when warranted, progress them into incidents and tickets. Maintenance Technicians verify the intended asset, consult operational and procedural context in an augmented/mixed-reality (AR/MR) interface, perform assigned work, and record evidence and results. AI can assist investigation through evidence-based analysis and suggestions, but does not make binding operational decisions or autonomously remediate incidents.

The PoC runs on a small, resource-constrained lab testbed; it does **not** claim to be a production-scale data center infrastructure management system.

## 3. Problem and Product Direction

### 3.1 Problem

Monitoring systems can expose unhealthy conditions without reliably connecting those signals to the affected physical or logical asset, the responsible workflow, or an eventual maintenance record. Separately maintained asset inventories, alerts, tickets, investigation notes, and field observations make it harder to understand what happened and what was done. Technicians also need to verify that they are operating on the correct asset when moving from a dashboard to the work site.

### 3.2 Product Direction

AR-IMMS seeks to make the operational context **continuous and traceable**:

**Registered asset / workload → observed signals → alert / detected condition → incident assessment → ticket → on-site verification and maintenance → evidence and outcome.**

The PoC emphasizes:

- **Asset-grounded operations:** associate monitoring and work records with registered infrastructure and relevant workloads.
- **Human-controlled response:** operators and technicians review, confirm, and perform consequential actions.
- **Field verification:** use QR/physical markers to confirm the intended asset before on-site work; present contextual information through AR/MR.
- **Evidence-based investigation:** connect alerts with relevant logs, traces, and, where useful, metric queries and historical patterns.
- **Traceability:** preserve the relationship between a detected condition, incident/ticket, maintenance activity, verification, and outcome.

AR/MR is a field-work interaction channel, **not** a replacement for the monitoring or ticketing system. AI is a decision-support capability, **not** an autonomous operator.

## 4. Users and System Actors

| Actor                             | Product-level responsibility                                                                                                                       |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **System Monitoring Operator**    | Reviews monitoring information and detected conditions; investigates, assesses, and coordinates incident/ticket handling.                          |
| **IT Admin**                      | Manages registered infrastructure assets and their locations, access, and monitoring preparation; authorizes and manages agent registration.       |
| **Maintenance Technician**        | Accepts assigned work; verifies target assets on site; follows applicable inspection/maintenance steps; records actions, evidence, and outcomes.   |
| **Telemetry Agent** _(non-human)_ | Enrolls against an authorized Node, applies supported monitoring configuration, and supplies selected monitoring and health/inventory information. |

Other technical systems (for example, observability tools and the ticket system) are integration participants, not human business roles. Precise permissions, owner/assignee responsibilities, and system-to-system interactions are specified in the relevant business-flow and integration documents.

## 5. Business Flows and Product Capabilities

AR-IMMS organizes its main product behavior into **five business flows**. The flows describe desired business outcomes, not five independent services.

| Flow                                                            | Business outcome                                                                           | Primary capabilities                                                                                                                                                                                        |
| --------------------------------------------------------------- | ------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **01 — Infrastructure Asset Management**                        | Maintain a trustworthy inventory and placement context.                                    | Site/Room/Rack/Node representation, logical location/placement, asset state, QR mappings, asset verification context.                                                                                       |
| **02 — Telemetry Agent Registration & Management**              | Reliably associate an authorized Agent installation with a registered Node.                | Agent bootstrap/enrollment, identity and binding, management/revocation, configuration and operational status as supported.                                                                                 |
| **03 — Monitoring & Anomaly Detection**                         | Observe infrastructure and selected workloads and identify conditions requiring attention. | Node/workload signals, data freshness/completeness, dashboards, alerting, contextual detection and incident investigation inputs.                                                                           |
| **04 — Ticket Management**                                      | Make accountable, trackable work available to users.                                       | Incident-related or manually created tickets, ownership/assignment, status progression, discussion and business context, links to affected resources.                                                       |
| **05 — Incident-driven Maintenance and AI/AR-assisted Support** | Turn an actionable incident/ticket into verified, documented maintenance.                  | Incident assessment, evidence and AI-assisted investigation, technician assignment, QR verification, inspection/checklist, controlled execution, recovery verification, maintenance records and MR context. |

An alert is not automatically a confirmed incident; an incident is not itself a ticket; and a ticket does not imply physical maintenance has already been performed. These distinctions must be maintained in the detailed workflows.

## 6. Product Goals

The PoC aims to demonstrate that:

1. Registered Nodes and selected Workloads can be related to their monitoring signals and current operational context.
2. Users can distinguish **an observed problem** from **missing, stale, or incomplete telemetry** rather than treating absent data as healthy.
3. Selected alerts can be assessed, correlated, and placed into an understandable incident context with supporting operational evidence.
4. An operator can initiate or manage accountable ticket-based work, with links to the affected resource or incident where applicable.
5. A technician can identify and verify the intended asset on site and consult useful AR/MR context before and during assigned work.
6. Inspection, execution, recovery checks, and maintenance outcomes can be documented and traced back to the work item and asset(s).
7. AI can produce evidence-grounded investigation support without replacing human judgment or performing autonomous remediation.
8. The preceding capabilities can be demonstrated credibly on a small testbed, with explicit limitations and reproducible scenarios.

## 7. PoC Scope and Boundaries

### 7.1 In Scope

**Infrastructure and identity**

- Represent a small inventory of Sites/Rooms/Racks/Nodes, including asset placement and the logical/physical relationship needed for demonstrations.
- Use a registered Node as the target for Agent enrollment and monitoring identity; support authorized enrollment and agent lifecycle management.
- Support QR-based identification/verification for selected Rack and Node assets.

**Monitoring and investigation**

- Collect and present selected Node/host/VM and workload/service/database telemetry supported by the lab setup.
- Surface telemetry freshness/completeness and avoid interpreting unavailable signals as healthy.
- Use selected operator-configured alert rules and demonstration fault scenarios to initiate investigation.
- Connect relevant alerts and context into incident assessment; use alerts, logs, traces, asset/workload identity, and metric queries when available.
- Support selected service relationships/impact hints when trace coverage permits; do not promise comprehensive root-cause or blast-radius accuracy.

**Tickets and maintenance**

- Support tickets associated with incidents **and** tickets created without a prior incident.
- Support **ticket-driven** maintenance covering software and basic infrastructure checks; work may relate to more than one asset.
- Allow a technician to verify target assets, use a checklist/inspection where relevant, record work and evidence, check recovery, and complete a maintenance report/outcome.
- Provide AR/MR-assisted field context and asset verification. Navigation or richer spatial interaction is a demonstration enhancement subject to the selected MR implementation.

**AI and controlled actions**

- Provide AI-assisted investigation and optional draft guidance based on available evidence and known procedures; users review conclusions and actions.
- Explore a **single user-confirmed service operation (initial candidate: restart)** as part of a selected maintenance demonstration. The exact execution mechanism, safeguards, and delivery commitment require a separate approved workflow/contract before implementation; this text does not authorize AI-initiated execution.

The scope is intentionally **scenario-based**, not exhaustive across every fault, device, or workload type.

### 7.2 Out of Scope for the PoC Baseline

- Production-grade DCIM, high availability, multi-site scale, or enterprise operational guarantees.
- Comprehensive hardware, power, cooling, networking, or facility management and diagnosis.
- Physical component replacement or actual repair of production hardware.
- General-purpose automated remediation, arbitrary shell execution, or AI-authorized operational actions.
- Preventive or calendar-driven maintenance as a main business flow; baseline maintenance is ticket-driven.
- Guaranteed root-cause identification, accurate blast-radius estimation in every case, or detection of all fault classes.
- Complex step-by-step 3D repair guidance or full-fidelity spatial digital twins as required deliverables.

### 7.3 Possible Future Extensions (Not Current Commitments)

- Broader cooling/power/environmental asset and telemetry types, fault catalogs, integrations, and automated discovery.
- Richer spatial digital twins, navigation, and interactive MR procedures.
- Additional approved operations and remediation workflows with appropriate controls.
- Broader analytics and historical pattern learning, more mature incident correlation, and larger testbed evaluations.
- Scheduled/preventive maintenance and production deployment capabilities.

## 8. End-to-End Product Journey

A representative **incident-driven** journey is:

1. **Prepare:** an Admin registers and places assets, then enables authorized monitoring for a Node.
2. **Observe:** telemetry arrives; monitoring presents usable data or identifies gaps/staleness.
3. **Detect:** alerting identifies a selected condition; the operator reviews it with supporting signals.
4. **Assess:** contextual information is correlated into an incident candidate and, where justified, a confirmed incident. AI may offer supporting analysis, subject to user review.
5. **Coordinate:** an operator creates or links a ticket and assigns accountable work.
6. **Verify on site:** the technician locates the intended asset and confirms identity through its QR/marker; AR/MR provides related context.
7. **Perform and record:** the technician inspects, follows an applicable checklist/runbook, performs an authorized action, and records evidence.
8. **Check and conclude:** the technician/operator reviews recovery and records the result; the ticket advances according to its defined lifecycle.

Not every observed alert becomes an incident, not every incident requires on-site work, and tickets may also originate independently of monitoring. Detailed normal and exception paths belong in the business-flow documents.

## 9. PoC Operating Context

The PoC uses a resource-constrained testbed, typically **up to three laptops/PCs or equivalent Nodes**, to emulate selected infrastructure interactions:

| Concept                | PoC representation                                                                                  |
| ---------------------- | --------------------------------------------------------------------------------------------------- |
| **Node**               | A registered laptop/PC or represented host/VM asset, according to the asset model.                  |
| **Rack / Room**        | Logical grouping and placement context, with physical or virtual markers for verification.          |
| **Workload / Service** | Selected containerized applications, databases, and their observable service activity.              |
| **Monitoring source**  | Agent-managed collectors/exporters and application instrumentation.                                 |
| **Detected condition** | Selected real or deliberately injected faults, supported by known test scenarios.                   |
| **Field context**      | QR/marker-based identification and an AR/MR client (Meta Quest-class device for the demonstration). |

Metrics, logs, and traces may have unequal coverage depending on the chosen workload and instrumentation. Synthetic activity and fault injection may be used for repeatable evaluation; the demo must distinguish injected ground truth from observations and AI interpretations.

The detailed observability stack, integrations, deployment topology, and specific test workloads belong in architecture and engineering documentation, not in this product baseline.

## 10. Success Criteria and Evaluation Principles

The PoC is successful to the extent that selected **end-to-end scenarios** visibly demonstrate the following outcomes:

| ID        | Expected demonstrable outcome                                                                                                            |
| --------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| **SC-01** | A registered Node/Workload is identifiable in monitoring and linked to its relevant asset context.                                       |
| **SC-02** | A user can identify stale, absent, or incomplete monitoring data rather than misreading it as a healthy state.                           |
| **SC-03** | A selected fault results in an observable alert/condition with evidence sufficient for operator assessment.                              |
| **SC-04** | Incident-related context can be linked to an actionable ticket; independent/manual ticket creation remains possible.                     |
| **SC-05** | A technician verifies the intended asset using a QR/marker and views relevant field context through AR/MR.                               |
| **SC-06** | Assigned maintenance steps, evidence, and a recovery/outcome check are recorded and associated with the correct ticket and asset(s).     |
| **SC-07** | AI output identifies its supporting evidence and is presented as advice requiring human judgment.                                        |
| **SC-08** | If the selected restart-operation demo is approved and implemented, it requires explicit user confirmation and has an observable result. |
| **SC-09** | A reproducible scenario report identifies injected faults, observed behavior, implemented boundaries, and unimplemented limitations.     |

Detailed acceptance criteria, test fixtures, evaluation metrics (including detection/correlation accuracy and latency where feasible), and scenario selection must be maintained separately. A proposed success criterion is **not evidence that it already passes**.

## 11. Documentation Map

Use the following **existing paths** as entry points; do not treat future documents as already present:

| Area                              | Reference                                                                                            |
| --------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Product terminology               | [`GLOSSARY.md`](GLOSSARY.md)                                                                         |
| Flow 01 — Asset Management        | [`business/01-infrastructure-asset-management.md`](business/01-infrastructure-asset-management.md)   |
| Flow 02 — Agent Registration      | [`business/flow-02-agent-registration-business.md`](business/flow-02-agent-registration-business.md) |
| Telemetry Agent overview          | [`architecture/telemetry-agent-overview.md`](architecture/telemetry-agent-overview.md)               |
| Agent enrollment API              | [`contracts/openapi/agent-enrollment.openapi.yaml`](contracts/openapi/agent-enrollment.openapi.yaml) |
| Core–Agent enrollment integration | [`integration/telemetry-agent-core-enrollment.md`](integration/telemetry-agent-core-enrollment.md)   |
| Engineering policy                | [`engineering/AI_CODING_POLICY.md`](engineering/AI_CODING_POLICY.md)                                 |

**Documentation gaps:** dedicated business-flow documents for Monitoring & Anomaly Detection (03), Ticket Management (04), and Incident-driven Maintenance with AI/AR Support (05) are planned but not present in the supplied shared-docs tree. Future system-level architecture and configuration/reporting contracts likewise require review before being linked here.

## 12. Open Product Decisions and Assumptions

These items should be resolved in the appropriate flow, ADR, or integration contract rather than silently decided inside this overview:

| Topic                              | Current position / needed resolution                                                                                                                                          |
| ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Controlled service restart**     | Candidate human-confirmed demo; finalize authority, target, safety constraints, execution path, rollback/recovery behavior, and whether it is a required acceptance scenario. |
| **AR/MR navigation**               | QR verification and in-place context are baseline; decide how much room/rack navigation is feasible and necessary to demonstrate differentiated value.                        |
| **Incident policy**                | Define the exact criteria for candidate vs confirmed incident, alert correlation, deduplication, severity, and incident-to-ticket transition.                                 |
| **Workload identity and coverage** | Agree on which sample workloads/services are modeled and how their telemetry maps to registered Nodes/assets.                                                                 |
| **AI evidence and evaluation**     | Finalize evidence boundaries, pattern catalog usage, trace/metric tool access, review requirements, and success measures.                                                     |
| **Scenario selection**             | Choose the small set of primary E2E demonstrations and identify which components and exception cases each must cover.                                                         |

**Product-level decisions already reflected here:** maintenance is ticket-driven; AI supports analysis but does not autonomously act; the demonstration is intentionally small and scenario-bounded; preventive maintenance is deferred.

---

_This product baseline intentionally separates **desired behavior** from **verified implementation**. Any feature's actual readiness must be corroborated against repository source, tests, and integration evidence._
