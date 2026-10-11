# AR-IMMS Product

## 1. Document Purpose

This document provides a business-level overview of the AR-IMMS product for business analysts, developers, and other project contributors.

It explains the product context, intended users, direction, and goals for the proof of concept (PoC). It is an entry point to the product documentation, not a detailed specification of business workflows, rules, or technical design.

Detailed workflows, business rules, architecture, and integration contracts are maintained in their respective documents and referenced from this document.

## 2. Product Overview

AR-IMMS is a proof-of-concept system for infrastructure monitoring and maintenance support.

It connects infrastructure assets with monitoring information and maintenance activities. Operators can use it to understand detected conditions in relation to the affected assets. Technicians can identify an asset in the field, review relevant context, and record maintenance results. Augmented reality (AR) provides an in-place view of that context, while AI may assist users by analyzing available operational information and providing supporting suggestions.

The PoC uses a small, resource-constrained testbed to demonstrate how these capabilities can work together. It is intended to explore the product approach, not to provide a full-scale data center infrastructure management system.

## 3. Problem and Product Direction

### 3.1 Problem

Infrastructure monitoring can reveal that a condition needs attention, but acting on that information also requires people to understand which asset is affected, verify its identity and condition, and record what was done.

When monitoring information, asset context, and maintenance records are handled separately, users may need to manually connect those pieces of information. This can make investigation and field work harder to follow and can leave maintenance outcomes disconnected from the conditions that prompted them.

### 3.2 Product Direction

AR-IMMS aims to connect monitoring, asset identification, and maintenance support in a shared operational context.

The product direction is to help:

- Operators relate detected conditions to the relevant infrastructure assets.
- Technicians identify and verify assets at the work site and review relevant information there.
- Teams record maintenance activities and their outcomes for later reference.
- Users interpret available operational evidence with AI-assisted analysis and suggestions.

AR is used to present relevant context during field work. AI supports the user’s assessment; it does not replace the user’s responsibility for decisions or actions.

## 4. Users

| User or system actor       | Role at product level                                                                          |
| -------------------------- | ---------------------------------------------------------------------------------------------- |
| System Monitoring Operator | Reviews monitoring information and handles detected conditions.                                |
| IT Admin                   | Maintains infrastructure asset records and prepares assets for monitoring.                     |
| Maintenance Technician     | Identifies assets in the field, performs assigned work, and records its outcome.               |
| Telemetry Agent            | Reports information from a Node to support monitoring. It is a system actor, not a human user. |

Detailed responsibilities, interactions, and workflow steps are defined in the relevant business-flow documents.

## 5. Product Goals

The AR-IMMS PoC aims to demonstrate that:

1. Monitoring information can be understood in relation to registered infrastructure assets.
2. A detected condition can provide useful context for investigation and maintenance work.
3. A technician can identify an asset in the field and review relevant information through an AR interface.
4. Maintenance work and its outcome can be recorded in relation to the affected asset or condition.
5. AI-assisted analysis can provide useful, evidence-based support while leaving operational decisions with the user.
6. These capabilities can be demonstrated using a small testbed, with its limitations made clear.

## 6. Scope

### 6.1 In Scope

The PoC focuses on infrastructure conditions that can be observed through the available testbed and monitoring data.

- **Node monitoring:** observe selected hardware and operating-system signals collected from registered Nodes, together with the freshness and completeness of monitoring data.
- **Workload monitoring:** observe selected service and database signals, including request rate, errors, duration, resource usage, and service communication relationships where available.
- **Incident context:** relate detected conditions to the registered asset or workload involved so users can investigate them.
- **Field support:** allow a technician to identify an asset and review relevant operational context through the AR interface.
- **Maintenance support:** support inspection, verification, and recording of maintenance work and its outcome.
- **Controlled workload operation:** demonstrate one user-confirmed operational action, initially restarting a service, as part of a maintenance scenario.
- **AI assistance:** provide analysis or suggestions based on available operational evidence to support the user’s assessment.

The PoC covers selected scenarios within these areas. It does not aim to detect or resolve every possible infrastructure problem.

### 6.2 Out of Scope

The PoC does not aim to:

