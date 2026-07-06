# Project Rules -- AI Long-Term Investment Analysis System

This file defines the standards every Agent, Component, Spec, and document must follow. No exceptions.

**Inherited from**: ai-investment-mentor/PROJECT_RULES.md
**Additions**: SDD Spec Template, A-share-specific data rules

---

## 1. Architecture Rules

1. **Single responsibility per Agent.** Each Agent does one thing.
2. **Chief Investment Agent never accesses data sources directly.**
3. **All inter-Agent communication uses structured data (JSON).**
4. **Every analysis must include evidence, confidence, and risks.**
5. **Deterministic before LLM.** If a formula can decide, the formula decides.
6. **Degrade gracefully.** Partial output beats no output.
7. **The user is the decision-maker.** System recommends, never executes.

---

## 2. Document Standards

| Document Type | Question It Answers | Location |
|---|---|---|
| Product Vision | Who is this for and what does success look like? | docs/architecture/ |
| System Overview | How do the pieces fit together? | docs/architecture/ |
| Agent Architecture | Why these Agents? How do they relate? | docs/architecture/ |
| Agent Spec | What does this Agent do? | agents/ |
| Component Spec | What does this deterministic component do? | components/ |
| ADR | Why did we make this decision? | adr/ |
| Spec (SDD) | What is the contract for this phase? | specs/ |
| TASK | What should Codex implement? | tasks/ |

---

## 3. SDD Spec Template

Every Spec in specs/ must follow this structure. A Spec is the binding contract between design and implementation for one Phase.

`
# Spec: [Phase N] - [Name]

## Purpose
[One sentence.]

## Inputs
[What data, config, or previous-phase output does this consume?]

## Outputs
[What concrete artifacts are produced? List every file.]

## Interface Contract
[Function signatures, class APIs, data schemas. THE binding part.]

## Behavior Specification
[For each function: normal behavior, edge cases, error cases.]

## Acceptance Criteria
[Testable statements. Each maps to at least one test.]

## Dependencies
[Which phases? Which libraries?]

## Out of Scope
[What must NOT be built.]
`

### When a Spec is "Done"
- Every function has a Behavior Specification
- Every Acceptance Criterion is testable
- Out of Scope prevents at least 3 scope-creep items
- A new developer could implement from it without asking questions

---

## 4. Agent Specification Template (inherited)

`
# [Agent Name]

## Identity
## Purpose
## Goal
## Inputs
## Outputs
## Memory
## Tools
## Workflow
## Constraints
## Consumers
## Failure Handling
## Future Evolution
`

---

## 5. Component Specification Template (inherited)

`
# [Component Name]

## Name
## Purpose
## Responsibilities
## Public Interface
## Dependencies
## Consumers
## Constraints
## Future Evolution
`

---

## 6. ADR Standard (inherited)

`
# ADR-[NNN]: [Title]

## Status
## Context
## Decision
## Consequences
## Alternatives Considered
`

---

## 7. TASK Standard (inherited)

`
# TASK-[NNN]: [Title]

## Context
## Objective
## Requirements
## Acceptance Criteria
## Out of Scope
## References
`

---

## 8. Directory Structure

`
ai-long-term-investment/
  AGENTS.md
  PROJECT_RULES.md          (this file)
  ROADMAP.md
  COMPREHENSIVE_INVESTMENT_FRAMEWORK.md
  docs/
    STATE-OF-THE-PROJECT.md
    architecture/
      01-product-vision.md
      02-system-overview.md
      03-agent-architecture.md
  adr/
    ADR-001 through ADR-004
  specs/        (SDD specs, one per Phase)
  agents/       (Agent specs only)
  components/   (Component specs only)
  tasks/        (Codex implementation tasks)
  backlog/      (archived ChatGPT output)
  src/          (source code)
  tests/        (tests)
  data/         (SQLite + cache, gitignored)
`
