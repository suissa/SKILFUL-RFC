# SKILFUL-RFC

**SKILFUL-RFC (Skill-Faithful RFC)** is a format for transforming conventional RFCs into specifications that agents can implement and integrate while preserving the source standard's requirements.

It separates domain behavior from technology selection and repository execution instructions, making each package easier to review, implement, and translate into an agent SKILL.

## Canonical files

Every capability uses three files sharing the same `{name}` slug:

1. **`01-{name}-skilful.concept.md`** — technology-independent requirements, behavior, agent decision semantics, safety boundaries, and acceptance criteria.
2. **`01-{name}-skilful.technology.md`** — explicit technologies, version policy, architecture boundaries, APIs, configuration, and integration decisions.
3. **`01-{name}-skilful.instructions.md`** — agent role, repository inspection, implementation sequence, testing, Git rules, safety constraints, and definition of done.

The source RFC remains the normative source for adopted requirements. Additional agent-oriented requirements must be identified as derived requirements, implementation decisions, or local extensions rather than silently attributed to the source.

## Specification

Read [SKILFUL-RFC.md](./SKILFUL-RFC.md) for the format definition, required sections, normative language, conformance criteria, and guidance for translating the package into a SKILL.

## Worked example: RFC-WF-0007

The repository includes a SKILFUL-RFC translation of [RFC-WF-0007 — Observability over Conversation (OoC)](https://github.com/suissa/WhatsApp-first-RFCs/blob/main/RFC-WF-0007.md).

- [OoC concept](./01-ooc-skilful.concept.md) — technology-independent semantics and requirements.
- [OoC technology](./01-ooc-skilful.technology.md) — Node.js 22, TypeScript, Fastify 5, PostgreSQL, OpenTelemetry, and the evolution-go WhatsApp service.
- [OoC instructions](./01-ooc-skilful.instructions.md) — agent implementation, integration, testing, Git, and completion rules.

The technology choices are specific to this worked example; they do not become universal requirements of RFC-WF-0007 or SKILFUL-RFC.

## Status

Draft specification, version 0.1.0. Contributions should preserve the separation between concept, technology, and execution instructions, and keep each requirement traceable to its source or explicitly labeled as an extension.
