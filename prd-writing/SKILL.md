# PRD Writing — Spec-Driven Development

Write behavioral specifications with traceable requirement IDs (FR-XX, NFR-XX, EC-XX) before any code is written. Based on the Spec-Driven Development methodology.

**Philosophy:** The spec defines WHAT the system must do, not HOW. Every requirement is testable. Every edge case is traced. When a bug is found downstream, debug the spec first, then fix the code.

---

## When to Use

- Before building any new feature
- When user says: "new feature", "I want to build", "write a PRD", "write a spec", "spec this out"
- This skill produces the behavioral spec. The `ticket-writer` skill then decomposes it into Linear tickets.

---

## Phase 1: Problem Validation

> Purpose: Kill bad ideas early. Validate this is a real problem worth solving.

Ask these 5 questions **one at a time**. Wait for the answer before asking the next.

**Q1:** "What is the problem? Describe the pain in one sentence."

**Q2:** "Who has this problem? Be specific — a persona with a role, not 'users.'"

**Q3:** "How do they solve it today without this feature?" *(The status quo is the real competitor. If they have a workaround, explain why it is insufficient.)*

**Q4:** "What evidence do you have that this is a real problem? Behavior and money count — waitlists and 'people said they want it' do not."

**Q5:** "Why now? What changed that makes this urgent?"

### Rules for this phase

- **Push back on vague answers.** "That persona is too vague — who specifically?" not "Great, let's refine that a bit."
- **Take positions.** If evidence is weak, say so: "That sounds like interest, not demand. What behavior or money backs this up?"
- **No sycophancy.** Never say "That's an interesting approach" or "There are many ways to think about this." Take a position.
- If the PM cannot articulate the problem clearly after pushback, flag the risk: "The problem statement is weak. Proceeding, but this is a risk."

### Output

Synthesize a **validated problem statement** (3-5 sentences) covering: who has the problem, what the problem is, current workaround, evidence, and urgency. Present it to the PM for confirmation before proceeding.

---

## Phase 2: Strategic Alignment

> Purpose: Define what the solution does, what it does NOT do, and how success is measured.

Ask these 4 questions **one at a time**.

**Q1:** "What is the ONE primary job this feature performs? State it as one job, not three."

**Q2:** "What is explicitly out of scope? Every feature has boundaries — name at least one thing this will NOT do."

**Q3:** "What are the success metrics? Pick one primary metric (what you're trying to move) and one guardrail metric (what must not regress)."

**Q4:** "State your hypothesis: 'We believe [action] will [outcome] for [persona], measured by [metric].'"

### Rules for this phase

- The PM **must** state at least one out-of-scope item. If they say "nothing is out of scope," push back: "Every feature has boundaries. What is one thing this feature will NOT do?"
- **Scope is sacred.** Once defined, adding anything from Out of Scope later requires the PM to explicitly acknowledge they are expanding scope and revisit the success metric.

### Output

A **scope block** containing: primary job, out of scope list, primary metric, guardrail metric, and hypothesis. Present for confirmation.

---

## Phase 3: Behavioral Specification

> Purpose: Write the formal SDD spec. This is the core deliverable.

This phase is NOT question-and-answer. Draft the spec from Phase 1-2 answers, then present it for review.

### 3a. User Stories (US-XX)

Write in Given-When-Then format with MoSCoW priority.

```
| ID    | Story                                                          | Priority |
|-------|----------------------------------------------------------------|----------|
| US-01 | Given [precondition], When [action], Then [expected outcome]   | Must     |
| US-02 | Given [precondition], When [action], Then [expected outcome]   | Should   |
```

### 3b. Functional Requirements (FR-XX)

Each requirement: one sentence, testable, traces to a user story, uses "shall."

```
| ID    | Requirement                              | Story | Priority |
|-------|------------------------------------------|-------|----------|
| FR-01 | The system shall [behavior].             | US-01 | Must     |
| FR-02 | The system shall [behavior].             | US-01 | Must     |
| FR-03 | The system shall [behavior].             | US-02 | Should   |
```

**Language rule:** FRs always use "shall." Never "should," "might," or "could consider." The word "Should" belongs in the priority column, not in the requirement text.

### 3c. Non-Functional Requirements (NFR-XX)

Performance, security, accessibility, and reliability constraints.

```
| ID     | Requirement                              | Category      |
|--------|------------------------------------------|---------------|
| NFR-01 | [constraint]                             | performance   |
| NFR-02 | [constraint]                             | security      |
```

### 3d. Edge Cases (EC-XX)

Boundary conditions. Each traces to an FR.

```
| ID    | Condition                | Behavior                           | Traces to |
|-------|--------------------------|------------------------------------|-----------|
| EC-01 | When [edge condition]    | The system shall [behavior]        | FR-01     |
| EC-02 | When [edge condition]    | The system shall [behavior]        | FR-03     |
```

### Self-review checklist

Before presenting the spec, run this checklist silently:

1. **Placeholder scan** — Are there any "TBD" or vague terms? If something is unknown, mark it explicitly as `TBD - needs research` rather than inventing a requirement.
2. **Traceability** — Does every FR trace to a US? Does every EC trace to an FR?
3. **Language** — Do all FRs use "shall"? Are there any weasel words ("should," "might," "could")?
4. **Completeness** — Is there at least one EC for each Must-have FR?
5. **Testability** — Can you write a test for every FR? If not, rewrite it until you can.

If any check fails, fix the spec before presenting it.

### Output

Present the full spec (US + FR + NFR + EC tables) to the PM for review. Iterate until the PM is satisfied.

---

## Phase 4: Codebase Grounding

> Purpose: Validate the spec against the actual codebase for feasibility.

If invoked inside a project directory, explore the codebase and report:

