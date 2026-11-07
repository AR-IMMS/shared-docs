# AR-IMMS Glossary

> **Document type:** Shared product and domain terminology  
> **Revision:** Draft for review — 2026-11-08  
> **Related baseline:** [`PRODUCT.md`](PRODUCT.md)  
> **Purpose:** Use consistent meanings across business flows, architecture, contracts, and repository-local documentation. These definitions do not assert implementation completeness.

## 1. Infrastructure and Digital Twin

| Term | Definition |
| --- | --- |
| **Asset** | A registered infrastructure item managed by AR-IMMS, such as a Node or Rack. Do not assume every monitored Workload is a registered Asset; its relationship to assets is established through resource mapping. |
| **Site** | A top-level location used to organize infrastructure in the asset inventory. |
| **Room** | A location within a Site containing Racks or other represented infrastructure. The PoC may model its layout using a simplified grid. |
| **Rack** | A physical or logically represented grouping/location for Nodes. In the PoC, it need not be a full-size physical rack. |
| **Node** | A registered computing resource associated with the monitored infrastructure. A laptop/PC typically represents a Node in the PoC; the asset model may also represent an eligible VM as a Node. |
| **Virtual machine (VM)** | A software-defined computer running on a host. It may be monitored as a separate resource/Node where the asset and agent mapping support it. |
| **Asset placement** | The recorded relationship between an Asset and its infrastructure location, such as a Node placed in a Rack. |
| **Workload** | An application or application group running on one or more computing resources, commonly composed of multiple Services. A Workload is a monitored logical resource; it is not automatically a first-class asset-registry entry. |
| **Service** | A distinct functional or observable component within a Workload (for example, an API, worker, or database service). |
| **Resource identity / mapping** | The information used to relate observed telemetry and Workloads/Services to registered Nodes and relevant infrastructure context. |
| **Digital twin** | The digital representation of registered infrastructure, locations, and relationships. In this PoC it is simplified and does not promise a complete or synchronized physical replica. |

## 2. Agent and Observability

| Term | Definition |
| --- | --- |
| **Telemetry Agent** | Software installed on a monitored host or VM to manage its monitoring identity, configuration and supported integrations, and help supply telemetry. It is a non-human system actor. |
| **Agent installation** | A particular installed instance of the Telemetry Agent with its own installation identity and lifecycle. |
| **Agent enrollment** | The authorized process by which an installation registers with Core and obtains a persistent credential. |
| **Agent binding** | The association of an Agent installation with a registered Node, with changes tracked according to the agent lifecycle. |
| **Bootstrap token** | A short-lived, limited-use credential issued to authorize initial Agent enrollment for an eligible Node. It is not the Agent's long-term operating credential. |
| **Agent configuration snapshot** | The desired monitoring configuration provided by Core for an Agent to validate, reconcile, and activate. The acceptance and activation rules are specified in integration contracts. |
| **Integration** | An Agent-supported monitoring capability for a target or technology (such as Redis or PostgreSQL), typically contributing configuration to the collector. Not every integration is automatically discovered or available on every platform. |
| **Collector** | The OpenTelemetry Collector runtime that receives, processes, and exports supported telemetry as the Agent's data plane. It is distinct from the Agent's control plane. |
| **Exporter** | A component exposing or exporting telemetry for collection or delivery. Its exact role depends on the monitoring integration; it is not synonymous with the Telemetry Agent or Collector. |
| **Telemetry** | Operational signals such as metrics, logs, and traces associated with monitored resources. |
| **Metric** | A quantitative measurement sampled or aggregated over time, such as CPU use or request error rate. |
| **Log** | A timestamped event record emitted by a host, application, or service. |
| **Trace** | A record of an operation's execution across one or more services, represented through spans where instrumentation is available. |
| **Service dependency** | An observed or inferred relationship between services, potentially supported by trace data; its completeness depends on instrumentation. |
| **Monitoring completeness** | The extent to which expected monitoring signals or exporters are available. It is about coverage, not proof that the asset is functioning correctly. |
| **Telemetry freshness** | Whether telemetry is recent enough to be usable under applicable monitoring rules. |
| **Node monitoring state** | A monitoring-coverage classification based on expected exporter availability. In the current baseline terminology, **HEALTHY** means all expected exporters report, **DEGRADED** means only some report, and **NO DATA** means none report. These names do **not** assert hardware health; exact thresholds belong in monitoring rules. |

## 3. Detection, Investigation, and Incident

| Term | Definition |
| --- | --- |
| **Alert rule** | A monitoring condition configured to generate an alert when specified criteria are met. Grafana Alerting is the selected alerting boundary for the PoC. |
| **Alert** | A signal or alert instance emitted when monitoring identifies a condition requiring attention. It is not automatically a confirmed Incident or Ticket. |
| **Detection episode** | A contextual grouping of related alerts/signals for correlation and assessment. Its exact persistence and lifecycle remain subject to the monitoring/incident design. |
| **Incident candidate** | A suspected operational issue assembled from relevant detected signals and context that has not yet met the criteria for a confirmed Incident. Confirmation policy remains to be defined. |
| **Incident** | A confirmed, tracked operational issue requiring investigation or coordinated response. It can be associated with affected resources and one or more supporting observations. |
| **Impact assessment** | An evaluation of which assets, workloads, or services may be affected by an Incident and to what extent, based on available evidence. |
| **Affected resource** | A Node, Workload, Service, or other identified resource associated with the impact or scope of an Incident. |
| **Incident evidence** | An alert, log, trace, metric query result, change record, observation, or other referenced information supporting an incident assessment. |
| **Correlation** | The process of relating observations that may represent the same or connected operational issue, using available context and evidence. Correlation is not proof of causation. |
| **Pattern Catalog** | A collection of reusable incident patterns, fingerprints, and historical knowledge that may support matching and investigation. Its automation and learning behavior are not assumed implemented. |
| **Runbook** | Documented guidance for investigation or operational work. AR-IMMS may store runbook metadata or references while keeping the source content in external Markdown documents. |
| **AI-assisted investigation** | Evidence-based analysis and suggestions that support human assessment of an operational issue. AI does not autonomously authorize or execute remediation. |

