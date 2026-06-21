---
name: agent-harness
description: Design, audit, or improve an AI agent harness — the infrastructure layer that wraps a model to make it reliable. Trigger with "design a harness for", "my agent keeps failing", "review my AGENTS.md", "set up agent observability", "agent security", "multi-agent orchestration", or any question about building reliable LLM agents beyond the model itself.
argument-hint: "<task description, AGENTS.md content, system architecture, or problem to solve>"
---

# /agent-harness

> **Agent = Model + Harness**
>
> For complex, long-horizon tasks, reliability depends less on the model than on the infrastructure wrapping it. A decent model with a great harness beats a great model with a bad harness.

Design, review, or improve the harness around an LLM agent — covering all seven ETCLOVG layers.

## Usage

```
/agent-harness <what you want to build or fix>
```

## How It Works

```
┌──────────────────────────────────────────────────────────────────┐
│                    AGENT HARNESS ENGINEERING                     │
├──────────────────────────────────────────────────────────────────┤
│  PHASE 1 — DIAGNOSE                                              │
│  ✓ Identify which ETCLOVG layer the problem lives in             │
│  ✓ Check for anti-patterns (model-blaming, context bloat, etc.)  │
│  ✓ Map failure mode → harness component that should fix it       │
├──────────────────────────────────────────────────────────────────┤
│  PHASE 2 — DESIGN                                                │
│  ✓ Recommend specific harness components per layer               │
│  ✓ Draft or improve AGENTS.md / CLAUDE.md                        │
│  ✓ Address cost–quality–speed tradeoffs explicitly               │
├──────────────────────────────────────────────────────────────────┤
│  PHASE 3 — TIGHTEN (The Ratchet)                                 │
│  ✓ Every past failure → a permanent rule or hook                 │
│  ✓ Verify behavior is observable and self-correctable            │
│  ✓ Review whether scaffolding is still load-bearing              │
└──────────────────────────────────────────────────────────────────┘
```

---

## The ETCLOVG Seven-Layer Taxonomy

The standard architecture for enterprise-grade agent harnesses.

### E — Execution Environment & Sandbox
The physical substrate where agent actions run. A sandbox is both a **cage** (protecting the host) and a **license** (allowing autonomous action without per-action approval).

| Concern | Engineering Approach |
|---------|----------------------|
| Safety | Isolate code execution (E2B, Docker, microVMs) |
| Autonomy | Define the bounded region where the agent acts freely |
| Permission reduction | Sandboxing can reduce permission prompts by ~84% |
| Cost | More faithful sandboxes = higher startup latency |

### T — Tool Interface & Protocol
How the agent discovers and executes external capabilities. More tools ≠ better — oversized tool menus degrade reliability.

| Concern | Engineering Approach |
|---------|----------------------|
| Interoperability | Use MCP (host-client-server, JSON-RPC typed exchange) |
| Tool selection | 10 focused tools > 50 overlapping ones |
| Security risk | MCP tool descriptions land in the prompt — malicious MCPs can prompt-inject |
| Progressive disclosure | Reveal tools only when the task calls for them |

### C — Context & Memory Management
Governs what the model sees at each step. **Context Rot** and **Context Drift** are the two main failure modes.

| Concept | Description |
|---------|-------------|
| **Context Rot** | Performance degrades as the window fills — starts well before the limit (e.g., a 200K model may degrade at 50K) |
| **Context Drift** | Over 100+ turns, agents lose track of the original goal, repeat work, contradict earlier decisions |
| **Lost in the Middle** | Accuracy drops 30%+ when relevant content sits in the middle of a long context |
| **Just-In-Time Context** | Load information only when the task needs it — not upfront |
| **Pointer Pattern** | `AGENTS.md` as a router: instructs the agent to use `read_file` for deep context, rather than hard-injecting everything |
| **Compaction** | Summarize and offload old context before the window fills; offload large tool outputs to disk |

### L — Lifecycle & Orchestration
Manages execution flow: planning, ReAct loops, error recovery, multi-agent coordination.

**Single-agent inner loop (ReAct):**
```
Reason → Act → Observe → Reason → ...
```

**Multi-agent patterns:**
| Pattern | Use Case |
|---------|----------|
| Hierarchical | Controller assigns work to subagents, integrates outputs |
| Team | Named agents with specialized roles (planner, coder, reviewer) |
| Fan-out | Parallel agents explore diverse solutions |
| Workflow | Explicit stages/control logic |
| Graph composition | Agents and tools as nodes in an interaction graph |

### O — Observability & Operations
First-class layer (not just lifecycle hooks). Uses OpenTelemetry as the de facto standard.

| What to capture | Tools |
|-----------------|-------|
| Structured traces | Langfuse, Arize Phoenix, MLflow |
| Token counts, cost, latency | OpenLLMetry, OpenInference |
| Failure attribution | AgentTrace, AgentSight |
| Anomaly detection | Arize Phoenix |

**Principle:** Observability tells you *why* an agent succeeded or failed, not just whether it did.

### V — Verification & Evaluation
A quality-control loop, not just a pass/fail metric. Evaluate the **trajectory** (was the path acceptable?), not only the final output.

| Stage | What to check |
|-------|---------------|
| Pre-execution | Readiness validation, tool availability |
| During | Trace capture, permission boundary respect |
| Post | Multi-level judgement, failure attribution |
| Regression | Feed findings back into AGENTS.md and hooks |

**Principle — Success is Silent, Failures are Verbose:**
- If lint/tests pass → say nothing (preserve context)
- If they fail → inject detailed error text into the loop for self-correction

### G — Governance & Security
The "safety brakes." Constrains behavior, enforces permissions, and holds agents accountable.

