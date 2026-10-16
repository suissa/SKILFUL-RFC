# SKILFUL-RFC: OoC — Technology Specification

**Status:** Draft  
**Version:** 0.1.0  
**Concept:** [01-ooc-skilful.concept.md](./01-ooc-skilful.concept.md)  
**Source RFC:** [RFC-WF-0007](https://github.com/suissa/WhatsApp-first-RFCs/blob/main/RFC-WF-0007.md)

## 1. Purpose

This document fixes the implementation technologies for the OoC example. Unlike the concept document, these choices are implementation-specific. They MUST NOT be interpreted as requirements imposed on every OoC implementation.

## 2. Required technology stack

| Responsibility | Required technology | Version policy |
|---|---|---|
| Runtime | Node.js | Node.js 22 LTS, latest available patch in CI and deployment |
| Language | TypeScript | TypeScript 5.x; pin exact resolved version in the lockfile |
| HTTP server | Fastify | Fastify 5.x; pin exact resolved version in the lockfile |
| Request/response validation | Zod | Zod 4.x; pin exact resolved version in the lockfile |
| Persistence for OoC query metadata and local correlation records | PostgreSQL | PostgreSQL 17.x |
| Database access | `pg` Node.js driver | Pin exact resolved version in the lockfile |
| Telemetry instrumentation | OpenTelemetry JS SDK and API | Compatible stable releases, pinned in the lockfile |
| Metrics endpoint | `prom-client` | Stable release compatible with Node.js 22 |
| WhatsApp transport/service | **evolution-go** | Use the deployed project's documented API; do not assume undocumented endpoints or payload shapes |
| Automated tests | Vitest | Stable release compatible with the selected TypeScript version |
| HTTP integration tests | Fastify's `inject()` plus a controlled evolution-go test double | Must not send messages to real recipients in automated tests |
| Package manager | pnpm | Pin the exact version in `package.json` via `packageManager` |

All resolved versions MUST be committed through the package manager lockfile. Do not use floating versions in production deployment definitions.

## 3. Responsibility map

### 3.1 Node.js + TypeScript

The service MUST use TypeScript for application code and Node.js 22 LTS as the runtime. Enable strict TypeScript checks. External input must enter through validated boundaries rather than being trusted because its type is declared in TypeScript.

### 3.2 Fastify

Fastify exposes the HTTP API for health checks and OoC queries. Route handlers MUST remain thin: validate input, establish actor/tenant context, invoke an application use case, and serialize the result. Domain decisions and authorization rules MUST not be embedded only in route handlers.

The service MUST provide health/readiness endpoints suitable for deployment probes. The exact paths must follow the host repository's existing conventions; if none exist, use `GET /health/live` and `GET /health/ready`.

### 3.3 Zod

Zod MUST validate external HTTP payloads and data received from the evolution-go adapter. Validation errors MUST be converted into safe, structured responses. Raw upstream payloads MUST NOT be returned to users.

### 3.4 PostgreSQL

PostgreSQL stores only the data the OoC service is responsible for owning, such as local correlation metadata, query audit records, and indexes or projections explicitly required by the host architecture. It MUST NOT become a competing authority for command outcomes if the host system already has an authoritative evidence store.

Schema changes MUST be versioned and repeatable. Queries MUST be parameterized. Tenant-scoped records MUST be filtered by the authenticated tenant context, not by a caller-supplied tenant ID alone.

### 3.5 OpenTelemetry and prom-client

OpenTelemetry provides traces and instrumentation for request handling and upstream calls. `prom-client` exposes service metrics. Telemetry MUST avoid recording plaintext secrets, authentication tokens, full phone numbers, message contents, or other sensitive payloads by default.

Metrics SHOULD include request counts, latency, error counts, upstream availability, and query outcomes. Trace attributes MUST use stable opaque identifiers where correlation is required and policy permits them.

This stack does not require a particular hosted observability backend. The host project may export telemetry to its existing infrastructure.

### 3.6 evolution-go adapter

The WhatsApp transport integration MUST be isolated behind an adapter (for example, `EvolutionGoClient`). The adapter is responsible for sending a response through the configured evolution-go service and handling upstream timeout, authentication, and error mapping.

The implementation MUST inspect the actual evolution-go repository or deployment documentation before selecting endpoint paths, authentication headers, request fields, and response schemas. These details MUST be configurable where appropriate and validated by integration tests. Do not invent an API contract.

OoC query execution MUST be read-only. Sending the answer back to WhatsApp is a delivery operation; it must not execute the operation being observed.

### 3.7 Vitest

Vitest MUST cover unit tests for intent/query canonicalization, authorization decisions, redaction, evidence formatting, and error mapping. Integration tests MUST exercise Fastify routes and a controlled evolution-go adapter test double. Tests MUST NOT require production credentials or contact real WhatsApp users.

## 4. Logical component architecture

The implementation SHOULD separate these components:

1. **Fastify transport:** HTTP routes and request lifecycle.
2. **Identity/context adapter:** obtains authenticated actor and tenant context from the host system.
3. **OoC query interpreter:** maps natural-language or canonical query input to one of the six read-only capabilities.
4. **Authorization policy:** checks actor, tenant, scope, and requested detail level before retrieval.
5. **Evidence/query ports:** interfaces to the authoritative health, command, error, trace, and evidence sources.
6. **Evidence evaluator:** validates completeness and freshness, marks partial or conflicting results, and normalizes status.
7. **Redaction layer:** filters protected fields before any response formatting.
8. **Response presenter:** produces summary-first responses and numbered drill-down choices.
9. **EvolutionGoClient:** sends approved response text to the configured WhatsApp service.
10. **Audit/telemetry:** records access metadata and operational measurements without leaking sensitive content.

The repository's established architecture takes precedence if it provides equivalent boundaries. Do not duplicate an existing identity, evidence, authorization, or messaging adapter.

## 5. API and integration contracts

The public HTTP contract MUST expose the OoC capabilities required by the concept. The exact route naming should follow the host project's conventions. If the repository has no established convention, a proposed REST mapping is:

| Capability | Proposed route | Method | Behavior |
|---|---|---|---|
| Health summary | `/v1/ooc/status` | GET | Read-only |
| Recent operations | `/v1/ooc/last` | GET | Read-only; actor/tenant scope required |
| Command lifecycle | `/v1/ooc/commands/:commandId` | GET | Read-only |
| Command error | `/v1/ooc/commands/:commandId/error` | GET | Read-only |
| Command evidence | `/v1/ooc/commands/:commandId/evidence` | GET | Read-only; authorization applies to each detail level |
| Correlated trace | `/v1/ooc/traces/:correlationId` | GET | Read-only |

These paths are **local implementation proposals**, not routes mandated by RFC-WF-0007. They MUST be reconciled with the target project's existing API conventions before coding.

Natural-language input MAY be accepted through an existing conversational command endpoint. If no such endpoint exists and one is needed for the example, add a documented endpoint that maps text to canonical queries; do not create a second, conflicting command interpreter.

The application layer MUST depend on interfaces for evidence retrieval. Adapters to existing host-system sources MUST be implemented only after their real contracts are inspected.

## 6. Configuration

Use environment-based configuration validated at startup. At minimum, define configuration for:

- service port and host;
- PostgreSQL connection URL;
- evolution-go base URL and authentication secret/reference;
- upstream request timeout;
- telemetry service name and exporter settings;
- allowed origins or network exposure policy where applicable;
- query rate limits;
- environment name.

Use a documented `.env.example` containing placeholders only. Real credentials MUST NOT be committed. Production secrets MUST be injected by the deployment environment or its approved secret manager. Do not log the configuration object.

The actual evolution-go configuration names and authentication scheme MUST be confirmed from its deployment or repository documentation before implementation.

## 7. Error model

Use stable internal error categories, at minimum:

- `INVALID_QUERY`
- `AMBIGUOUS_TARGET`
- `NOT_FOUND`
- `FORBIDDEN`
- `UPSTREAM_UNAVAILABLE`
- `UPSTREAM_TIMEOUT`
- `EVIDENCE_INCOMPLETE`
- `EVIDENCE_CONFLICT`
- `RATE_LIMITED`
- `INTERNAL_ERROR`

Public messages MUST be safe and must not expose stack traces, credentials, SQL, upstream authentication details, or protected topology. Preserve a correlation identifier where available.

## 8. Security constraints

- Validate all HTTP inputs and upstream responses.
- Enforce authentication and authorization using the host system's established mechanism.
- Never trust caller-supplied tenant identifiers without verifying them against authenticated context.
- Redact secrets and personal data before logs, traces, and responses.
- Use parameterized SQL.
- Apply rate limits to query endpoints and drill-down operations.
- Set bounded timeouts for database and upstream calls.
- Keep raw evidence behind a distinct permission check.
- Ensure the WhatsApp transport cannot be used to bypass OoC authorization.
- Do not allow a read-only query to trigger a mutation or recovery action.

## 9. Compatibility and dependency policy

The implementation MUST reuse existing host-project infrastructure and dependencies where they already satisfy the required contract. Do not introduce a second database, telemetry stack, or WhatsApp client without a documented need.

If the repository's current Node.js or Fastify version conflicts with this document, stop before making broad dependency upgrades. Record the conflict and make the smallest safe change consistent with the project's declared compatibility policy.

## 10. Verification requirements

The chosen stack is considered correctly integrated only when:

- TypeScript strict checks pass;
- all six query capabilities have application-level coverage;
- Fastify route tests verify validation and safe responses;
- authorization and tenant-isolation tests pass;
- redaction tests prove sensitive values are absent from responses and captured telemetry;
- the evolution-go adapter is tested against a controlled test double and, where credentials and a safe test environment exist, a documented non-production integration;
- timeout and upstream failure behavior are tested;
- telemetry correlation works without recording sensitive payloads;
- all package versions are locked and reproducible.

## 11. Traceability

| Technology decision | Concept requirement served |
|---|---|
| Fastify + TypeScript | HTTP boundary and explicit application contracts |
| Zod | Input validation and safe upstream data handling |
| PostgreSQL | Durable local audit/correlation metadata where needed |
| OpenTelemetry + prom-client | Correlation and service health measurements |
| evolution-go adapter | Conversational delivery through WhatsApp |
| Vitest + Fastify inject | Verifiable acceptance, isolation, and failure behavior |
