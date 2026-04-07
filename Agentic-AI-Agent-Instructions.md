# Agent Instructions

## Rules for AI Agents in Software Development

*This document is loaded into your context at the start of every session. Every rule exists because its absence caused a real bug, integration failure, or wasted work. Follow them exactly.*

*Companion document: [**Agentic-AI-Guide-For-Teams.md**](./Agentic-AI-Guide-For-Teams.md) (for humans setting up and orchestrating AI-assisted development).*

---

## Table of Contents

1. [Session & Context Management](#1-session--context-management)
2. [Version & API Trust](#2-version--api-trust)
3. [Fail-Fast Philosophy](#3-fail-fast-philosophy)
4. [No Placeholders](#4-no-placeholders)
5. [Repository Boundaries](#5-repository-boundaries)
6. [Destructive Operations](#6-destructive-operations)
7. [Security](#7-security)
8. [Specification Compliance](#8-specification-compliance)
9. [Code Conventions](#9-code-conventions)
10. [Your Role in Story Execution](#10-your-role-in-story-execution)
11. [Completion Verification](#11-completion-verification)
12. [When to Escalate](#12-when-to-escalate)

---

## 1. Session & Context Management

Context compression and new sessions cause you to "forget" rules. This section exists to counter that.

**At the START of every session or after context compression:**
1. Re-read this instruction file.
2. Re-read the repo-specific instruction file for the repo you're working in.
3. Check the project's version file for current component versions.

**Signs you need to re-read:**
- Starting work in a repo you haven't touched recently.
- The user says "you're violating the rules again."
- Context feels "fresh" or you're unsure of project standards.

---

## 2. Version & API Trust

**Your training data is OUTDATED. You MUST verify current versions before writing code.**

```
WRONG: "I know [library] version X uses [API]..."
CORRECT: Check the project's version file, fetch current docs, read existing codebase patterns.
```

**Before using ANY library, framework, or tool:**
1. Check the project's version pinning file for the pinned version.
2. Use documentation fetching tools to get current docs for that version.
3. Read existing code in the repo to match established patterns.
4. **When in doubt, ASK the user** rather than guessing.

**This has caused real bugs:**
- Using config syntax for version X when the project uses version Y.
- Using deprecated API patterns from training data.
- Assuming features exist in the pinned version that were added later.

---

## 3. Fail-Fast Philosophy

Missing config = immediate failure. Error in a required path = raise, don't suppress. Precondition not met = don't start the pipeline.

### 3.1 Forbidden Patterns

**Silent error swallowing:**
```python
# FORBIDDEN
except:
    pass

# FORBIDDEN
except:
    log.error("failed")
    return default_value
```

**Default value substitution in error paths:**
```python
# FORBIDDEN - hides missing required config
config = os.environ.get('REQUIRED_VAR', 'default-value')
```

**Silent return on missing data:**
```python
# FORBIDDEN - in required data paths
if data is None:
    return 0  # or return [], None, ""
```

### 3.2 Correct Patterns

```python
# CORRECT - fail immediately on missing config
config = os.environ['REQUIRED_VAR']  # KeyError if missing

# CORRECT - explicit validation
if not config:
    raise ValueError("REQUIRED_VAR environment variable required")

# CORRECT - propagate exceptions
try:
    result = process()
except SpecificException as e:
    logger.error(f"Processing failed: {e}")
    raise  # Propagate to caller

# CORRECT - gate pipeline activation
if not calibration_complete:
    raise RuntimeError("Cannot start: calibration incomplete")
```

### 3.3 `return None` Rules

**Valid uses of `return None`:**
1. Lookup/search that may legitimately find no match (e.g., `get_item_by_name`).
2. Correct semantic result (e.g., optional enrichment data not available).
3. Test/diagnostic scripts, not production data paths.

**NEVER valid uses of `return None`:**
1. Preconditions not met (sync, calibration, warmup) — gate the pipeline instead.
2. Hardware failure without consecutive error tracking — track and declare dead.
3. Missing data in a required data path — raise or close the gate.

### 3.4 Forbidden Rationalizations

These all mask the same anti-pattern — do not use them:
- "Data isn't ready yet, so we skip it" — **Don't start until data IS ready.**
- "Warming up / calibrating" — **Don't call until calibration IS done.**
- "Buffer is empty at startup" — **Don't start processing until buffer has data.**
- "This is transient" — **Track consecutive failures. Declare dead after threshold.**

The correct pattern: **validate preconditions BEFORE activating the pipeline.** Once active, precondition failures are bugs.

### 3.5 Reviewing Scan Results

- **NEVER blanket-dismiss scan results.** "These are all legitimate because they're in [path type]" is FORBIDDEN.
- Review EACH match individually: Can this case happen at runtime? If yes — is it a config bug (validate at init) or a data bug (raise, don't drop)?
- If you believe a match is legitimate, you MUST get **explicit user confirmation** before dismissing it.
- Logging an error and returning a fallback value is NOT fail-fast. Fail-fast means the error is **impossible to ignore**.

---

## 4. No Placeholders

Either implement fully or raise `NotImplementedError`.

- No `TODO` comments, no `FIXME`, no mock data in production code.
- No "would implement", "actual implementation", "in production" comment stubs.
- No "simplified for now" implementations.

**This applies to CODE only.** When a task requires product or design decisions, **ASK the user** rather than guessing.

---

## 5. Repository Boundaries

**NEVER modify code outside your assigned repository.**

If you find a bug in another repository:
1. **STOP** — do not attempt to fix it.
2. **DOCUMENT** the issue with full details (file, line, what's wrong, suggested fix).
3. **REPORT** to the user.

If a specification appears wrong:
1. Do NOT change your implementation to "fix" the spec.
2. Report to the user so they can follow the spec change process.

---

## 6. Destructive Operations

**NEVER perform destructive operations without EXPLICIT user permission:**

- Deleting files, databases, tables, buckets, or certificates.
- Revoking credentials or certificates.
- Modifying user accounts.
- Dropping cloud resources.
- Force-pushing or resetting git history.

**When troubleshooting:**
- **DIAGNOSE** root cause first — don't guess.
- **ASK** before taking destructive actions.
- **VERIFY** assumptions before acting.

```
WRONG: "Deploy failed, let me delete and recreate the resource."
CORRECT: "Deploy failed — checking logs for the specific error."
```

---

## 7. Security

- **SQL Injection:** Use parameterized queries for ALL data values. No string formatting.
- **Credentials:** NEVER hardcode. Use IAM roles, environment variables, or secret management.
- **Auth failures:** Invalid token → 401 (no fallback). Missing permissions → 403 (no defaults). Missing config → startup failure.
- **Multi-tenant:** Every query must be scoped to the current tenant. RLS enforced at the database level.
- **Secrets:** Never in source control. Use secret management services.

---

## 8. Specification Compliance

**Before writing ANY integration code (message structures, API schemas, protocol implementations):**

1. **Read the relevant spec FIRST.** Find your topic/endpoint in the spec file.
2. **Copy field names EXACTLY.** NEVER guess — copy from spec verbatim.
3. **Add a spec reference comment** in code with file path and line numbers.
4. **Verify before commit.** Diff your field names against the spec.

```go
// Per specs/mqtt/SYNC_PROTOCOL.md lines 210-222:
// patrol_id, name, waypoint_ids (not full waypoints)
type PatrolPayload struct {
    PatrolID    string   `json:"patrol_id"`   // NOT "id"
    Name        string   `json:"name"`
    WaypointIDs []string `json:"waypoint_ids"` // NOT []Waypoint
}
```

**If the spec appears wrong:** Do NOT change your implementation to "fix" it. Report to the user.

**Past violations that caused integration failures:**
- `id` vs `patrol_id` mismatch — consumer couldn't parse messages.
- `waypoints` (full objects) vs `waypoint_ids` (references only) — wrong data structure sent.

---

## 9. Code Conventions

- **Portable shell:** Use POSIX syntax (`counter=$((counter + 1))`), not bash-specific (`((counter++))`).
- **API responses:** Return empty arrays `[]`, never `null` for list endpoints.
- **Encoding:** Follow the project's encoding standard per channel (e.g., CBOR for internal messaging, JSON for HTTP APIs). Never mix encodings on the same channel.
- **Imports:** Match existing codebase patterns before introducing new ones.
- **No backwards compatibility code** (unless the story explicitly requires it): No fallback logic for old formats, no migration paths, no deprecated field handling unless specified. Implement exactly what the story specifies, and delete old code it replaces. If the story is silent on compatibility: **ASK** rather than adding it speculatively.

---

## 10. Your Role in Story Execution

You operate within a structured story-by-story process. Your role depends on where you are in the cycle:

### When PLANNING (step a):
- Generate an implementation plan for the assigned story.
- The plan must cover ALL acceptance criteria in the story.
- Reference official documentation (versioned URLs, not `/latest/`).
- Reference relevant design documents (HLD/LLD sections with line numbers).

### When IMPLEMENTING (step c):
- Follow the validated plan.
- If something is unclear or a decision is needed: **STOP and ASK the user.** Do not make ad-hoc decisions.
- If you encounter a multi-choice situation: **STOP and ASK.**
- If the change might impact logic/features beyond this story: **STOP and ASK.**

### When REVIEWING (step d):
- Review against the **original story acceptance criteria**, not the implementation plan.
- For each acceptance criterion: is it fully implemented? Can it be verified with the specified command?
- Check for fail-fast violations, placeholder code, spec compliance.

### When FIXING & TESTING (step e):
- Fix all issues found in review.
- Build and deploy. Validate nothing broke.
- Run all completion verification scans (Section 11).
- Verify each acceptance criterion with the specified command and capture actual output.

### Workflow Rules:
- **One story at a time.** Complete fully before starting the next.
- **Maximum 1-2 stories per session.** Larger scope leads to incomplete work.
- **Build and deploy after every story** to catch regressions.
- **Never mark complete without verification.** See Section 11.

---

## 11. Completion Verification

**Before marking ANY story or task as complete, you MUST do ALL of the following:**

### 11.1 Run Mandatory Scans

```bash
# Placeholder/TODO scan — fix ALL matches in code you wrote or modified
grep -rn "placeholder\|TODO\|FIXME\|would.*implement\|actual.*implementation\|in production" \
  <modified_directories> --include="*.go" --include="*.ts" --include="*.tsx" --include="*.py"
```

```bash
# Fail-fast violation scan (Python) — review EACH match individually

# Silent error swallowing
grep -rn "except.*pass$\|except:$" <modified_files> --include="*.py"

# Clock domain substitution
grep -rn "get_clock().now()\|time\.time()\|datetime\.now()" <modified_files> --include="*.py"

# Default value substitution in error paths
grep -rn "\.get(.*,\s*0)\|\.get(.*,\s*\"\")\|\.get(.*,\s*False)\|\.get(.*,\s*None)" <modified_files> --include="*.py"

# Silent return on missing data
grep -rn "return 0$\|return 0\.0$\|return \[\]$\|return None$\|return \"\"$" <modified_files> --include="*.py"

# Bare except
grep -rn "except:" <modified_files> --include="*.py"
```

**For EACH match:** Is it a genuine violation? If in doubt, it IS a violation — fix it. If you believe it's legitimate, get **explicit user confirmation** before dismissing.

### 11.2 Verify Each Acceptance Criterion

- Run the concrete verification command specified in the story.
- Capture the **actual output** (not "expected output").
- If you can't demonstrate it works, it is **NOT complete**.

**Static analysis is NOT verification.** Grepping files, reading code, and code review don't count. Only actual build, run, deploy, and test execution counts.

### 11.3 Build & Deploy Verification

After every story that touches deployed code:
- Build the project.
- Deploy to the test environment.
- Run a smoke test to confirm nothing broke.

### 11.4 Provide Proof

Your completion report MUST include:

```
## Story [ID] Complete

### Acceptance Criteria Verification:
1. "[Criterion text]"
   - Command: `<actual command run>`
   - Output: <actual output>

2. "[Criterion text]"
   - Command: `<actual command run>`
   - Output: <actual output>

### Placeholder Scan:
$ grep -rn "placeholder\|TODO\|FIXME" path/to/modified/files
(no output - clean)

### Fail-Fast Violation Scan:
$ grep -rn "except.*pass$\|except:$" path/to/modified/files
(no output - clean)
$ grep -rn "return 0$\|return None$" path/to/modified/files
(no output - clean, OR each match reviewed and confirmed with user)

### Build & Deploy:
$ <build command>
(success output)
$ <deploy/test command>
(success output)

### Files Modified:
- path/to/file.ext (+lines, -lines)
```

---

## 12. When to Escalate

**ALWAYS stop and ask the user when:**

| Situation | Why |
|-----------|-----|
| Multi-choice design decisions arise | You should not make product decisions |
| Changes might impact logic or features beyond this story | Scope creep prevention |
| A blocking review finds violations | Human decides: fix, override, or defer |
| A specification appears wrong or incomplete | Cross-team coordination needed |
| A scan result seems legitimate but you're not sure | Don't self-rationalize dismissals |
| The task requires a product or design decision | Don't guess — ASK |
| You're uncertain about a convention or pattern | Read codebase first, then ask if still unclear |
| Implementation requires adding a new dependency | Human approves library choices; don't add without asking |

**NEVER do this:** Make a decision, rationalize it, and move on. That is the single most damaging pattern in AI-assisted development. When in doubt, **ask**.

---

*These rules are non-negotiable. They exist because their absence caused real failures. Follow them exactly, and ask the user if anything is unclear.*
