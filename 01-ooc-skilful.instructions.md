# SKILFUL-RFC: OoC — Agent Implementation Instructions

**Status:** Draft  
**Version:** 0.1.0  
**Concept:** [01-ooc-skilful.concept.md](./01-ooc-skilful.concept.md)  
**Technology:** [01-ooc-skilful.technology.md](./01-ooc-skilful.technology.md)  
**Source RFC:** [RFC-WF-0007](https://github.com/suissa/WhatsApp-first-RFCs/blob/main/RFC-WF-0007.md)

## 1. Agent objective

Implement and integrate Observability over Conversation (OoC) into the target project while preserving RFC-WF-0007 requirements, the host project's architecture, and the selected stack: Node.js 22 LTS, TypeScript, Fastify 5, PostgreSQL 17 where persistence is required, OpenTelemetry, and evolution-go as the WhatsApp service.

The agent's goal is a verified, maintainable implementation, not merely code generation. The agent MUST report actual evidence and MUST NOT claim a test, integration, commit, push, or deployment succeeded unless the operation was performed and its result observed.

## 2. Repository inspection before editing

Before modifying files, the agent MUST:

1. Confirm the repository and current working directory.
2. Read `README`, contribution guidelines, existing `AGENTS.md`, `INSTRUCTIONS.md`, package manifests, lockfiles, and relevant architecture documentation.
3. Inspect the current branch, working-tree status, recent history, and any open work relevant to OoC.
4. Identify existing HTTP routes, authentication/authorization middleware, tenant context, evidence source, command registry, telemetry, persistence, and WhatsApp integration.
5. Determine the project's actual Node.js, TypeScript, Fastify, package-manager, and database versions.
6. Inspect the real evolution-go integration contract from the project or its authoritative documentation. Verify endpoint paths, authentication, payload schema, delivery semantics, and error behavior. Do not guess.
7. Map each RFC-WF-0007 requirement to an existing capability, a gap, or a proposed change.
8. Record incompatible requirements, missing contracts, and assumptions before making architectural changes.

Do not replace an existing subsystem simply because the preferred stack names another tool. Reuse compatible project components and document any deviation from the technology specification.

## 3. Safety boundaries

The agent MUST:

- preserve unrelated user changes and never discard a dirty working tree;
- avoid destructive commands such as hard resets, forced cleanups, or broad file deletion unless explicitly authorized and demonstrably necessary;
- never commit credentials, tokens, private phone numbers, message contents, production data, or generated secrets;
- never send real WhatsApp messages from unit or automated integration tests;
- never use production data for tests unless the project policy explicitly permits it and data is protected;
- never expose raw logs or evidence before authorization and redaction;
- never add a mutation to an OoC read-only query;
- never fabricate external API routes, configuration keys, database schemas, test output, or evidence;
- ask for clarification or report a blocker when a required external contract cannot be verified.

## 4. Implementation workflow

### Step 1 — Build a traceability map

Create or update a checklist linking every source requirement to the relevant code, test, or explicit blocker. At minimum cover the six canonical query capabilities, natural-language canonicalization, evidence-backed responses, correlation, drill-down, access control, redaction, rate limiting, and failure behavior.

### Step 2 — Define application contracts

Before implementing route handlers, define typed contracts for:

- authenticated actor and tenant context;
- canonical OoC query and its parameters;
- health, command, error, evidence, and trace results;
- evidence freshness/completeness and conflict status;
- authorization decisions;
- redaction policy;
- structured user-facing response and drill-down options.

Represent unknown or unavailable information explicitly. Do not encode “unknown” as a success status.

### Step 3 — Integrate authoritative data sources

Inspect the host project to locate the authoritative source for each kind of evidence. Implement adapters against verified contracts. Do not create a competing source of truth for command status or outcomes.

If a required source does not exist, add a clearly bounded interface and report the missing integration as a blocker or staged capability. Do not return fabricated sample data from production routes.

### Step 4 — Implement query interpretation

Map the canonical commands `OOC.STATUS`, `OOC.LAST`, `OOC.CMD`, `OOC.ERROR`, `OOC.EVIDENCE`, and `OOC.TRACE` to read-only application use cases.

Natural-language input MUST be normalized into one of these query types or rejected/clarified safely. If multiple targets are plausible, narrow the search within the authenticated actor's permitted scope or ask the user to choose. Never pick an arbitrary command when ambiguity could reveal or misrepresent operational data.

### Step 5 — Enforce authorization before disclosure

Verify actor, tenant, and scope before querying protected records where feasible, and always before returning protected results. Apply authorization to each drill-down level. Do not trust a tenant ID supplied by the request without matching it to authenticated context.

Redaction must happen before response formatting and before sensitive data can enter logs or traces. Test the absence of secrets and personal data, not just the presence of expected fields.

### Step 6 — Implement response presentation

Responses MUST be concise and summary-first. Include stable identifiers, relevant status, last known lifecycle stage, and a safe next action when available. Use numbered drill-down options when more authorized detail exists.

Distinguish observed, derived, and unknown information. Do not present predictions, guesses, or incomplete evidence as confirmed facts. Do not return raw logs unless the request is explicit and the actor is authorized.

### Step 7 — Integrate evolution-go

Implement or reuse an isolated adapter for the evolution-go service. Verify the actual API contract before coding. Configure the service URL, authentication, and timeouts through validated configuration.

The adapter should map upstream failures into stable internal error categories without exposing credentials or raw upstream payloads. Use a controlled test double in automated tests. Real WhatsApp delivery may be tested only in an explicitly designated non-production environment with an approved test recipient and valid credentials.

### Step 8 — Add persistence only where required

Use PostgreSQL 17 for local records the service owns, such as access audit metadata or correlation indexes, only when the host architecture requires persistence. Add versioned migrations and tenant-aware access patterns. Do not duplicate authoritative command/evidence data unnecessarily.

### Step 9 — Add telemetry and operational protection

Instrument request handling and upstream calls. Add request limits, bounded timeouts, and safe structured errors. Telemetry must exclude secrets, message contents, full phone numbers, and protected evidence by default. Verify that correlation IDs are useful without exposing sensitive internal topology.

### Step 10 — Document operations

Document configuration variables, local startup, migration commands if applicable, tests, health/readiness probes, evolution-go setup, and limitations. Provide an `.env.example` with placeholders only. Keep instructions consistent with the actual scripts and deployment manifests.

## 5. Required test plan

Adapt exact commands to the repository's scripts after inspecting its package manifest. Do not assume a script exists.

### Unit tests

- natural-language/canonical query mapping;
- missing and ambiguous identifiers;
- status normalization and evidence evaluation;
- authorization and scope decisions;
- tenant isolation;
- redaction of secrets and personal data;
- response summary and drill-down generation;
- upstream error mapping and retryability;
- rate-limit policy.

### HTTP/API integration tests

Use Fastify's injection facilities or the repository's established test harness to verify:

- each of the six canonical query capabilities;
- input validation and stable error responses;
- missing, malformed, and unknown identifiers;
- authorization failures;
- no cross-tenant data leakage;
- redaction before serialization;
- evidence-source timeout and partial results;
- rate-limited requests;
- no mutation side effects from read-only queries.

### evolution-go integration tests

- adapter request construction follows the verified upstream contract;
- authentication is supplied without appearing in logs;
- timeout and upstream error paths are handled;
- the test double records attempted outbound messages;
- automated tests never contact real recipients.

A live integration test is optional unless the repository's release policy requires it. If performed, it must use a designated non-production instance and a controlled recipient.

### Security and privacy tests

Test that responses, logs, traces, and error messages do not contain test secrets, tokens, message bodies, full phone numbers, or unauthorized raw payloads. Test tenant boundaries and privileged drill-down separately.

### Verification sequence

Run the applicable commands in this order:

1. formatting and lint;
2. TypeScript type-check;
3. unit tests;
4. HTTP and adapter integration tests;
5. security/privacy tests;
6. build;
7. any repository-required end-to-end or load tests.

Report each command and its actual result. If a command cannot run because dependencies, credentials, or infrastructure are unavailable, mark it **not run** and state why; do not mark it as passed.

## 6. Git and collaboration rules

1. Inspect the working tree before creating a branch or editing files.
2. Follow the repository's branch naming and contribution rules. If none exist, use a focused branch such as `feat/ooc-conversational-observability`.
3. Never overwrite or revert unrelated user work.
4. Keep changes scoped to OoC and its tests/documentation.
5. Use descriptive commits that state the change, for example `feat(ooc): add evidence-backed query handlers`.
6. Do not commit until relevant local validation has run or the reason it could not run is recorded.
7. Push only to the intended repository and branch.
8. Open a pull request when repository policy expects review; summarize behavior, tests, known limitations, and the source RFC.
9. Do not merge, release, or deploy unless explicitly authorized and the project's policy permits it.
10. After each Git operation, inspect its result. Never report a commit, push, PR, or merge that was not confirmed.

If using GitHub tools instead of a local checkout, verify the target repository, branch, existing file content, and resulting commit/PR URLs before reporting completion.

## 7. Definition of done

The implementation is complete only when all applicable conditions hold:

- [ ] Every RFC-WF-0007 mandatory capability is implemented or explicitly recorded as blocked with evidence.
- [ ] The six canonical query types are supported.
- [ ] Natural-language requests are mapped safely to read-only queries.
- [ ] Evidence-backed answers include stable identifiers when available.
- [ ] Authorization, tenant isolation, redaction, and rate limiting are enforced and tested.
- [ ] Ambiguous, missing, stale, partial, or conflicting evidence is handled honestly.
- [ ] Drill-down is progressive and rechecks authorization.
- [ ] evolution-go integration follows a verified contract.
- [ ] Telemetry and error handling avoid sensitive data disclosure.
- [ ] Tests and build pass, or every unexecuted check is explicitly disclosed.
- [ ] Configuration and operational documentation match the implementation.
- [ ] Git changes are scoped and the final branch/commit/PR state is verified.

## 8. Required final report

The agent's final report MUST contain:

1. Summary of implemented behavior.
2. Files changed and why.
3. Source RFC requirements covered.
4. Exact validation commands and pass/fail/not-run results.
5. Integration status for the evidence sources and evolution-go.
6. Security/privacy measures and tests.
7. Known limitations, assumptions, and blockers.
8. Branch, commit, and pull-request links where applicable.

The report MUST distinguish completed work from proposed work and verified behavior from unverified assumptions.