1. **Reuse** — Which existing modules, services, entities, or patterns support the spec?
2. **New work** — Which FRs require new modules, entities, or endpoints?
3. **Risks** — Any FRs that conflict with the current architecture?
4. **Suggested phases** — What to build first based on dependencies (e.g., backend before frontend).

The PM may adjust spec priorities based on feasibility (e.g., downgrade an FR from Must to Should if it requires significant new infrastructure).

If invoked outside a project directory (no codebase), skip this phase and note: "Codebase grounding: skipped (no project context)."

---

## Phase 5: Approval and Handoff

> Purpose: Final review, save the spec, and connect to the downstream workflow.

### 5a. Present the complete spec

Assemble the full document using the template below and present it to the PM.

### 5b. Get explicit approval

Ask: **"Is this spec approved? If not, tell me what to change."**

Iterate until the PM says it is approved. The spec is NOT done until explicitly approved.

### 5c. Save the spec

Save to `docs/specs/[feature-name]-spec.md` in the project directory.

### 5d. State the next steps

After approval, tell the PM:

1. **Create tickets:** "Use `/ticket-writer` to create Linear tickets from this spec. Must-have FRs become acceptance criteria. EC-XX IDs become test cases in engineering sub-issues."
2. **Implement:** "Follow `development-plan.md` for the build workflow, and `github-flow` for the push-to-merge workflow."
3. **When bugs are found:** "Trace the bug to its FR/EC ID. If the spec was wrong or incomplete, fix the spec first — add a new EC or correct the FR. Then fix the code. Debug the spec, not the code."

---

## Spec Document Template

```markdown
# [Feature Name] — Behavioral Specification

**Status:** Draft | Approved
**Author:** [PM name]
**Date:** [date]
**Version:** 1.0

---

## 1. Problem Statement

[Validated problem statement from Phase 1 — who, what, current workaround, evidence, urgency]

## 2. Strategic Alignment

**Primary job:** [one sentence]
**Hypothesis:** We believe [action] will [outcome] for [persona], measured by [metric].
**Success metric:** [primary metric + target]
**Guardrail metric:** [guardrail metric + threshold]

## 3. Out of Scope

- [Item 1] — [reason]
- [Item 2] — [reason]

## 4. User Stories

| ID | Story | Priority |
|----|-------|----------|
| US-01 | Given [precondition], When [action], Then [outcome] | Must |
| US-02 | Given [precondition], When [action], Then [outcome] | Should |

## 5. Functional Requirements

| ID | Requirement | Story | Priority |
|----|-------------|-------|----------|
| FR-01 | The system shall [behavior]. | US-01 | Must |
| FR-02 | The system shall [behavior]. | US-01 | Must |
| FR-03 | The system shall [behavior]. | US-02 | Should |

## 6. Non-Functional Requirements

| ID | Requirement | Category |
|----|-------------|----------|
| NFR-01 | [constraint] | performance |
| NFR-02 | [constraint] | security |

## 7. Edge Cases

| ID | Condition | Behavior | Traces to |
|----|-----------|----------|-----------|
| EC-01 | When [edge condition] | The system shall [behavior] | FR-01 |
| EC-02 | When [edge condition] | The system shall [behavior] | FR-03 |

## 8. Codebase Feasibility

**Reuse:** [existing modules/patterns]
**New work:** [new modules/entities/endpoints needed]
**Risks:** [architectural conflicts]
**Suggested phases:** [implementation order]

## 9. Decisions Log

| Decision | Choice | Alternatives Considered | Rationale |
|----------|--------|------------------------|-----------|
| [decision] | [choice] | [alternatives] | [why] |

## 10. Approval

- [ ] PM approved
- [ ] Spec saved to `docs/specs/`
```

---

## Global Rules

1. **One question at a time.** Never dump multiple questions. Wait for the answer before asking the next.
2. **Anti-sycophancy.** Take positions. If an answer is weak, say so directly. Never hedge.
3. **No invented requirements.** If something is unknown, write `TBD - needs research`. Never fabricate.
4. **FRs use "shall."** Every FR must be testable. If you cannot write a test for it, rewrite it.
5. **HARD GATE: No implementation before spec approval.** No code, no tickets, no branches until the PM explicitly approves the spec.
6. **Debug the spec, not the code.** When a bug is found downstream, trace it to the FR/EC ID. Fix the spec first, then fix the code.
7. **Linear phase progression.** You cannot skip to Phase 3 without completing Phases 1-2. Each phase's output feeds the next.
8. **Scope is sacred.** Adding anything from Out of Scope requires the PM to explicitly acknowledge the scope change and revisit the success metric.

---

## How This Skill Fits the Workflow

```
/prd-writing  →  approved behavioral spec
      ↓
/ticket-writer  →  Linear parent (User Story + SPICED) + engineering sub-issues
                    FR-XX → SPICED Decision (Acceptance Criteria)
                    EC-XX → engineering sub-issue Testing Requirements
      ↓
development-plan.md  →  branch, build, test, commit
      ↓
/github-flow  →  push, PR, review, CI, merge, deploy
      ↓
Bug found?  →  trace to FR/EC ID → fix the spec → then fix the code
```

---

## Attribution

Combined from:
- **gstack** `/office-hours` — demand validation, forcing questions, anti-sycophancy
- **everything-claude-code** `/prp-prd` — interactive PRD phases, hypothesis format, codebase grounding
- **superpowers** `brainstorming` — hard gate (no implementation before spec), spec self-review
- **SDD article** (technomanagers.com) — behavioral spec format (FR/NFR/EC with IDs), debug the spec not the code
- **claude-skills** — MoSCoW priorities, Given-When-Then format, INVEST validation
