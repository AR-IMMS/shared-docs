# AR-IMMS Shared Documentation

This repository contains documentation shared across AR-IMMS contributors and
repositories. Repository-specific implementation details remain with their
owning codebase.

## Product and architecture

- [AR-IMMS Product](PRODUCT.md) — product purpose, users, scope, and capabilities.
- [Telemetry Agent Architecture Overview](architecture/telemetry-agent-overview.md)
  — Agent/Collector boundaries, data flow, current implementation status, and
  Core integration limits.

## Business workflows

Business processes and terminology are documented in `business/` and
`GLOSSARY.md`. Technical implementation details belong in the owning
repository's architecture and runbook documentation.

## Engineering guidance

- [AI-Assisted Development Policy](engineering/AI_CODING_POLICY.md)

## Frontend guidance and API contracts

- [Frontend implementation guide](guides/frontend/frontend-implementation-guide.md)
  — shared design and coding principles.
- [IAM frontend guide](guides/frontend/iam-frontend-implementation-guide.md)
  and [walkthrough](guides/frontend/iam-frontend-walkthrough.md) — IAM-specific
  additions to the shared guide.
- [Platform HTTP response guide](guides/frontend/platform-http-response-guide.md)
  — frontend handling of Core response envelopes and errors.
- OpenAPI contracts and reusable HTTP schemas live in `contracts/openapi/`;
  keep this directory for contract files, not narrative guides.
