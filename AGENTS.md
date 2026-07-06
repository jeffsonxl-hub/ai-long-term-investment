# AGENTS.md -- Project Workflow Instructions

## Inherited from ai-investment-mentor

This file exists because the reference project (ai-investment-mentor)
established it as the single source of workflow truth for Codex. We
inherit the structure and adapt the rules.

---

## Phase Validation Checklist

Before writing any code in a new phase, run this checklist. If any
answer is "No," fix it first.

- [ ] Does the phase have a completed spec in `specs/`? (SDD rule)
- [ ] Does every new Agent use the standard Agent template?
- [ ] Are Components separated from Agents?
- [ ] Does every document answer one specific question?
- [ ] Can Codex implement this without guessing?

The SDD rule means: no `.py` file is created until the corresponding
`specs/<phase>-<name>.md` is written, reviewed, and approved. The spec
defines the contract. The code implements the contract. Tests verify
the contract.

---

## SDD Workflow (Specification Driven Development)

Every Phase follows this sequence:

```
1. SPEC  -- write specs/<phase>-<name>.md
2. REVIEW -- user approves the spec
3. TASK  -- write tasks/<phase>-<name>.md (implementation instructions)
4. CODE  -- implement src/ and tests/
5. VERIFY -- run tests, confirm acceptance criteria pass
```

Steps 1-2 are the "spec gate." No code passes through without a spec.
A spec answers: "If I gave this to another developer, could they
implement it without asking any clarifying questions?"

### Spec Template

```markdown
# Spec: [Phase N] -- [Name]

## Purpose
[One sentence. What does this phase deliver and why?]

## Inputs
[What data, config, or previous-phase output does this consume?]

## Outputs
[What concrete artifacts does this produce? Files, modules, database tables.]

## Interface Contract
[Function signatures, class APIs, data schemas. THE binding part.]

## Behavior Specification
[For each function/method: normal behavior, edge cases, error cases.]

## Acceptance Criteria
[Testable statements. "When X, then Y." Each criterion maps to a test.]

## Dependencies
[Which phases must be complete? Which external libraries?]

## Out of Scope
[What must NOT be built in this phase. Prevent scope creep.]
```

---

## How We Work

1. Codex is the active collaborator on this project, not ChatGPT.
   Previous ChatGPT output (Long-term investment chat with Chatgpt.md)
   is domain reference, not authoritative implementation guidance.
2. The user specifies which phase is active. Do not assume.
3. Every phase follows: spec -> review -> task -> code -> verify.
4. No code without a spec. No spec without acceptance criteria.

## Always-Update Rule

Whenever the project status changes or the roadmap advances, update:

- `docs/STATE-OF-THE-PROJECT.md` -- update "Current Phase" and context
- `ROADMAP.md` -- update phase statuses and fix stale references

These are persistent memory. If they go stale, context is lost between
sessions.

## Document Standards

All documents follow templates defined in `PROJECT_RULES.md`:

- Agent specs use the Agent Template (12 sections)
- Component specs use the Component Template (8 sections)
- ADRs follow the ADR standard (5 sections)
- TASK files must be specific enough for Codex to implement without guessing
- Spec files follow the SDD Spec Template above

## Key Distinction (inherited from ai-investment-mentor)

Agents think and decide (they use LLMs).
Components execute and store (deterministic, no LLM).

Never confuse them. Never put a Component in `agents/`.
Never let an Agent open a database connection.
Never promote a deterministic function to an Agent because "Agent" sounds cool.

## Backlog Folder

The `backlog/` directory contains original ChatGPT-generated files.
These are archival only. The definitive documents live in `docs/`,
`adr/`, `components/`, `specs/`, and `tasks/`.

## Language

All project documentation, code comments, and commit messages are in
English. Domain-specific Chinese terms (e.g., stock names, indicator
names like "扣非净利润") may appear in data and variable names.
