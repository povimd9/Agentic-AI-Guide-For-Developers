# LLD #<XX>: <Component Title>

**Version:** <X.Y.Z>
**Status:** Draft | Review | Approved
**Last Updated:** <YYYY-MM-DD>
**Owner:** <Team or individual>
**HLD Reference:** §<X.X> <Section Title>

---

## 1. Purpose and Scope

### What This Component Implements
<Brief description of what this component does.>

### What It Does NOT Implement
<Explicit exclusions to prevent scope creep.>

### HLD References
- §<X.X> <Section Title> — <what aspect>
- §<X.X> <Section Title> — <what aspect>

---

## 2. Context and Dependencies

### Upstream Dependencies

| Component | What It Provides | Interface |
|-----------|-----------------|-----------|
| <Component> | <Data/service it provides> | <API/protocol> |

### Downstream Consumers

| Component | What It Consumes | Interface |
|-----------|-----------------|-----------|
| <Component> | <Data/service it uses from us> | <API/protocol> |

### External Dependencies

| Dependency | Version | Purpose |
|------------|---------|---------|
| <Library/service> | <version from VERSIONS.yaml> | <why needed> |

---

## 3. Interfaces and Contracts

### 3.1 API Endpoints (if applicable)

```
<HTTP method> <path>
  Request:  <schema or TypeScript type>
  Response: <schema or TypeScript type>
  Errors:   <error codes and meanings>
```

### 3.2 Events / Messages (if applicable)

```
Event: <event_name>
  Payload: <schema>
  Frequency: <rate>
  Publisher: <who sends>
  Subscribers: <who receives>
```

### 3.3 Error Model

<How errors are represented, propagated, and handled at boundaries.>

---

## 4. Data Models and Persistence

### 4.1 Database Schema

```sql
-- <Table name and purpose>
CREATE TABLE <table_name> (
    <columns with types and constraints>
);
```

### 4.2 Indexes

| Table | Index | Columns | Purpose |
|-------|-------|---------|---------|
| <table> | <index_name> | <columns> | <why needed> |

### 4.3 Retention and Lifecycle

| Data Type | Retention | Archive Strategy |
|-----------|-----------|------------------|
| <type> | <period> | <what happens after> |

---

## 5. Detailed Design

### 5.1 Architecture

```
<Component-internal architecture diagram.
Show internal modules, data flow, key abstractions.>
```

### 5.2 Algorithms / State Machines

<Describe key algorithms, state transitions, or decision logic.>

### 5.3 Resource Budgets

| Resource | Budget | Measurement |
|----------|--------|-------------|
| CPU | <target> | <how measured> |
| Memory | <target> | <how measured> |
| Disk | <target> | <how measured> |
| Network | <target> | <how measured> |

### 5.4 Concurrency Model

<Thread/process model, connection pooling, backpressure strategy.>

### 5.5 Configuration

| Config Key | Type | Default | Description |
|------------|------|---------|-------------|
| <key> | <type> | <value or REQUIRED> | <purpose> |

---

## 6. Timing and Performance

### 6.1 Latency Budgets

| Operation | Target (p50) | Target (p95) | Target (p99) |
|-----------|-------------|-------------|-------------|
| <operation> | <ms> | <ms> | <ms> |

### 6.2 Throughput

| Metric | Steady State | Peak |
|--------|-------------|------|
| <metric> | <rate> | <rate> |

---

## 7. Reliability and Failure Handling

### 7.1 Failure Modes

| Failure | Detection | Impact | Mitigation |
|---------|-----------|--------|------------|
| <what can fail> | <how detected> | <user impact> | <recovery action> |

### 7.2 Retry and Backoff

<Retry strategy for transient failures. Which operations are idempotent?>

---

## 8. Security and Privacy

### 8.1 Authentication / Authorization

<How this component validates identity and permissions.>

### 8.2 Data Classification

| Data Element | Classification | Handling |
|-------------|---------------|----------|
| <field> | <PII / Confidential / Public> | <encryption, masking, etc.> |

---

## 9. Invariant Compliance

Map to HLD §<X> Critical Invariants:

| Invariant | How This Component Complies | Verification |
|-----------|---------------------------|--------------|
| <invariant from HLD> | <how enforced here> | <test or check> |

---

## 10. Observability

### Metrics

| Metric Name | Type | Labels | Alert Threshold |
|-------------|------|--------|----------------|
| <metric> | Counter/Gauge/Histogram | <labels> | <when to alert> |

### Logs

| Event | Level | When Emitted |
|-------|-------|-------------|
| <event> | INFO/WARN/ERROR | <condition> |

### Health Checks

| Check | Endpoint | Criteria |
|-------|----------|----------|
| Liveness | <path> | <what it checks> |
| Readiness | <path> | <what it checks> |

---

## 11. Validation and Acceptance

### Test Cases

| Test | Input | Expected Output | Type |
|------|-------|-----------------|------|
| <test> | <input> | <expected> | Unit/Integration |

### Benchmarks

| Benchmark | Pass Criteria |
|-----------|--------------|
| <benchmark> | <threshold> |

---

## 12. Operational Procedures

### Maintenance

<Recurring maintenance tasks: log rotation, cache cleanup, etc.>

### Runbook

<Key operational procedures: restart, failover, data recovery.>

---

## 13. Risks and Open Questions

| Risk / Question | Severity | Mitigation / Status |
|----------------|----------|---------------------|
| <risk> | High/Medium/Low | <mitigation or "Open"> |

---

## Appendices

<Diagrams, sequence charts, example payloads, glossary.>