- Provide complete coverage of all data center faults, assets, or operational procedures.
- Diagnose every cause of a hardware, network, power, cooling, or facility problem.
- Demonstrate physical component replacement or repair.
- Automatically perform remediation without user review and confirmation.
- Provide production-scale data center infrastructure management, high availability, or operational guarantees.
- Treat AI output as an authoritative diagnosis or as a replacement for user decisions.

### 6.3 Future Direction

The following may be considered after the PoC, but are not commitments for its current scope:

- Broader asset, telemetry, and fault coverage.
- Additional workload operations and remediation options.
- More complete digital-twin visualization and spatial interaction.
- Expanded AI analysis using a wider range of operational and maintenance knowledge.

## 7. Product Capabilities

| Capability                       | Product-level description                                                                                                 | Detailed documentation                                                |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| Asset identification and context | Maintain asset records and help users relate operational information to the relevant asset.                               | [Business documentation](business/README.md)                          |
| Agent registration               | Associate a Telemetry Agent with a registered Node so the Node can provide monitoring data.                               | [Agent Registration](business/flows/01-agent-registration.md)         |
| Node and Workload monitoring     | Present selected monitoring information for Nodes and Workloads, including monitoring-data completeness where applicable. | [Monitoring and Detection](business/flows/02-monitoring-detection.md) |
| Incident context                 | Record detected conditions and make related asset or workload context available for investigation.                        | [Monitoring and Detection](business/flows/02-monitoring-detection.md) |
| Field support with AR            | Help technicians identify an asset and review relevant information at the work site.                                      | [Business documentation](business/README.md)                          |
| Maintenance support              | Support inspection, controlled operational work, and recording of outcomes.                                               | [Business documentation](business/README.md)                          |
| AI-assisted analysis             | Provide evidence-based analysis or suggestions to assist users. Users remain responsible for decisions and actions.       | To be defined                                                         |

## 8. PoC Operating Context

The PoC uses a small testbed to represent selected infrastructure concepts:

- A laptop or PC represents a **Node**.
- A containerized application and its database represent a **Workload** and its services.
- Monitoring agents and exporters provide the signals available for Node monitoring.
- Application telemetry provides selected service, database, and communication information.
- Physical markers or QR codes support asset identification during field demonstrations.
- The AR interface presents relevant asset and operational context during those demonstrations.

The testbed is limited in physical scale and cannot reproduce every condition found in a production data center. Results should therefore be interpreted as demonstrations of selected monitoring, identification, and maintenance-support scenarios.

## 9. Success Criteria

The PoC will be evaluated by whether it can demonstrate that:

1. Monitoring information can be associated with registered Nodes and Workloads.
2. Users can distinguish an observed condition from a lack of sufficiently recent or complete monitoring data.
3. A detected condition can be reviewed with relevant asset or workload context.
4. A technician can identify an asset and review relevant information through the AR interface.
5. A maintenance activity and its outcome can be recorded against the relevant asset or condition.
6. A user can review and confirm the selected controlled workload action before it is performed.
7. AI assistance can present suggestions grounded in available evidence while leaving operational decisions with the user.
8. The demonstrations clearly state the testbed’s limitations and do not imply coverage beyond the scenarios implemented.

Detailed test scenarios and evaluation procedures may be maintained separately as they are defined.

## 10. Related Documentation

| Topic                    | Documentation                                      |
| ------------------------ | -------------------------------------------------- |
| Product terminology      | [GLOSSARY.md](GLOSSARY.md)                         |
| Business flows and rules | [Business documentation](business/README.md)       |
| System architecture      | [System overview](architecture/system-overview.md) |
| Component boundaries     | [Context map](architecture/context-map.md)         |
| Component integrations   | [Integration](architecture/integration.md)         |
| API and event contracts  | [Contracts](contracts/README.md)                   |
| Architecture decisions   | [Decisions](decisions/README.md)                   |

## 11. Open Questions

- Which specific Node signals and Workload signals will be included in the final PoC demonstrations?
- Which maintenance scenarios will be selected for evaluation?
- What evidence and user confirmation are required before the controlled workload action is performed?
- Which AI-assisted suggestions will be demonstrated, and what evidence will each suggestion use?
- What measurable evaluation targets, if any, should be set for the PoC?
