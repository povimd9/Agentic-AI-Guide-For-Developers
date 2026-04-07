# Agentic AI Development: Guide for Teams

## Setting Up and Running AI-Assisted Software Development

*A practical guide for tech leads, prompt engineers, and engineering managers who want to use AI agents effectively in their development process. Compiled from battle-tested practices across a multi-repository, safety-critical systems project. Every recommendation exists because its absence caused a real bug, integration failure, or wasted work.*

---

## Table of Contents

1. [Overview & Philosophy](#1-overview--philosophy)
2. [Project Structure for Agentic AI](#2-project-structure-for-agentic-ai)
3. [The Design-to-Implementation Pipeline](#3-the-design-to-implementation-pipeline)
4. [Writing Effective Agent Instructions](#4-writing-effective-agent-instructions)
5. [Defining Agent Roles & Authority](#5-defining-agent-roles--authority)
6. [Quality Gates & Review Process](#6-quality-gates--review-process)
7. [Story Execution Orchestration](#7-story-execution-orchestration)
8. [Epic & Story Template Design](#8-epic--story-template-design)
9. [Specification Ownership & Cross-Team Coordination](#9-specification-ownership--cross-team-coordination)
10. [Completion Verification Framework](#10-completion-verification-framework)
11. [Escalation Framework](#11-escalation-framework)
12. [Common Pitfalls & Lessons Learned](#12-common-pitfalls--lessons-learned)

---

## 1. Overview & Philosophy

### Why This Approach

AI agents are powerful code generators but poor decision-makers. Left unguided, they produce plausible code that silently violates architectural invariants, swallows errors behind fallback values, and claims completion without verification. The practices in this guide exist to channel that power into reliable, high-quality output.

### Core Principles

1. **Design before implementation.** Agents execute plans well. They make architectural decisions poorly. Front-load all decisions into design documents and epics before any agent writes code.

2. **One story at a time.** Agents lose coherence across large scopes. Constrain each agent session to a single story with clear acceptance criteria, validate fully, then move on.

3. **Verify everything.** "It compiles" and "the tests pass" are necessary but insufficient. Every acceptance criterion needs a concrete command that produces observable proof. Static analysis is not verification.

4. **Fail-fast, never fail-silent.** The most damaging bugs agents produce aren't crashes — they're silent fallbacks that hide broken behavior behind default values. Enforce fail-fast at every level.

5. **Escalate decisions, not problems.** Agents should surface multi-choice situations to the human, not pick one and move on. The human makes product and architecture decisions; the agent implements them.

---

## 2. Project Structure for Agentic AI

### 2.1 Instruction Files

Every repository that an agent will work in needs an instruction file (for example `CLAUDE.md`, `AGENTS.md`, or a platform-specific file such as `.github/copilot-instructions.md`) at its root. In a multi-repo workspace, use a layered approach:

```
workspace-root/
  CLAUDE.md              # Shared rules across all repos
  VERSIONS.yaml          # Pinned component versions
  repo-a/
    CLAUDE.md            # Repo-specific rules (references root)
  repo-b/
    CLAUDE.md            # Repo-specific rules (references root)
```

**The root instruction file** contains cross-cutting rules: fail-fast philosophy, security standards, completion verification, workflow rules, encoding standards.

**Each repo's instruction file** contains repo-specific guidance: build commands, deployment steps, architecture context, boundary rules ("never modify code outside this repo"), and domain-specific conventions.

**Key rule:** The repo-specific file should say "Read the root instruction file first" at the top. Don't duplicate shared rules — reference them.

### 2.2 Version Pinning

Maintain a central version file that lists every library, framework, and tool version used in the project. Agents' training data is outdated — this file is the source of truth.

```yaml
# VERSIONS.yaml
components:
  mqtt_broker:
    name: Mosquitto
    version: "2.0.18"
  database:
    name: PostgreSQL
    version: "17"
    extensions: [timescaledb-2.23]
  # ...
```

**Why this matters:** We've seen agents use EMQX 4.x config syntax when the project uses 5.x, deprecated React Router patterns, and library features that don't exist in pinned versions. A version file prevents all of these.

### 2.3 Specification Repository

For multi-team projects, maintain a shared specification repository as the single source of truth for integration contracts (API schemas, message formats, protocol definitions). This prevents the "two teams, two interpretations" problem.

```
specs/
  mqtt/
    TOPIC_INDEX.md
    MESSAGE_FORMAT.md
    SYNC_PROTOCOL.md
  api/
    REST_ENDPOINTS.md
  _proposals/         # Change proposals (not yet approved)
    templates/
```

### 2.4 Agent Definitions

If using specialized agents, store their role definitions alongside the codebase:

```
.claude/agents/
  embedded-systems-agent.md
  cloud-infrastructure-agent.md
  security-reviewer.md
  # ...
```

Use whatever directory your platform supports for agent definitions; `.claude/agents/` is one concrete example.

Each definition includes: domain expertise, quality gates, fail-fast enforcement examples, relevant design doc references, and authority level.

---

## 3. The Design-to-Implementation Pipeline

**The most important structural decision:** front-load all design work before agents touch code. Agents are excellent at implementing well-specified plans and terrible at making architectural choices on the fly.

### 3.1 Pipeline Overview

```
High-Level Design (HLD)
  Defines: system architecture, component boundaries, interactions
    |
    v
Low-Level Design (LLD) — per component
  Defines: implementation details, interfaces, data structures, constraints
  References: specific HLD sections
    |
    v
Epic — per implementation scope
  Defines: 2-5 stories, all decisions pre-made, traceable to LLD sections
  Rule: targets ONE implementation domain only
    |
    v
User Story — I.N.V.E.S.T. principles
  Defines: acceptance criteria with verification commands
  Includes: official documentation URLs (versioned)
    |
    v
Implementation — agent executes, human validates
```

### 3.2 Why This Order Matters

Without HLD/LLD, agents make contradictory architectural decisions across stories. Without epics, agents scope-creep or under-deliver. Without acceptance criteria, "done" is undefined.

**The HLD** is your architecture contract. It answers: what are the components, how do they interact, what are the non-negotiable constraints (safety, latency, cost)?

**The LLDs** are per-component blueprints. They answer: what data structures, what interfaces, what algorithms, what performance budgets? Each LLD references specific HLD sections for traceability.

**The Epics** are the bridge between design and implementation. They pre-make every decision the agent would otherwise guess at. Each epic maps to specific LLD sections with line numbers.

**The Stories** are what the agent actually implements. Each story has concrete acceptance criteria with runnable verification commands.

### 3.3 Decision Rule

**ALL decisions must be in the epic before implementation begins.** If something is unclear during implementation: STOP > ASK THE USER > UPDATE THE EPIC > THEN CONTINUE. Agents should never make ad-hoc product or architecture decisions.

### 3.4 Mapping to Scrum

| Scrum Artifact | AI-Assisted Equivalent | Notes |
|---------------|----------------------|-------|
| Product Backlog | Epics derived from LLDs | Epics are pre-designed, not just "user wants X" |
| Sprint Backlog | Stories selected from epics | 1-2 stories per agent session |
| Sprint Goal | Epic completion | One epic per sprint is a good target |
| Definition of Done | Completion verification framework | Scans + acceptance criteria proof + deploy validation |
| Backlog Refinement | Epic writing with LLD traceability | The design work that makes agent execution effective |
| Sprint Review | Completion reports with proof | Commands run, outputs shown, scans clean |

---

## 4. Writing Effective Agent Instructions

### 4.1 Structure

A good instruction file has these sections, roughly in this order:

1. **Mandatory first actions** (re-read instructions, check versions)
2. **Repository boundaries** (what you can and cannot touch)
3. **Critical rules** (fail-fast, no placeholders, security)
4. **Architecture context** (what this system does, how components interact)
5. **Code conventions** (encoding standards, naming, patterns to match)
6. **Build/test/deploy commands** (quick reference)
7. **Completion verification** (mandatory scans, proof templates)
8. **Workflow rules** (one story at a time, escalation triggers)

### 4.2 Rules for Writing Rules

**Be specific about past violations.** "Don't use wrong API versions" is vague. "We've seen EMQX 4.x config syntax used when the project uses EMQX 5.x — always check VERSIONS.yaml first" is actionable.

**Include the scan commands.** Don't just say "no fallback values" — provide the exact grep commands that catch violations, and specify that each match must be reviewed individually.

**State what's forbidden AND what's correct.** Every "don't do X" should have a "do Y instead" with a code example.

**Explain WHY.** Rules with rationale are followed more consistently than arbitrary mandates. "CBOR for MQTT (not JSON) because 30-50% bandwidth reduction on constrained mesh networks" is better than "use CBOR."

**Keep it DRY.** Repo-specific files reference the root file for shared rules. Don't duplicate — it drifts.

### 4.3 The Session Reset Problem

AI agents lose context through compression and across sessions. Your instruction file must account for this:

- Start with "re-read this file after compression or new session"
- List explicit signs that re-reading is needed ("context feels fresh", "user says you're violating rules")
- Keep the most critical rules at the top — what gets read first survives longest

### 4.4 The Companion Agent Instruction File

Alongside this team guide, maintain the companion document **[Agentic-AI-Agent-Instructions.md](./Agentic-AI-Agent-Instructions.md)**, which contains the actual rules loaded into the agent's context. The two files mirror each other:

| Team Guide Section | Agent Instructions Section |
|-------------------|--------------------------|
| "Set up fail-fast enforcement" | "Fail-fast rules you must follow" |
| "Define completion verification" | "Scans you must run before completion" |
| "Establish escalation framework" | "When you must ask the user" |

### 4.5 Evolving Your Instruction Files

Instruction files are living documents. Every time an agent violates a rule that wasn't written down, that violation is a new rule waiting to be added.

**Continuous improvement loop:**

1. **Capture violations.** When an agent makes a mistake — wrong API, silent error, skipped verification — note the exact pattern.
2. **Convert to a rule.** Add a concrete "FORBIDDEN / CORRECT" pair with a code example, not abstract advice.
3. **Add the scan.** If the violation is detectable with grep, add the exact grep command to the mandatory scan suite.
4. **Update both files.** Add the rule to the agent instruction file. Add the rationale and process context to this team guide.
5. **Date or annotate the addition.** Future readers benefit from knowing which rules were added reactively ("we saw this cause X").

**Rule quality benchmark:** If you can show someone the rule and they could both identify a violation and fix it without asking follow-up questions, the rule is good. If not, it needs a code example or concrete scan command.

---

## 5. Defining Agent Roles & Authority

### 5.1 Why Specialize

A single general-purpose agent trying to handle firmware, cloud infrastructure, ML inference, and UI simultaneously produces shallow, inconsistent work. Specialized agents with domain context produce dramatically better results.

### 5.2 Role Categories

**Development Agents** — Write code, implement features. One per domain:

| Agent | Domain | Example Scope |
|-------|--------|--------------|
| Embedded Systems | Firmware, RTOS, hardware interfaces | STM32, FreeRTOS, CAN bus, motor control |
| Edge AI/ML | On-device inference, model optimization | TensorRT, ONNX, LoRA adapters |
| Cloud Infrastructure | Cloud services, IaC | AWS CDK, Lambda, SageMaker, multi-tenant |
| Platform/Backend | Backend services, databases, UI | Go/Python backend, React frontend, PostgreSQL |
| IoT/Networking | Messaging, device connectivity | MQTT, mTLS, device provisioning |
| Robotics | Navigation, localization, control | ROS2, Nav2, SLAM, sensor fusion |

**Review Agents** — Read-only validation. Cannot modify code. Two authority levels:

| Agent | Focus | Authority |
|-------|-------|-----------|
| Safety Compliance | Industry safety standards | **BLOCKING** — can prevent merge |
| Security | Access control, encryption, isolation | **BLOCKING** — can prevent merge |
| Architecture Consistency | Design traceability, fail-fast | **BLOCKING** — can prevent merge |
| Performance | Latency, memory, throughput budgets | Advisory — recommendations only |
| Cost Optimization | Infrastructure cost targets | Advisory — recommendations only |

**Validation Agents** — Run tests, simulate failures:

| Agent | Focus | Authority |
|-------|-------|-----------|
| Integration Test | End-to-end workflow validation | **BLOCKING** — can prevent merge |
| Chaos Engineering | Failure scenario simulation | Advisory |
| Load Test | Scalability validation | Advisory |

**Research Agents** — Explore techniques, build PoCs. No merge authority — proposals require architecture reviewer approval.

### 5.3 Setting Up Agent Definitions

Each agent definition file should contain:

```markdown
# [Agent Name]

## Domain
[What this agent knows about]

## Responsibilities
[What it does]

## Quality Gates
[Measurable thresholds it enforces — e.g., "inference latency < 50ms"]

## Design Doc References
[Which HLD/LLD sections are relevant]

## Fail-Fast Enforcement
[Domain-specific examples of correct vs. incorrect error handling]

## Authority
[BLOCKING / ADVISORY / DEVELOPMENT / RESEARCH]
```

---

## 6. Quality Gates & Review Process

### 6.1 Defining Quality Gates

Every domain needs explicit, measurable quality gates. Vague standards ("code should be performant") don't work with agents. Specific thresholds do.

**Example quality gate table:**

| Domain | Gate | Threshold | Blocking? |
|--------|------|-----------|-----------|
| Safety | Emergency stop response | < 10ms | Yes |
| ML Inference | Per-model latency | < 50ms | Yes |
| Cloud | Per-unit operational cost | < $X/month | No (advisory) |
| Backend | API response time | < 500ms | No (advisory) |
| Security | Tenant isolation | RLS enforced on all queries | Yes |
| Architecture | Design traceability | All code links to LLD | Yes |

### 6.2 The Review Pipeline

After a development agent implements a story, route it through relevant review agents:

```
Implementation complete
  |
  v
[Blocking reviews — ALL must pass]
  - Architecture consistency (traceability, fail-fast, no placeholders)
  - Security (if touching auth, data access, multi-tenant)
  - Safety (if touching control systems, hardware interfaces)
  - Integration tests (if touching cross-component interfaces)
  |
  v
[Advisory reviews — recommendations captured]
  - Performance (if touching hot paths)
  - Cost (if touching cloud resources)
  |
  v
Human final review
  |
  v
Merge
```

**Blocking reviews** must pass or the work goes back for fixes. **Advisory reviews** produce recommendations that are tracked but don't block.

---

## 7. Story Execution Orchestration

This is the operational core — how a single story moves from backlog to done.

### 7.1 The Story Cycle

```
Story selected from sprint backlog
  |
  v
a. PLAN — Task the most relevant specialized agent to generate
   an implementation plan for this story.
  |
  v
b. REVIEW PLAN — Review the plan yourself (or with another agent)
   to validate it covers ALL story requirements and adheres to
   official docs/examples. Return for revision if needed.
  |
  v
c. IMPLEMENT — Agent executes the validated plan.
  |
  v
d. REVIEW IMPLEMENTATION — Task an agent to review the implementation
   against the ORIGINAL STORY acceptance criteria (not the plan).
   This catches plan-vs-story drift.
  |
  v
e. FIX & TEST — Fix issues found in review. Build, deploy, and
   validate that nothing broke. Run completion verification scans.
  |
  v
f. UPDATE TRACKER — Mark story complete with proof. Move to next story.
```

### 7.2 Critical Rules

1. **One story at a time.** Complete fully before starting the next. Max 1-2 stories per agent session — larger scope leads to incomplete work.

2. **Build and deploy after every story** to catch regressions immediately, not after 5 stories when the cause is unclear.

3. **Review against the story, not the plan.** The plan is a means to an end. If the plan was followed perfectly but a story acceptance criterion is unmet, the work is not done.

4. **Decisions go to the human.** If multi-choice options arise, or changes that might impact logic/features, the agent stops and asks. It doesn't pick one and rationalize it later.

5. **Validate content, not just flow.** "Data is flowing" is not the same as "data values are correct." Spot-check actual values against expected.

6. **Static analysis is not verification.** Grepping files, reading code, and code review don't count. Only actual build, run, deploy, and test execution counts as verification.

### 7.3 Session Sizing

A productive agent session handles 1-2 stories. If a story is too large for one session, it's too large — break it down further. Signs a story is too large:
- More than 5-7 files modified
- Touches more than one domain (firmware AND cloud)
- Acceptance criteria that can't be verified in sequence

### 7.4 Multi-Agent Coordination

When multiple agents (or parallel sessions) are active on the same project, coordinate to avoid conflicts and integration surprises.

**Structural rules:**
- **One agent per story.** Assign each story to exactly one development agent. Two agents editing the same file in parallel produces merge conflicts and inconsistent results.
- **Serialize review pipelines.** Run blocking review agents (architecture, security, safety) after each story, not at the end of a batch. Catching a fail-fast violation in story 1 before story 2 starts prevents compounding rework.
- **Ownership at the file level.** If a story requires changes to a shared file (e.g., a central config), only the assigned story's agent should touch it. Other agents wait or work around it.

**Handoff checklist (when passing work from one agent to another):**
1. The handoff story is fully complete — build, deploy, scans clean, acceptance criteria proven.
2. All modified files are committed and the completion report is written.
3. The receiving agent is given the completion report as context, not assumed to have it.

**When multiple agents are blocked on the same dependency:** That's a sequencing error in epic design. Restructure to eliminate the dependency or serialize the dependent stories.

---

## 8. Epic & Story Template Design

### 8.1 Epic Template

Good epic templates encode your quality rules directly, so agents can't miss them.

**Essential sections in an epic template:**

| Section | Purpose |
|---------|---------|
| LLD Reference | Which design doc, which sections (with line numbers) |
| Overview | 2-3 sentence scope description |
| Stories | Ordered list with acceptance criteria and verification commands |
| Official Documentation | Versioned URLs the agent must read BEFORE implementation |
| Non-Negotiable Rules | Fail-fast, no placeholders, spec compliance — repeated in every epic |

### 8.2 Story Template

Each story needs:
- **"As a / I want / So that"** format (keeps it user-value-focused)
- **Acceptance criteria table** with verification commands and expected output
- **Implementation tasks** as checkboxes
- **Official documentation URLs** (versioned — never `/latest/`)

### 8.3 Non-Negotiable Rules Block

Include this in every epic template so it's in context every time an agent works:

```markdown
## Non-Negotiable Implementation Rules:
1. NO Fallbacks or Default Values on errors
2. NO Placeholders or Mock Data
3. NO Error Obfuscation (bare except, silenced exceptions)
4. NO Simplified "for now" Implementations
5. MANDATORY Verification Commands for each criterion
6. ALL Decisions documented in this epic (STOP > ASK if unclear)
7. MANDATORY Official Documentation before implementation
```

### 8.4 Decision Documentation

Every decision made during epic planning must be recorded in the epic itself with rationale. During implementation, if a new decision is needed, the agent stops and asks. The human decides and updates the epic. This creates a complete decision trail.

---

## 9. Specification Ownership & Cross-Team Coordination

### 9.1 The Problem

When multiple teams (or agents) work on components that integrate, specification drift causes integration failures. "I thought the field was called `id`" vs "the spec says `patrol_id`" has caused real bugs.

### 9.2 The Solution

- Maintain a **shared specification repository** as the single source of truth.
- Each spec has a defined **owner** (who can modify) and **reviewers** (who must approve changes).
- **Never modify a spec you don't own** without going through a proposal process.

### 9.3 Change Process

1. Create a proposal using templates in the spec repo
2. Reviewer team adds feedback
3. Iterate until both teams approve
4. Merge to production spec
5. THEN update implementations to match

### 9.4 Agent Rules for Specs

Agents must:
- Read the relevant spec BEFORE writing integration code
- Copy field names exactly — never guess
- Add spec reference comments in code (file path + line numbers)
- If a spec appears wrong: don't change the implementation to "fix" it — create a proposal

---

## 10. Completion Verification Framework

### 10.1 Why This Exists

Agents will claim work is done when it compiles. That's not enough. We've seen:
- Code that compiles but uses wrong field names (spec violation → integration failure)
- Error handlers that catch exceptions and return default values (silent bugs)
- Features that "work" in the agent's mental model but were never actually executed

### 10.2 Mandatory Scans

Define automated scans that catch common agent mistakes. Run BEFORE any story is marked complete.

**Example scan suite (Python projects):**

| Scan | What It Catches |
|------|----------------|
| Placeholder scan | TODO, FIXME, "would implement" stubs |
| Silent exception scan | `except: pass`, bare `except:` |
| Fallback value scan | `.get(key, default)` in error paths |
| Silent return scan | `return None`, `return 0` in error paths |
| Clock domain scan | Using wrong time source |

Provide the exact grep commands in the agent instruction file so there's no ambiguity about what to run.

### 10.3 Runtime Verification

For each acceptance criterion, define a **concrete command** that produces **observable proof**. The agent must run the command and include the actual output (not "expected output") in the completion report.

### 10.4 Deploy Verification

After every story that touches deployed code: build, deploy, and run a smoke test to validate nothing broke. This catches integration regressions immediately.

### 10.5 Completion Report

Require a structured completion report with:
- Acceptance criteria: command run + actual output for each
- Scan results: output of all mandatory scans
- Build/deploy: proof of successful build and deployment
- Files modified: list with line counts

---

## 11. Escalation Framework

### 11.1 Agent-to-Human Escalation

Define clear triggers for when the agent must stop and ask the human:

| Trigger | Why |
|---------|-----|
| Multi-choice design decisions | Agents should not make product decisions |
| Changes impacting logic/features beyond the story | Scope creep prevention |
| Blocking review agent finds violations | Human decides: fix vs. override vs. defer |
| Spec appears wrong or incomplete | Cross-team coordination needed |
| Scan result that might be legitimate | Agent should not self-rationalize dismissals |
| Cost exceeds defined thresholds | Business decision required |
| Multiple agents disagree | Human breaks the tie |
| Task requires product/design decisions | Agent should ASK, not guess |
| Implementation requires a new dependency | Human approves library choices; don't add without asking |

### 11.2 The Anti-Pattern to Prevent

The most dangerous agent behavior is **confident incorrectness** — making a decision, rationalizing it, and moving on. The escalation framework exists to convert these moments into human checkpoints.

---

## 12. Common Pitfalls & Lessons Learned

### 12.1 Agent-Specific Pitfalls

| Pitfall | Cause | Prevention |
|---------|-------|-----------|
| Uses outdated API syntax | Training data is stale | Version file + doc fetching tools + "read codebase first" rule |
| Blanket-dismisses scan results | Eager to report "clean" | Require individual review of each match + user confirmation for dismissals |
| Claims completion without running code | Optimizes for speed | "Static analysis is not verification" rule + runtime proof required |
| Makes architectural decisions mid-story | No epic specified the answer | "ALL decisions in epic" rule + escalation on ambiguity |
| Silently falls back on errors | Training data is full of try/except patterns | Fail-fast philosophy + violation scans + code examples of correct patterns |
| Modifies files in other repos | Doesn't understand boundaries | Explicit boundary rules per repo |
| Adds backwards compatibility code speculatively | Training data emphasizes it | "Only what the story specifies" rule + escalation when story is silent on compat |
| Forgets rules after context compression | Context window limitations | "Re-read instructions" rule at top of every instruction file |
| Writes tests that only validate its own assumptions | Tests are generated from the same mental model as the code | Require tests to verify acceptance criteria commands from the story, not just exercise the implementation; human spot-checks edge cases |
| Adds a new dependency without asking | Easiest path to solving the problem | Explicit escalation trigger: "new dependency required → ASK first" |

### 12.2 Process Pitfalls

| Pitfall | Cause | Prevention |
|---------|-------|-----------|
| Stories too large | Insufficient breakdown | Max 5-7 files, one domain, one session |
| Epics missing decisions | Rushed planning | "ALL decisions in epic" rule, STOP > ASK > UPDATE |
| No spec for integration points | Teams worked independently | Shared spec repo with ownership and change process |
| Review agents ignored | Blocking feels slow | Clear authority levels, advisory vs blocking distinction |
| No deploy verification between stories | "I'll test everything at the end" | Mandatory build+deploy after every story |
| Agent picks a design option without asking | Escalation triggers unclear | Explicit escalation trigger list in agent instructions |

### 12.3 Key Lessons

1. **Front-load design.** Every hour spent on HLD/LLD/Epic saves 5+ hours of agent rework.
2. **Specificity beats volume.** One concrete "do X not Y" example beats a page of abstract principles.
3. **Past violations are the best rules.** Document what went wrong and turn it into a rule with the specific grep/scan that would have caught it.
4. **Agents don't remember across sessions.** Design your instruction files assuming every session starts fresh.
5. **Verify verify verify.** The completion verification framework is the single highest-ROI practice in this guide.
6. **Evolve your instruction files continuously.** Every new agent mistake is a rule waiting to be written. The team guide and agent instruction file should grow after every sprint based on violations found — see §4.5.

---

*This guide is designed to be adapted. Remove what doesn't apply to your project, add your domain-specific rules, and evolve it as you learn what your agents get wrong. The companion document, **[Agentic-AI-Agent-Instructions.md](./Agentic-AI-Agent-Instructions.md)**, contains the rules that get loaded into agent context — maintain them in parallel.*
