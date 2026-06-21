---
name: sdd-spec
description: Create specifications for Specification-Driven Development (SDD) systems. Help write constitution.md, spec.md, plan.md, and tasks.md for AI-assisted projects. Trigger with "write a spec for", "create specs for my project", "SDD workflow", "spec-driven development", "write a constitution for", "create a technical plan", "break this into tasks", or any request to build structured documentation before coding.
argument-hint: "<project description, feature, or existing spec document to review/improve>"
---

# /sdd-spec

> **Specs are not documentation — they are executable intent.**
>
> SDD (Specification-Driven Development) treats structured specs as living artifacts that drive AI agents rather than describe what they already built. Write the spec first; the code follows.

Create, review, or improve SDD artifacts — constitution, spec, plan, and task breakdown — for AI-assisted development.

## Usage

```
/sdd-spec <project description, feature, or existing spec to review>
```

## How It Works

```
┌──────────────────────────────────────────────────────────────────┐
│                        SDD SPEC SKILL                            │
├──────────────────────────────────────────────────────────────────┤
│  PHASE 1 — CLARIFY                                               │
│  ✓ Identify what's being built and why (user problem)            │
│  ✓ Determine which artifact is needed (constitution / spec /     │
│    plan / tasks) or if a full SDD suite is required              │
│  ✓ Surface ambiguities before writing — resolve them in writing  │
├──────────────────────────────────────────────────────────────────┤
│  PHASE 2 — DRAFT                                                 │
│  ✓ Write spec artifacts using SDD templates                      │
│  ✓ Keep spec.md tech-agnostic; put stack in plan.md              │
│  ✓ Apply three-tier boundaries and six-area checklist            │
├──────────────────────────────────────────────────────────────────┤
│  PHASE 3 — VALIDATE                                              │
│  ✓ Cross-artifact consistency check (constitution ↔ spec ↔ plan) │
│  ✓ Confirm each task is independently testable                   │
│  ✓ Flag anti-patterns (vague criteria, implementation leakage,   │
│    over-engineering, missing success criteria)                   │
└──────────────────────────────────────────────────────────────────┘
```

---

## The Four SDD Artifacts

Each artifact answers a different question. They form an accountability chain — validate each before moving to the next.

| Artifact | File | Answers | Tech-specific? |
|----------|------|---------|----------------|
| **Constitution** | `constitution.md` | What are the non-negotiable constraints? | Yes |
| **Specification** | `spec.md` | What are we building and why? | **No** |
| **Technical Plan** | `plan.md` | How will we build it? | Yes |
| **Tasks** | `tasks.md` | What exactly does the agent do next? | Yes |

### Implementation Levels

| Level | Description | When to use |
|-------|-------------|-------------|
| **Spec-first** | Write a spec for one specific task or feature | Single-feature work |
| **Spec-anchored** | Maintain the spec alongside the codebase for long-term reference | Active projects |
| **Spec-as-source** | Spec is the primary artifact; code is purely generated from it | Greenfield AI-first projects |

---

## Artifact 1 — Constitution (`constitution.md`)

Establishes **non-negotiable ground rules** the AI must never violate. Written once; rarely changed.

### Template

```markdown
# constitution.md

## Tech Stack (Hard Constraints)
- Must use: [e.g., Next.js 14, TypeScript, PostgreSQL]
- Must NOT use: [e.g., class components, any, raw SQL outside ORM]
- Runtime: [e.g., Node 20 LTS]

## Quality Standards
- Test coverage: [e.g., ≥ 80% for all new code]
- Lint: [e.g., ESLint with project rules; zero warnings in CI]
- Performance: [e.g., p95 API response < 200ms]

## Non-Goals (Explicit Scope Exclusions)
- [e.g., Mobile app is out of scope for this iteration]
- [e.g., No internationalization in v1]

## Three-Tier Boundaries
✅ Always do (no approval needed):
- [e.g., Always run tests before committing]
- [e.g., Always log errors to the monitoring service]

⚠️ Ask first (human approval required):
- [e.g., Ask before adding new dependencies]
- [e.g., Ask before modifying database schemas]
- [e.g., Ask before changing CI/CD configuration]

🚫 Never do (hard stops):
- [e.g., Never commit secrets or API keys]
- [e.g., Never delete production data]
- [e.g., Never push directly to main]
- [e.g., Never remove a failing test without approval]
```

**Rules for a good constitution:**
- Keep it under 1 page — brevity makes every rule matter more
- Every constraint must trace to a real reason (compliance, past failure, team decision)
- Tech stack goes here, not in `spec.md`

