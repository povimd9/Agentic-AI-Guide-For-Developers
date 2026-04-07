# Decision Log Template

Use this template when a design, planning, or implementation choice requires explicit human approval and future readers need to understand why a path was chosen.

---

## Decision Metadata

- **Decision ID:** DECISION_<XX>
- **Date:** YYYY-MM-DD
- **Status:** Proposed | Approved | Superseded
- **Owner:** <Team or person accountable for the decision>
- **Related HLD/LLD/Epic:** <links with section references and line ranges>

---

## Context

<What problem or ambiguity triggered this decision? What constraint or tradeoff matters here?>

## Options Considered

### Option A — <Name>
- **Summary:** <What it is>
- **Pros:** <Benefits>
- **Cons:** <Costs, risks, operational concerns>

### Option B — <Name>
- **Summary:** <What it is>
- **Pros:** <Benefits>
- **Cons:** <Costs, risks, operational concerns>

Add more options only if they were seriously considered.

## Decision

- **Chosen Option:** <Name>
- **Rationale:** <Why this was selected over the alternatives>
- **Explicit Human Approval:** <Who approved it and when>

## Impact

- **Implementation impact:** <Files/components/stories affected>
- **Testing/verification impact:** <What must be proved after implementation>
- **Operational impact:** <Runtime, deployment, observability, support implications>

## Follow-Up

- [ ] Epic/LLD updated with the approved decision
- [ ] Specs updated or proposal created if integration contracts are affected
- [ ] Verification commands updated to reflect the choice

---

## Sample Entry

- **Decision ID:** DECISION_01
- **Date:** 2025-03-06
- **Status:** Approved
- **Owner:** Platform Team
- **Related HLD/LLD/Epic:** `SAMPLE_HLD.md` §4.2 (lines 120-148), `SAMPLE_LLD_01_Authentication.md` §3.4 (lines 88-121), `SAMPLE_EPIC_01_User_Auth.md` (lines 1-180)

### Context

TaskFlow needs a compact over-the-wire format for device telemetry while keeping the public REST API easy for web and mobile teams to debug and adopt.

### Options Considered

#### Option A — JSON for REST and MQTT
- **Summary:** Use JSON everywhere for consistency.
- **Pros:** Simpler tooling; easy to inspect by hand.
- **Cons:** Higher payload size on constrained links; redundant parsing costs for device traffic.

#### Option B — JSON for REST, CBOR for MQTT
- **Summary:** Keep human-facing APIs in JSON and use CBOR for device messaging.
- **Pros:** Smaller payloads for constrained networks; preserves readable REST APIs for external consumers.
- **Cons:** Requires dual serialization support and stronger spec discipline.

### Decision

- **Chosen Option:** Option B — JSON for REST, CBOR for MQTT
- **Rationale:** REST consumers prioritize readability and interoperability, while MQTT traffic benefits from lower bandwidth overhead. The tradeoff is acceptable because the serialization boundary is explicit and spec-owned.
- **Explicit Human Approval:** Platform lead approved on 2025-03-06 during design review.

### Impact

- **Implementation impact:** MQTT publishers/subscribers must use CBOR codecs; REST handlers remain JSON-only.
- **Testing/verification impact:** Add payload-format verification for MQTT topics and keep REST contract tests on JSON responses.
- **Operational impact:** Lower network usage on constrained links; observability tooling must decode CBOR in MQTT traces.

### Follow-Up

- [x] Epic/LLD updated with the approved decision
- [x] Specs updated or proposal created if integration contracts are affected
- [x] Verification commands updated to reflect the choice
