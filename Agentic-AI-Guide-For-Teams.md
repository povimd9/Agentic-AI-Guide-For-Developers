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

AI agents are powerful generators — of code, designs, and documentation — but poor decision-makers. Left unguided, they produce plausible output that silently violates architectural invariants, swallows errors behind fallback values, and claims completion without verification. The practices in this guide exist to channel that power into reliable, high-quality output at every stage of the development lifecycle — from high-level design through implementation.

**The human role is to direct, review, and decide.** You provide the vision, constraints, and judgment calls. Agents do the heavy lifting of generating, structuring, and implementing — at every layer. You don't need to write HLDs, LLDs, or epics by hand; you need to tell the agent what to build, review what it produces, and own every decision.

**Never assume an agent-generated deliverable is correct.** Agents produce plausible, well-structured output that can contain subtle errors: wrong assumptions baked into an HLD, missing failure modes in an LLD, acceptance criteria that don't actually prove the feature works. Every layer of agent output must be reviewed by a human who understands the domain well enough to catch what's wrong — not just confirm that it looks reasonable.

### Core Principles

1. **Design before implementation.** Agents generate design documents and implementation plans well when given clear direction. They make architectural *decisions* poorly. Use agents to draft HLDs, LLDs, and epics — but the human must provide the direction, review the output, and own every decision.

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

**The most important structural decision:** complete each design layer before moving to the next. Agents are excellent at generating structured documents and implementing well-specified plans. They are terrible at making architectural *decisions* on the fly. Use agents at every layer — but always with human direction and decision ownership.

### 3.1 Pipeline Overview

**Every layer in this pipeline can and should be agent-generated,** with the human providing direction, making decisions, and validating the output.

```
High-Level Design (HLD)
  Agent generates from: human's system description, constraints, goals
  Human owns: architecture decisions, component boundaries, non-negotiable constraints
  Defines: system architecture, component boundaries, interactions
    |
    v
Low-Level Design (LLD) — per component
  Agent generates from: HLD sections + human's component requirements
  Human owns: interface decisions, algorithm choices, performance budget tradeoffs
  Defines: implementation details, interfaces, data structures, constraints
  References: specific HLD sections
    |
    v
Epic — per implementation scope
  Agent generates from: LLD sections + human's scope/priority decisions
  Human owns: scope decisions, story prioritization, acceptance criteria approval
  Defines: 2-5 stories, all decisions pre-made, traceable to LLD sections
  Rule: targets ONE implementation domain only
    |
    v
User Story — I.N.V.E.S.T. principles
  Agent generates from: epic context + LLD traceability
  Human owns: acceptance criteria sign-off, verification command approval
  Defines: acceptance criteria with verification commands
  Includes: official documentation URLs (versioned)
    |
    v
Implementation — agent executes, human validates
```

### 3.2 Why This Order Matters

Without HLD/LLD, agents make contradictory architectural decisions across stories. Without epics, agents scope-creep or under-deliver. Without acceptance criteria, "done" is undefined.

**The HLD** is your architecture contract. Tell the agent your system goals, constraints, and key components — it generates the document. You review and decide: are these the right boundaries? The right interactions? The right constraints? The agent structures; you approve.

**The LLDs** are per-component blueprints. Point the agent at the relevant HLD sections and describe what the component needs to do — it generates the detailed design. You review and decide: are these the right interfaces? The right algorithms? The right performance budgets?

**The Epics** are the bridge between design and implementation. Point the agent at the LLD sections and tell it to break the work into stories — it generates the epic with acceptance criteria and verification commands. You review and decide: is the scope right? Are the acceptance criteria sufficient? Are all decisions captured?

**The Stories** are what the agent implements. Each story has concrete acceptance criteria with runnable verification commands — generated in the epic, approved by you.

### 3.3 Mandatory Human Review at Every Layer

**Agent output is a draft until a human approves it.** No layer's output should flow into the next without human review. This is not optional — skipping review at design layers compounds errors into implementation.