---

## Artifact 2 — Specification (`spec.md`)

Defines **what** to build and **why** — completely detached from technical implementation. Written before any code. The agent uses this to understand user intent.

### Template

```markdown
# spec.md — [Project / Feature Name]

## Problem Statement
[One paragraph: what problem does this solve? Who has the problem? What happens now without this?]

## Goals
- [Measurable outcome 1, e.g., "Users can reset their password in under 60 seconds"]
- [Measurable outcome 2]

## Non-Goals
- [Explicitly what this does NOT solve]

## User Stories
| As a... | I want to... | So that... |
|---------|-------------|-----------|
| [user type] | [action] | [outcome] |

## Functional Requirements
### [Feature Area 1]
- FR-01: [Requirement written as a verifiable statement]
- FR-02: [...]

### [Feature Area 2]
- FR-03: [...]

## Success Criteria / Conformance Tests
- [ ] [Specific input → expected output, e.g., "Submitting a valid email triggers a reset email within 5s"]
- [ ] [Edge case: "Invalid email shows inline error, not a page reload"]

## Out of Scope
- [Anything that could be misinterpreted as in scope]
```

**Rules for a good spec.md:**
- No tech stack, no framework names, no file paths — those go in `plan.md`
- Every requirement must be falsifiable (avoid "should be fast", "should look good")
- Success criteria are conformance tests: specific inputs → specific outputs
- Include edge cases and error states — they define 40% of the real behavior

---

## Artifact 3 — Technical Plan (`plan.md`)

Defines **how** to build it — architecture, data models, API contracts, and technology choices. Written after `spec.md` is validated. Must not contradict the constitution.

### Template

```markdown
# plan.md — [Project / Feature Name]

## Architecture Overview
[1–2 paragraphs or diagram describing the system structure]

## Tech Stack
- Language: [e.g., TypeScript 5.x]
- Framework: [e.g., Next.js 14 App Router]
- Database: [e.g., PostgreSQL 16 via Prisma ORM]
- Auth: [e.g., NextAuth.js v5]
- Testing: [e.g., Vitest + React Testing Library]

## Project Structure
```
src/
  app/          # Next.js App Router pages
  components/   # Reusable UI components
  lib/          # Business logic and utilities
  db/           # Prisma schema and migrations
tests/          # Unit and integration tests
```

## Data Models
```typescript
// [Model name]
type [Model] = {
  id: string;
  [field]: [type]; // [comment if non-obvious]
};
```

## API Contracts
### [Endpoint Name]
- **Method + Path:** `POST /api/[resource]`
- **Request body:** `{ field: type }`
- **Response (200):** `{ field: type }`
- **Error (4xx):** `{ error: string, code: string }`

## Key Technical Decisions
| Decision | Choice | Rationale |
|----------|--------|-----------|
| [e.g., ORM] | [e.g., Prisma] | [e.g., Type-safe queries, migration tooling] |

## Validation Against Constitution
- [ ] All stack choices comply with constitution constraints
- [ ] No prohibited libraries used
- [ ] Quality standards are achievable with this architecture
```

---

## Artifact 4 — Tasks (`tasks.md`)

Breaks the plan into **concrete, independently testable work items**. Each task is something an agent can implement and test in isolation — closer to TDD than a backlog.

### Template

```markdown
# tasks.md — [Project / Feature Name]

## Phase 1: [Foundation / Setup]

### Task 1.1: [Name — verb phrase]
**What:** [One sentence description]
**Acceptance criteria:**
- [ ] [Verifiable condition, e.g., "Unit tests pass for valid/invalid inputs"]
- [ ] [Verifiable condition]
**Spec refs:** FR-01, FR-02
**Dependencies:** none

### Task 1.2: [Name]
...

## Phase 2: [Core Feature]

### Task 2.1: [Name]
**What:** [...]
**Acceptance criteria:**
- [ ] [...]
**Spec refs:** FR-03
**Dependencies:** Task 1.1

## Phase 3: [Polish / Testing]
...
```

**Rules for good tasks:**
- Each task must be completable in one agent session (< 200 lines of new code)
- Acceptance criteria are runnable — they should map to test cases
- Reference spec FRs so the agent knows which requirement it's satisfying
- Don't mix feature implementation with database migrations in the same task

---

## Six-Area Completeness Checklist

From the GitHub analysis of 2,500+ agent config files — the most effective specs cover all six:

| Area | What to include | Status |
|------|----------------|--------|
| **Commands** | Full commands with flags: `npm test`, `pytest -v`, `npm run build` | ☐ |
| **Testing** | Framework, how to run, where test files live, coverage expectations | ☐ |
| **Project Structure** | Where source, tests, and docs belong — explicit paths | ☐ |
| **Code Style** | One real code snippet beats three paragraphs; naming conventions | ☐ |
| **Git Workflow** | Branch naming, commit format, PR requirements | ☐ |
| **Boundaries** | Three-tier (Always / Ask first / Never) — especially "Never commit secrets" | ☐ |

---

## SDD Workflow Sequence

```
[You] Describe vision
        ↓
/speckit.constitution → constitution.md (non-negotiables)
        ↓  validate
/speckit.specify → spec.md (WHAT, user-centric, tech-agnostic)
        ↓  /speckit.clarify → resolve ambiguities
        ↓  /speckit.checklist → completeness gate
/speckit.plan → plan.md (HOW, architecture, stack)
        ↓  validate against constitution
/speckit.tasks → tasks.md (concrete work items)
        ↓  /speckit.analyze → cross-artifact consistency check
/speckit.implement → agent executes tasks one-by-one
        ↓  verify after each task (run tests)
[Done]
```

**Gate rule:** Do not move to the next phase until the current artifact is validated. Never let the agent write code before `spec.md` and `plan.md` exist.

---

## Key Principles

**1. Spec before code — always**
Use Plan Mode (read-only) to draft the spec before writing a single line. Once the spec is solid, exit Plan Mode and execute.

**2. Separate WHAT from HOW**
`spec.md` contains zero tech stack details. `plan.md` contains zero user stories. Mixing them causes the agent to optimize for the wrong thing.

**3. Modular context**
Don't dump the full spec into every prompt. Feed the agent only the relevant section for the current task. Backend work → Backend section only.

**4. The spec is the single source of truth**
When the agent discovers a better approach, update the spec first, then re-sync the agent: "I've updated the spec as follows… adjust accordingly."

**5. Human-in-the-loop at every gate**
The developer's role is to steer and critique, not to write every line. Verify: Does the spec capture what I want? Does the plan account for my constraints? Are there edge cases the AI missed?

**6. Authoritative natural language over structured formats**
Write specs in clear, authoritative prose. LLMs process authoritative natural language better than terse JSON or YAML.

---

## Anti-Patterns

| Anti-Pattern | What it looks like | Fix |
|---|---|---|
| **Vague criteria** | "Should be fast", "looks good", "user-friendly" | Write measurable: "p95 < 200ms", "passes WCAG AA" |
| **Implementation leakage** | Mentioning React/PostgreSQL in spec.md | Move all tech choices to plan.md |
| **Over-engineering early** | Forcing microservices into constitution for an MVP | Start simple; add complexity only when the spec demands it |
| **Monolithic prompt** | Full 50-page spec in every message | Break into sections; feed only the relevant slice |
| **Spec abandonment** | Write spec once, then ignore it | Update the spec with every decision; it's a living document |
| **Skipping human review** | Letting the agent commit what passes tests | Review critical code paths — "I won't commit code I couldn't explain" |
| **Vibe coding to production** | Shipping rapid-prototype code without specs | Distinguish vibe coding (exploration) from production engineering |
| **No test increments** | Testing only after full implementation | Verify after each task — catching errors early costs far less |

---

## Output Format

```markdown
## SDD Artifact: [artifact type] — [project/feature name]

### Artifact
[Full artifact content using the appropriate template above]

### Validation Notes
- ✅ [What's solid]
- ⚠️ [What's ambiguous or needs clarification]
- ❌ [What's missing or violates an SDD principle]

### Next Step
[Which artifact to write next and what input is needed]
```

---

## Quick Reference

| If you need to... | Artifact | Key rule |
|---|---|---|
| Define non-negotiable constraints | `constitution.md` | < 1 page; every line has a reason |
| Describe what to build for users | `spec.md` | No tech stack; only measurable outcomes |
| Choose architecture and stack | `plan.md` | Must not contradict constitution |
| Give the agent concrete work | `tasks.md` | Each task independently testable |
| Check spec completeness | Six-area checklist | Commands, Testing, Structure, Style, Git, Boundaries |
| Prevent agent misbehavior | Three-tier boundaries | Always / Ask first / Never |
| Keep agent on track across sessions | Feed spec sections as context | Modular — not the whole file every time |