## 4. Ticket and Maintenance

| Term | Definition |
| --- | --- |
| **Ticket** | A trackable work item with responsibility, status, and discussion/history. It may be linked to an Incident or created independently. The external ticketing system is the source of truth for ticket lifecycle data in the planned integration. |
| **Ticket owner** | The person accountable for the work item and its outcome under the business workflow. Ownership is distinct from operational assignment. |
| **Ticket assignee** | The person assigned to perform or handle work associated with a Ticket. The owner and assignee may differ. |
| **Maintenance activity** | Work performed to inspect, verify, or address a condition, typically in the PoC as part of ticket-driven maintenance. |
| **Corrective maintenance** | Maintenance performed in response to an identified problem or Incident. |
| **Preventive maintenance** | Scheduled or proactive work intended to reduce future problems. It is a valid domain term but **outside the current PoC baseline flow**. |
| **Inspection** | A structured set of checks and observations performed against relevant assets, often using a checklist. It records what was examined and the findings. |
| **Checklist** | A predefined or reviewed set of inspection/maintenance steps used to guide and record work. AI-generated drafts require human review. |
| **Maintenance Record** | The record of work actually performed, including actions, responsible personnel, results and relevant evidence. Unlike an Inspection, it documents the broader maintenance activity and outcome. |
| **Evidence** | Supporting material such as a photo, measured value, observation, log reference, or other artifact attached to an inspection or maintenance outcome. |
| **Asset verification** | Confirmation that the technician is interacting with the intended registered Asset, for example by scanning its QR code/marker. This is distinct from verifying that a fault has been resolved. |
| **Recovery verification** | A check of relevant system signals or operational behavior after work is performed to assess whether the reported condition has improved or been resolved. |
| **Controlled operation** | An explicitly authorized, user-confirmed action performed on a target Service or Workload. A service restart is a candidate PoC demonstration, not yet a universally committed capability. |

## 5. Field Support and Product Scope

| Term | Definition |
| --- | --- |
| **QR mapping** | The maintained association between a QR code/marker and a registered target, such as a Rack or Node, used for identification and verification. |
| **Augmented reality (AR)** | Digital content presented within the user's view of the physical environment. |
| **Mixed reality (MR)** | A spatial interface that combines the user's physical surroundings with digitally positioned information and interactions; AR/MR are used as related field-support terms in AR-IMMS. |
| **MR-assisted maintenance** | Technician-led field work supported by in-place asset context and verification through the MR interface. Navigation and advanced 3D procedures are not assumed baseline capabilities. |
| **Proof of concept (PoC)** | A bounded implementation used to demonstrate selected product capabilities through reproducible scenarios on a constrained testbed. |
| **Fault injection** | The deliberate introduction of a controlled failure or degraded condition to test monitoring, investigation, and maintenance workflows. |
| **Ground truth** | The known facts of a controlled test scenario (such as the injected fault and timing), used to evaluate observed system behavior. |

## Naming and Boundary Notes

- Use **Asset** for registered infrastructure items; use **Workload** and **Service** for logical application resources unless their registration as Assets is explicitly defined.
- Do not conflate **host**, **VM**, **Node**, or **Agent installation**: they represent different concepts and may not have one-to-one relationships.
- Do not use **Agent**, **Exporter**, and **Collector** interchangeably; distinguish the control plane from telemetry-producing or data-plane components.
- **Monitoring state** and **telemetry freshness/completeness** do not prove an asset is physically healthy. The label `HEALTHY` is inherited terminology for coverage and should be displayed with that qualification.
- Keep **Alert → Detection Episode / Incident Candidate → Incident → Ticket** conceptually distinct. The exact candidate confirmation and grouping policies remain open design decisions.
- Use **Inspection** for structured checks/findings and **Maintenance Record** for the work performed and its overall outcome.
- Use **asset verification** for checking identity and **recovery verification** for checking results after maintenance.
- **AI recommendations** never imply autonomous authorization or execution. A proposed controlled service restart requires separate approved workflow and safeguards.
- **Preventive maintenance**, rich MR navigation, and full digital-twin fidelity are not current baseline commitments.

## Document Boundaries

This glossary establishes shared meanings, not field-level schemas, HTTP contracts, implementation status, or final lifecycle policies. Consult [`PRODUCT.md`](PRODUCT.md) for scope; `business/` for rules and workflows; `contracts/` and `integration/` for precise inter-system behavior; and repository-local `docs/local/` for implementation details. Terms reflecting pending design (such as Incident Candidate policies or controlled operations) must not be interpreted as already implemented.
