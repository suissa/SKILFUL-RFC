# SKILFUL-RFC — Skill-Faithful RFC Format

**Status:** Draft Standard  
**Version:** 0.1.0  
**Category:** Agent Engineering / Specification Format  
**Purpose:** Convert conventional RFCs into implementation-ready agent instructions without mixing requirements, technology choices, and project execution procedures.

## Abstract

SKILFUL-RFC is a document format for translating a conventional Request for Comments (RFC), product specification, or protocol proposal into a set of artifacts that agents can use to implement and integrate a capability into a project.

It preserves the source RFC's normative requirements and rationale while adding agent-oriented semantics: what outcome an agent must achieve, how it should reason about incomplete information, what evidence it must produce, which decisions require confirmation, and how completion is verified. It deliberately separates those behavioral requirements from technology selection and repository-specific execution instructions.

A SKILFUL-RFC package consists of three canonical documents:

1. `01-{name}-skilful.concept.md` — technology-independent behavior and requirements.
2. `01-{name}-skilful.technology.md` — explicit technology choices, versions, integration boundaries, and technical constraints.
3. `01-{name}-skilful.instructions.md` — agent execution plan, repository workflow, tests, Git rules, and completion criteria.

The source RFC remains the normative source for the requirements it defines. A SKILFUL-RFC package must not silently change, weaken, or invent source requirements.

## 1. Design principles

### 1.1 Source fidelity

Every source requirement adopted by the package MUST be traceable to a source section or explicitly labeled as an implementation decision, derived requirement, or local extension. The package MUST preserve normative strength: `MUST` must not become `SHOULD`, and `MUST NOT` must not become optional.

### 1.2 Separation of concerns

The concept document defines **what the capability means and how it must behave**, without prescribing libraries, frameworks, vendors, programming languages, infrastructure products, or deployment platforms.

The technology document defines **which technologies must be used** and how they are assigned to responsibilities. It MUST NOT redefine domain semantics to fit a library.

The instructions document defines **how an agent performs the work in a particular repository**: inspection, implementation order, validation, integration, Git operations, and delivery evidence.

### 1.3 Agent-oriented semantics

SKILFUL-RFC adds instructions that conventional RFCs often leave implicit for autonomous or semi-autonomous implementers. The concept document SHOULD specify:

- the intended outcome and observable acceptance conditions;
- actors, capabilities, inputs, outputs, state transitions, and invariants;
- behavior when context or required fields are missing;
- uncertainty, ambiguity, confirmation, and authorization boundaries;
- failure handling, retries, idempotency, and recovery expectations where relevant;
- evidence that must be produced to prove completion;
- prohibited behavior and scope boundaries.

These semantics constrain outcomes and decisions, not the implementation technology.

### 1.4 Explicit uncertainty

An agent MUST distinguish facts confirmed by the repository or source specification from assumptions and proposed choices. It MUST NOT fabricate APIs, configuration names, endpoints, test results, credentials, or successful integration outcomes. Unresolved decisions must be recorded and handled according to their impact.

### 1.5 Verifiable completion

A task is not complete merely because code or documentation was written. Completion MUST be tied to observable evidence, such as passing tests, verified integration behavior, updated documentation, or an explicitly recorded blocker.

## 2. Canonical package

Each capability uses the same `{name}` slug across the three files. Use lowercase kebab-case and keep the slug stable.

### 2.1 Concept file

Filename: `01-{name}-skilful.concept.md`

Required sections:

1. Metadata: status, version, date, source RFC, and dependencies.
2. Abstract.
3. Problem and motivation.
4. Scope and non-goals.
5. Terminology.
6. Desired outcomes and invariants.
7. Actors and responsibilities.
8. Inputs, outputs, and state model.
9. Normative behavior (`MUST`, `MUST NOT`, `SHOULD`, `SHOULD NOT`, `MAY`).
10. Agent decision semantics: ambiguity, missing information, confidence, authorization, and escalation.
11. Failure and recovery semantics.
12. Security and privacy requirements.
13. Acceptance criteria and evidence.
14. Traceability to source RFC sections.
15. Open questions and explicit assumptions.

This document MUST remain technology-independent. Product or technology names may appear only when they are part of the source requirement or a necessary external system boundary; implementation choices belong in the technology document.

### 2.2 Technology file

Filename: `01-{name}-skilful.technology.md`

Required sections:

1. Metadata and linked concept document.
2. Required technology stack with exact versions or version policy.
3. Responsibility map for each technology.
4. Architecture and component boundaries.
5. Data model and persistence decisions.
6. API, event, and integration contracts.
7. Configuration and secret handling.
8. Observability and operational requirements.
9. Compatibility and versioning.
10. Dependency restrictions and alternatives explicitly rejected.
11. Verification requirements for the selected stack.
12. Traceability from each technical choice to a concept requirement.

This document MUST make technology choices explicit. Avoid vague phrases such as “use a suitable database” when a specific technology is required. If a version cannot be confirmed, state the allowed version range and how the agent must verify the actual version before implementation.

### 2.3 Instructions file

Filename: `01-{name}-skilful.instructions.md`

Required sections:

1. Metadata and linked concept/technology documents.
2. Agent role and objective.
3. Repository inspection checklist.
4. Preconditions and safety boundaries.
5. Ordered implementation workflow.
6. Integration steps.
7. Test plan and required commands.
8. Git and branch/commit/PR rules.
9. Secrets, generated files, and destructive-operation rules.
10. Failure, blocker, and escalation behavior.
11. Definition of done.
12. Required final report and evidence.

Instructions MUST be executable by an agent working in a real repository. They MUST not assume files, scripts, commands, branches, or endpoints exist until verified. If a prescribed command is unavailable, the agent must inspect the project, select a justified equivalent only when permitted, and document the deviation.

## 3. Normative language

The terms **MUST**, **MUST NOT**, **REQUIRED**, **SHOULD**, **SHOULD NOT**, and **MAY** are to be interpreted as normative requirements.

- **MUST / REQUIRED:** absolute requirement for a conforming package or implementation.
- **MUST NOT:** prohibited behavior.
- **SHOULD / SHOULD NOT:** recommended behavior that may be overridden only for a documented, reasoned case.
- **MAY:** optional behavior.

An agent MUST preserve these distinctions when translating the package into a SKILL or another agent instruction format.

## 4. Translation into a SKILL

A SKILFUL-RFC package is designed to be translated into an agent SKILL with minimal interpretation.

A translator SHOULD map:

- the concept abstract and trigger conditions to the SKILL description;
- the desired outcome and scope to the SKILL purpose;
- normative behavior and decision semantics to the SKILL instructions;
- acceptance criteria and evidence requirements to validation steps;
- the technology document to implementation-specific guidance or references;
- the instructions document to repository workflow, commands, and integration procedures.

The generated SKILL MUST retain links to the source RFC and all three SKILFUL-RFC documents. It MUST NOT merge technology choices into universal behavior instructions if that would make the skill misleading for other stacks.

If the target SKILL format supports progressive disclosure, keep the core instructions concise and reference detailed technology or workflow files as supporting resources. Translation MUST be deterministic enough that a reviewer can trace each critical SKILL instruction back to a source requirement or an explicitly identified local decision.

## 5. Conformance

A package conforms to SKILFUL-RFC when:

1. all three canonical files exist and use the same name slug;
2. the concept document is technology-independent and contains agent decision semantics;
3. the technology document specifies concrete choices and their responsibilities;
4. the instructions document defines a safe, testable implementation workflow;
5. normative requirements are traceable to the source RFC or labeled as local additions;
6. acceptance criteria are observable and testable;
7. assumptions, unresolved questions, and deviations are disclosed;
8. no document claims tests, deployment, or integration succeeded without evidence.

## 6. Example package

The repository includes a worked translation of [RFC-WF-0007 — Observability over Conversation (OoC)](https://github.com/suissa/WhatsApp-first-RFCs/blob/main/RFC-WF-0007.md):

- `01-ooc-skilful.concept.md`
- `01-ooc-skilful.technology.md`
- `01-ooc-skilful.instructions.md`

The example preserves OoC's conversational observability requirements while choosing Node.js, Fastify, and evolution-go for the implementation-specific layer.

## 7. Source fidelity and extensions

A SKILFUL-RFC package may add agent-focused details needed to make implementation reliable, but MUST label them as one of:

- **Source requirement** — explicitly required by the source RFC.
- **Derived requirement** — necessary to satisfy a source requirement.
- **Implementation decision** — a concrete choice made for this package.
- **Local extension** — an additional capability outside the source RFC's mandatory scope.

This labeling prevents an implementation choice from being mistaken for a universal protocol requirement.

## 8. License and evolution

SKILFUL-RFC is an open specification intended for reuse. Implementations may add domain-specific sections while retaining the three canonical files and their separation of concerns. A future revision SHOULD define machine-readable front matter and automated conformance checks without making those tools prerequisites for reading or authoring a package.
