# Project Management Framework

Date: 2026-07-03

## Purpose

This project will be managed as a spec-driven and phase-driven engineering effort. The goal is to keep the agentic-SOAR implementation traceable, testable, and incrementally shippable as the lab evolves from Demo 1 to Demo 2 and Demo 3.

The project is not managed around a single coding-agent tool. Codex is the primary development agent. OpenCode may be used later as an optional secondary tool for comparison, local experimentation, or alternate model workflows.

## Operating Model

Every meaningful feature or system change should move through this sequence:

```text
spec -> plan -> implementation -> tests -> UAT -> evaluation -> review -> decision log
```

Definitions:

- `spec`: describes what must exist and why.
- `plan`: describes how the work will be implemented.
- `implementation`: code, infrastructure, fixtures, UI, or documentation changes.
- `tests`: automated checks for behavior and regressions.
- `UAT`: user acceptance criteria for the workflow being delivered.
- `evaluation`: measurement of SOAR and LLM behavior.
- `review`: bug, security, maintainability, and architecture review.
- `decision log`: ADRs for choices that affect future work.

## Repository Documentation Structure

The documentation structure should evolve toward:

```text
docs/
  PROJECT-MANAGEMENT.md
  architecture/
  adr/
  evals/
  phases/
  specs/
  threat-models/
  superpowers/
    specs/
```

Current foundational specs may live under `docs/superpowers/specs/` because that is the active design workflow location. As implementation grows, durable project specs can also be mirrored or promoted into `docs/specs/`.

## Phase Structure

Each phase should have a dedicated directory:

```text
docs/phases/YYYY-MM-DD-phase-name/
  SPEC.md
  PLAN.md
  UAT.md
  EVAL.md
  REVIEW.md
```

Required phase files:

- `SPEC.md`: scope, requirements, non-goals, interfaces, data contracts, risks.
- `PLAN.md`: implementation steps, dependencies, sequencing, rollback notes.
- `UAT.md`: acceptance criteria written from the operator or developer perspective.
- `EVAL.md`: measurable evaluation criteria, especially for deterministic-vs-LLM analysis.
- `REVIEW.md`: post-implementation findings, residual risks, and follow-up work.

## ADRs

Architectural decisions that affect future work must be recorded in `docs/adr/`.

ADR filename format:

```text
YYYY-MM-DD-short-decision-title.md
```

ADR content:

```text
# Title

Date:
Status: proposed | accepted | superseded

## Context
## Decision
## Consequences
## Alternatives Considered
```

Initial ADR candidates:

- Use PostgreSQL for Demo 1 persistence.
- Use a canonical internal schema inspired by ECS/OCSF.
- Use local LLM inference instead of paid external APIs.
- Use Ollama as the first local runtime adapter.
- Keep Demo 1 recommend-only.
- Compare deterministic SOAR analysis against LLM agentic analysis.

## Evaluation Discipline

Because this is an agentic security project, every phase that touches detection, triage, investigation, recommendations, or LLM behavior must include evaluation.

Minimum evaluation dimensions:

- Correct scenario detection.
- Correct affected entities.
- Evidence quality.
- Severity quality.
- Recommendation quality.
- Deterministic baseline result.
- LLM result.
- Agreement or disagreement between engines.
- False positives on normal activity.
- Latency and timeout behavior.
- Audit completeness.

LLM changes must preserve:

- Prompt template versioning.
- Model/runtime metadata.
- Case packet input references.
- Structured output.
- Error and timeout capture.

## Tooling Policy

Codex is the primary coding and planning agent for this repository.

Use Codex for:

- Spec and plan creation.
- Code implementation.
- Tests and verification.
- Architecture review.
- Security review.
- Documentation updates.

OpenCode is optional and may be used for:

- Alternate local development workflows.
- Experiments with different model providers.
- Comparing coding-agent behavior.
- Independent review when useful.

OpenCode should not become a project dependency. The repository must remain buildable and testable without a specific coding-agent product.

## Git Policy

The project should use Git before implementation begins. This workspace currently contains a `.git` path that is not recognized as a valid repository by Git, so repository initialization or repair should be handled explicitly before code implementation.

Recommended branch pattern:

```text
main
feature/demo1-foundation
feature/demo1-ingestion
feature/demo1-analysis-engines
feature/demo1-dashboard
```

Recommended commit pattern:

```text
docs: add agentic-soar foundation spec
docs: add project management framework
feat: add canonical event schema
feat: add deterministic ssh brute force rule
test: cover case packet generation
```

## Demo Roadmap

### Demo 1 Foundation

Goal:

```text
event -> alert -> case -> deterministic/LLM analysis -> comparison -> recommendation -> audit -> SOC dashboard
```

Expected phase order:

1. Repository and project skeleton.
2. Canonical schema and PostgreSQL persistence.
3. Scenario fixtures and event generator.
4. Ingestion and trigger pipeline.
5. Case manager and evidence model.
6. Deterministic analysis engine.
7. Local LLM adapter and case packet builder.
8. Comparison and recommendation engine.
9. SOC dashboard.
10. End-to-end Demo 1 validation.

### Demo 2 Enterprise Realism

Goal:

```text
replace simplified sources with more realistic enterprise services while preserving core contracts
```

### Demo 3 Cyber Range

Goal:

```text
support richer adversary emulation, segmented networks, and controlled response execution
```

## Definition Of Ready

A phase is ready to implement when:

- Scope and non-goals are explicit.
- Required data contracts are known.
- Acceptance criteria exist.
- Evaluation criteria exist for security or LLM behavior.
- Dependencies are identified.
- Risky decisions have ADRs or explicit notes.

## Definition Of Done

A phase is done when:

- Implementation matches the phase spec.
- Tests pass.
- UAT criteria are satisfied.
- Evaluation results are recorded.
- Security and failure modes are considered.
- Documentation is updated.
- Follow-up work is captured instead of left implicit.