| Layer | What the Human Must Verify | Common Agent Mistakes to Catch |
|-------|---------------------------|-------------------------------|
| **HLD** | Component boundaries make sense; constraints are correct and complete; nothing critical is missing from non-goals | Invents plausible-sounding components that aren't needed; omits constraints the human mentioned verbally but didn't write down; under-specifies security or multi-tenancy |
| **LLD** | Interfaces match what consumers actually need; failure modes are realistic, not theoretical; performance budgets are achievable | Generates symmetrical/clean interfaces that don't match real usage patterns; lists failure modes without practical mitigations; copies performance numbers from training data instead of your actual constraints |
| **Epic** | Stories cover all LLD requirements; acceptance criteria actually prove the feature works (not just exercise it); all decisions are captured with rationale | Writes acceptance criteria that test the happy path but miss edge cases; leaves implicit decisions undocumented ("obvious" choices that aren't obvious); generates verification commands that pass even when the feature is broken |
| **Story** | Verification commands are runnable and meaningful; implementation tasks are complete; LLD traceability is correct | References wrong LLD sections; generates curl commands that test the wrong endpoint; omits tasks that are "obvious" but will be forgotten |

**The review question at every layer is: "If I hand this to an agent that follows it literally, will the result be correct?"** If the answer is "only if the agent also understands X" — then X needs to be in the document.

### 3.4 Decision Rule

**ALL decisions must be in the epic before implementation begins.** Agents generate the design documents and epics, but humans must review and approve every decision within them. If something is unclear during any stage — design, planning, or implementation — the agent must STOP > ASK THE USER > UPDATE THE DOCUMENT > THEN CONTINUE. Agents draft; humans decide.

### 3.5 Mapping to Scrum

| Scrum Artifact | AI-Assisted Equivalent | Notes |
|---------------|----------------------|-------|
| Product Backlog | Epics derived from LLDs | Agent-generated from LLDs, human-approved |
| Sprint Backlog | Stories selected from epics | 1-2 stories per agent session |
| Sprint Goal | Epic completion | One epic per sprint is a good target |
| Definition of Done | Completion verification framework | Scans + acceptance criteria proof + deploy validation |
| Backlog Refinement | Agent-assisted epic generation from LLDs | Human provides scope/priority; agent drafts; human reviews |
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

This is the operational core — how a single story moves from backlog to done. Note that by this point, the HLD, LLDs, and epics have already been agent-generated and human-approved. The story cycle is the execution phase.

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

Good epic templates serve two purposes: they guide the agent that *generates* the epic (ensuring all required sections are filled in), and they guide the agent that later *implements* the stories (encoding quality rules directly so they can't be missed).

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

Every decision must be recorded in the epic with rationale. When an agent generates an epic, it should flag any point where multiple valid approaches exist and present them to the human for decision — not pick one silently. The human decides, the agent records the decision and rationale in the epic. During implementation, if a new decision surfaces, the same rule applies: the agent stops and asks. This creates a complete decision trail where every judgment call is human-owned.

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

### 11.2 The Anti-Patterns to Prevent

**On the agent side:** The most dangerous behavior is **confident incorrectness** — making a decision, rationalizing it, and moving on. The escalation framework exists to convert these moments into human checkpoints.

**On the human side:** The most dangerous behavior is **uncritical approval** — glancing at agent output, seeing that it's well-structured and plausible, and approving it without verifying the substance. Agent-generated designs are especially dangerous because they *look* thorough. A polished HLD with clean diagrams and complete sections can still have wrong component boundaries. A detailed LLD can have interfaces that don't match what consumers actually need. An epic with well-formatted acceptance criteria can have verification commands that pass even when the feature is broken. Your review is the last line of defense before errors compound into the next layer.

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
| Humans write all design docs manually | Underusing agent capabilities | Agents generate HLDs/LLDs/epics from templates; humans direct and decide |
| Rubber-stamping agent-generated designs | Agent output looks polished and complete | Review every layer critically — see §3.3 for what to check at each level |

### 12.3 Key Lessons

1. **Front-load design — with agents.** Use agents to generate HLDs, LLDs, and epics from your direction. Every hour spent on agent-assisted design saves 5+ hours of implementation rework. You don't write these documents by hand — you direct, review, and decide.
2. **Specificity beats volume.** One concrete "do X not Y" example beats a page of abstract principles.
3. **Past violations are the best rules.** Document what went wrong and turn it into a rule with the specific grep/scan that would have caught it.
4. **Agents don't remember across sessions.** Design your instruction files assuming every session starts fresh.
5. **Verify verify verify.** The completion verification framework is the single highest-ROI practice in this guide.
6. **Review is not optional — at any layer.** Agent-generated HLDs, LLDs, and epics look polished even when they contain wrong assumptions, missing failure modes, or acceptance criteria that don't actually prove anything. The more competent the output looks, the harder you must look for what's wrong. See §3.3.
7. **Evolve your instruction files continuously.** Every new agent mistake is a rule waiting to be written. The team guide and agent instruction file should grow after every sprint based on violations found — see §4.5.

---

*This guide is designed to be adapted. Remove what doesn't apply to your project, add your domain-specific rules, and evolve it as you learn what your agents get wrong. Use agents at every layer — design, planning, implementation, review — and reserve human effort for what only humans can do: providing direction, making decisions, and owning the outcome. The companion document, **[Agentic-AI-Agent-Instructions.md](./Agentic-AI-Agent-Instructions.md)**, contains the rules that get loaded into agent context — maintain them in parallel.*
