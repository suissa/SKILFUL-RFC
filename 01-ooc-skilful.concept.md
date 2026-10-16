# SKILFUL-RFC: Observability over Conversation (OoC)

**Status:** Draft  
**Version:** 0.1.0  
**Source RFC:** [RFC-WF-0007 — Observability over Conversation](https://github.com/suissa/WhatsApp-first-RFCs/blob/main/RFC-WF-0007.md)  
**Source status/version:** Draft Standard, v1.0.0, 22 Feb 2026  
**Category:** Standards Track / Conversational Operations  
**Dependencies:** RFC-WF-0001 (WFCS), RFC-WF-0003 (CCP), RFC-WF-0005 (CRCD), RFC-WF-0006 (EAS)

## 1. Abstract

Observability over Conversation (OoC) makes operational state, command progress, failures, authorization decisions, and evidence accessible through conversation. The user must be able to understand what happened and what can be done next without depending on a dashboard as the only operational interface.

This SKILFUL concept preserves RFC-WF-0007's normative capabilities and adds agent-oriented decision semantics. An agent must identify the operational question, resolve the relevant command or correlation context, retrieve only authorized evidence, summarize it accurately, and offer safe follow-up actions. It must distinguish observed facts from inference and must never invent operational status.

This document defines behavior and acceptance conditions only. Concrete technologies and repository execution procedures are defined in the companion technology and instructions documents.

## 2. Problem and motivation

Conversational systems can execute operations but become difficult to operate if users cannot inspect progress, failures, decisions, and outcomes through the same conversational interface. OoC closes that gap by making operational observability conversational, evidence-backed, permission-aware, and progressively detailed.

## 3. Scope

The capability covers:

- system health and service status;
- recent operations and command lifecycle;
- error reason, retryability, and recovery guidance;
- explanations of authorization or policy decisions;
- evidence summaries and authorized detail retrieval;
- correlation across commands, traces, services, and outcomes;
- natural-language requests mapped to canonical read-only observability commands;
- concise responses with stable identifiers and progressive drill-down.

### Non-goals

OoC does not prescribe a particular telemetry vendor, database, web framework, messaging provider, dashboard, or deployment platform. It does not authorize a user to perform an operation merely because that user can observe it. It does not expose unrestricted raw logs or internal service topology to ordinary actors.

## 4. Normative language

**MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are normative as defined by RFC-WF-0007.

This document labels requirements as:

- **Source requirement:** directly required by RFC-WF-0007.
- **Derived requirement:** needed to make a source requirement reliably implementable by an agent.
- **Local extension:** added implementation behavior not mandated by the source RFC.

## 5. Definitions

- **OoC query:** a conversational request for operational state or evidence.
- **Operational actor:** a user or agent permitted to inspect a defined portion of operational information.
- **Evidence-backed response:** an answer grounded in retrievable operational evidence.
- **Drill-down:** progressive disclosure from a short summary to additional authorized detail.
- **Command identifier:** a stable identifier for a submitted command or operation.
- **Correlation identifier:** a stable identifier linking related work across services.
- **Canonical query:** a normalized read-only command representing an observability request.
- **Unverified hypothesis:** an interpretation that has not been confirmed by available evidence.

## 6. Desired outcomes and invariants

1. **Dashboard independence — Source requirement.** Essential operational visibility MUST be available through conversation.
2. **Evidence-backed answers — Source requirement.** Answers about command execution MUST be grounded in available evidence and reference stable identifiers.
3. **Progressive detail — Source requirement.** Responses MUST summarize first and provide numbered follow-up choices when more detail is available.
4. **Least privilege — Source requirement.** The actor MUST only receive data allowed by their permissions and tenant boundary.
5. **Correlation — Source requirement.** The system MUST correlate queries and outcomes using command ID, correlation ID, or conversation/message identifiers.
6. **Truthfulness — Derived requirement.** The agent MUST NOT report an operation as successful, failed, running, or recovered unless the retrieved evidence supports that status.
7. **Read-only observability — Source requirement.** Canonical OoC queries MUST be read-only. A suggested recovery operation is a separate command and must pass its own authorization and confirmation rules.
8. **Redaction by default — Source requirement.** Secrets, tokens, personal data, and sensitive payment details MUST be redacted unless a specific authorized policy permits disclosure.
9. **Tenant isolation — Source requirement.** An actor MUST NOT retrieve another tenant's operational data.
10. **No fabricated evidence — Derived requirement.** Missing evidence must be reported as unavailable, not reconstructed as fact.

## 7. Actors and responsibilities

- **User / operational actor:** asks about system status, a command, an error, a decision, or evidence.
- **Conversational agent:** interprets the question, determines missing identifiers, selects a canonical query, presents grounded results, and requests clarification where necessary.
- **Authorization authority:** decides whether the actor may access the requested level of detail.
- **Observability capability:** retrieves health, command, error, trace, and evidence data.
- **Evidence authority:** provides the authoritative operational records used to support the response.
- **Recovery capability:** may expose a separately authorized retry or remediation operation; it is not part of the read-only query itself.

Implementations MAY combine responsibilities internally, but MUST preserve their security and behavioral boundaries.

## 8. Canonical query capabilities

An implementation MUST support the minimum query set from RFC-WF-0007:

- `OOC.STATUS` — overall health, impacted services, incident identifier when present, and last update time.
- `OOC.LAST` — the actor's or tenant's recent operations, within the permitted scope.
- `OOC.CMD <command_id>` — lifecycle, last action, stage, outcome, and last error when present.
- `OOC.ERROR <command_id>` — error details, reason code, human explanation, and retryability when available.
- `OOC.EVIDENCE <command_id>` — evidence summary and authorized drill-down.
- `OOC.TRACE <correlation_id>` — a summarized correlated chain across services.

Natural-language equivalents MUST be accepted and canonicalized to the appropriate read-only capability. The exact wording is not fixed.

## 9. Agent decision semantics

### 9.1 Intent interpretation

The agent SHOULD infer the likely observability intent from the user's wording and context. Examples include asking whether the system is healthy, what happened to an order, why a command was rejected, or whether an operation can be retried.

The agent MUST use the narrowest canonical query that answers the request. It MUST NOT fetch raw payloads by default when a summary is sufficient.

### 9.2 Progressive specification

If the user asks “What happened to my command?” but provides no identifier, the agent SHOULD search only within the actor's authorized recent-operation scope if that can safely identify a unique candidate. If multiple candidates remain, it MUST ask a focused clarification or present a short authorized list. It MUST NOT silently choose an ambiguous operation.

### 9.3 Evidence and uncertainty

The agent MUST separate:

- **Observed:** directly returned by authoritative evidence.
- **Derived:** computed from observed values using an explainable rule.
- **Unknown:** unavailable or not established by evidence.

If the evidence source is unavailable, stale, or contradictory, the agent MUST state the limitation and MUST NOT invent a status. If records conflict, the agent SHOULD expose the conflict and refer to the authoritative source policy.

### 9.4 Authorization and confirmation

The agent MUST verify access before retrieving or revealing protected detail. It MUST apply redaction before composing the user-facing response. A request for more detail does not itself grant additional permission.

The agent MUST NOT retry, cancel, compensate, or otherwise mutate an operation as a side effect of an OoC query. If recovery is available, the agent MAY offer a separately authorized action and MUST follow the applicable confirmation policy.

### 9.5 Response shape

Every response about an operation SHOULD include, when available:

1. concise outcome summary;
2. current status and last known lifecycle stage;
3. stable command or correlation identifier;
4. relevant error or policy reason;
5. next safe actions or numbered drill-down choices.

The agent MUST avoid raw-log dumps unless explicitly requested and authorized. Sensitive fields MUST be redacted before output.

## 10. State model

An OoC request follows this logical lifecycle:

1. **Received:** conversational request is accepted.
2. **Interpreted:** likely query intent and identifiers are extracted.
3. **Disambiguated:** required context is confirmed or safely resolved.
4. **Authorized:** actor, tenant, scope, and detail level are checked.
5. **Retrieved:** read-only evidence is queried.
6. **Evaluated:** evidence is checked for completeness, freshness, and conflicts.
7. **Presented:** a concise, redacted, evidence-backed response is returned.
8. **Drilled down (optional):** the user selects a numbered detail option and the system rechecks authorization for that request.

Implementations MAY use different internal states, but externally observable behavior MUST preserve these safety boundaries.

## 11. Failure and recovery semantics

- If no matching operation exists, the agent MUST say that no matching record was found within the searched authorized scope.
- If an identifier is malformed, the agent SHOULD explain the expected identifier form without revealing internal secrets.
- If a query times out or an evidence source is unavailable, the agent MUST report that the status could not be verified.
- If access is denied, the agent MUST explain the denial at a level allowed by policy and MUST NOT reveal protected policy internals.
- If only partial evidence is available, the agent MUST label the response as partial.
- Retry guidance MUST be evidence-based. The agent MUST NOT promise that a retry is safe unless the operation's retry policy supports it.

## 12. Security and privacy

The implementation MUST enforce least privilege, tenant isolation, default redaction, and auditability of observability access. It MUST NOT expose plaintext credentials, integration secrets, signing material, or internal keys. Full payment identifiers and personal data MUST remain hidden unless a policy explicitly permits disclosure.

Repeated queries SHOULD be rate-limited. Suspicious enumeration patterns SHOULD trigger reduced detail or additional authorization checks. Access to raw artifacts SHOULD require stronger authorization where the governing policy requires it.

## 13. Acceptance criteria

An implementation conforms to this concept when:

- [ ] All six canonical query capabilities are available.
- [ ] Natural-language queries map to read-only canonical commands.
- [ ] Status, lifecycle, errors, and evidence summaries are grounded in authoritative records.
- [ ] Responses contain concise summaries and stable identifiers when available.
- [ ] Missing or ambiguous identifiers trigger safe narrowing or clarification.
- [ ] Unauthorized details and cross-tenant data are not disclosed.
- [ ] Sensitive data is redacted by default.
- [ ] Raw payloads are not returned by default.
- [ ] Query failures and missing evidence are reported honestly.
- [ ] Recovery suggestions do not execute mutations implicitly.
- [ ] Correlation across commands and services is summarized without exposing protected topology.
- [ ] Tests provide evidence for authorization, redaction, ambiguity, error handling, and successful queries.

## 14. Source traceability

| Concept requirement | RFC-WF-0007 source |
|---|---|
| Conversational operational visibility | Sections I, V |
| Minimum query set | Section VII.A |
| Natural-language canonicalization | Section VII.A |
| Summary-first and drill-down | Sections V, VII.B–C |
| Evidence-backed response | Section VIII |
| Correlation identifiers and cross-service summaries | Section IX |
| Least privilege and redaction | Section X |
| Rate limiting and abuse resistance | Section XI |
| Compliance profiles | Section XII |
| Security and tenant isolation | Section XIV |

## 15. Assumptions and open questions

- The source system's authoritative evidence schema and actual available query APIs must be inspected before implementation.
- Authentication and authorization integration must use the host project's existing identity and scope model where one exists.
- The implementation must not assume an endpoint or payload shape for an external service until its documentation or repository code confirms it.
- This concept does not mandate a dashboard or a specific telemetry vendor.