| Mechanism | Purpose |
|-----------|---------|
| Permission models | Least-privilege, role-based access |
| Identity management | Audit trail — who authorized what |
| Lifecycle hooks | Block dangerous actions before execution |
| Declarative constitutions | Rules encoded in AGENTS.md (e.g., `NEVER delete database`) |
| Audit infrastructure | Provenance for all agent actions |

---

## AGENTS.md Design Guide

The single highest-leverage configuration point. It lands in the system prompt every turn.

### Ideal Structure (5 core sections)

```markdown
# [Project Name] — AGENTS.md

## 1. Project Context (WHAT & WHY)
# Solves the cold-start problem
- What this project does
- Directory structure overview
- Framework / stack at a glance

## 2. Architecture & Core Libraries
# Immutable technical rules
- Required libraries (use X, never use Y)
- Architectural constraints (e.g., "always use App Router, not Pages Router")

## 3. Coding Conventions (HOW)
# Ensures consistency across all AI coding tools
- Naming conventions, file structure
- Testing requirements
- "Never comment out tests — delete or fix them"

## 4. Governance & Boundaries
# Safety brakes — absolute prohibitions
- NEVER delete production data
- NEVER commit secrets
- NEVER push directly to main

## 5. CI/CD Workflows
# Teaches the agent your operational habits
- "Always run lint and test before committing"
- Required checks before any PR
```

### Rules for a good AGENTS.md

- **Keep it under 60 lines** — every line competes for attention; more rules = each rule matters less
- **Pilot's checklist, not style guide** — action-oriented constraints, not documentation
- **Earn each line** — every rule must trace to a specific past failure or hard external constraint
- **The Pointer Pattern** — use `read_file` directives for deep context, don't hard-inject everything
- **Never unconditionally load** the full file into every sub-agent — isolate context per role

---

## Core Engineering Principles

### The Ratchet
Treat every agent mistake as a permanent signal — not a retry, not a story.

```
Agent mistake → diagnose root cause → add rule to AGENTS.md or verification hook
→ specific error never repeats
```

Every line in a good AGENTS.md should trace back to a specific failure. If it doesn't, it's noise.

### Behaviour-First Design
Work backwards from the behaviour you want:

```
Behaviour we want (or want to fix) → harness component that delivers it
```

| Unwanted behaviour | Harness fix |
|--------------------|-------------|
| Agent uses wrong library | Add to Architecture section of AGENTS.md |
| Agent runs destructive commands | Add Governance hook to block it |
| Agent gets lost in 40-step task | Split into planner + executor subagents |
| Agent "finishes" broken code | Wire typecheck back-pressure into the loop |
| Agent repeats work after 100 turns | Add context reset + state summary artifact |

### Guides vs. Sensors (Feedforward vs. Feedback)
| Type | What it does | Example |
|------|-------------|---------|
| **Guide** (feedforward) | Steers the agent before it acts | AGENTS.md rule, tool description |
| **Sensor** (feedback) | Observes after the agent acts, enables self-correction | Linter output injected into loop, test failure message |

Sensors are most powerful when their output is optimized for LLM consumption (e.g., custom linter messages that include the self-correction instruction).

---

## Anti-Patterns to Avoid

| Anti-Pattern | What it looks like | Fix |
|---|---|---|
| **Model Blaming** | "The model is dumb, wait for GPT-next" | Reframe as a configuration problem; fix the harness |
| **Hard-Injection** | Concatenating all docs into one giant system prompt | Use the Pointer Pattern + JIT context loading |
| **Unconditional Loading** | Loading 500-line AGENTS.md into every sub-agent | Isolate context per agent role |
| **Context Bloat** | More rules = better | More rules = each rule matters less; earn every line |
| **Scaffold Hoarding** | Never removing harness components | Every component encodes an assumption — review as models improve |
| **Tool Menu Sprawl** | 50 tools exposed at startup | Progressive disclosure; 10 focused tools > 50 overlapping |
| **Silent Failures** | Checks run but errors aren't fed back | Success is silent; failures are verbose |

---

## System-Level Tradeoffs

### Cost–Quality–Speed Trilemma
Stronger verification (V) and governance (G) → higher quality and safety → higher cost and latency. You must choose which checks are synchronous vs. asynchronous.

### Capability–Control Tradeoff
More tools = broader capability = larger prompt-injection surface + planning errors. More permissive sandboxes = more autonomy = larger blast radius if something goes wrong.

### Harness Coupling Problem
Layers are not independent. A change to a tool schema can break context management or orchestration. **Test harness changes as system changes, not layer changes.**

### Harness Staleness
Every harness component encodes an assumption about what the model can't do. As models improve, some scaffolding becomes unnecessary overhead. Periodically audit: is each component still load-bearing?

---

## Three Phases of AI Engineering

| Phase | Era | Focus |
|-------|-----|-------|
| Prompt Engineering | 2022–2024 | Optimize a single text input to a single model call |
| Context Engineering | 2025 | Optimize what information the model sees at each step |
| **Harness Engineering** | **2026** | **Optimize the entire system: state, tools, governance, verification** |

---

## Output Format

```markdown
## Harness Review: [project or agent name]

### ETCLOVG Layer Assessment
| Layer | Status | Key Issue |
|-------|--------|-----------|
| E – Execution | ✅ / ⚠️ / ❌ | [observation] |
| T – Tool Interface | ... | ... |
| C – Context | ... | ... |
| L – Lifecycle | ... | ... |
| O – Observability | ... | ... |
| V – Verification | ... | ... |
| G – Governance | ... | ... |

### Top Issues
1. [Layer] [Issue] — [Recommended fix]

### AGENTS.md Recommendations
[Draft or revised AGENTS.md sections]

### The Ratchet — Rules to Add
- [ ] [Rule traced to specific failure]
```
