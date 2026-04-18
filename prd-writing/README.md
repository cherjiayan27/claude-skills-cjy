# prd-writing

A Spec-Driven Development (SDD) skill for product managers. Guides you through writing a formal behavioral specification with traceable requirement IDs before any code is written. Combined from:

- **gstack** `/office-hours` — demand validation, forcing questions, anti-sycophancy
- **everything-claude-code** `/prp-prd` — interactive PRD phases, hypothesis format, codebase grounding
- **superpowers** `brainstorming` — hard gate (no implementation before spec), spec self-review
- **SDD article** (technomanagers.com) — behavioral spec format (FR/NFR/EC with IDs)
- **claude-skills** — MoSCoW priorities, Given-When-Then format

**The 5 phases:**

| Phase | What it does |
|---|---|
| 1. Problem Validation | 5 forcing questions — kill bad ideas early, push back on vague answers |
| 2. Strategic Alignment | Define primary job, out of scope, success metrics, testable hypothesis |
| 3. Behavioral Specification | Draft formal spec with US-XX, FR-XX, NFR-XX, EC-XX — all traceable |
| 4. Codebase Grounding | Validate spec against actual codebase — reuse, new work, risks |
| 5. Approval & Handoff | PM approves → spec saved → next step is `/ticket-writer` for Linear tickets |

**Core principle:** Debug the spec, not the code. When a bug is found downstream, trace it to the FR/EC ID and fix the spec first.
