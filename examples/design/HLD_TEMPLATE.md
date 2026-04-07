# <Project Name> — High-Level Design

**Version:** <X.Y.Z>
**Status:** Draft | Review | Approved
**Last Updated:** <YYYY-MM-DD>
**Owner:** <Team or individual>

---

## 1. Executive Summary

<2-3 sentences describing the system's purpose and key value proposition.>

**Key Characteristics:**

| Attribute | Value |
|-----------|-------|
| System Type | <e.g., Web SaaS, IoT Platform, Mobile App> |
| Users | <Target audience> |
| Scale Target | <e.g., 10K users, 1M requests/day> |
| Deployment | <e.g., AWS, self-hosted, hybrid> |

---

## 2. System Purpose & Goals

### 2.1 Goals

| Goal | Description | Measurable Target |
|------|-------------|-------------------|
| <Goal 1> | <What it achieves> | <How to measure success> |
| <Goal 2> | <What it achieves> | <How to measure success> |

### 2.2 Non-Goals

| Non-Goal | Rationale |
|----------|-----------|
| <What this system does NOT do> | <Why it's excluded> |

---

## 3. Architecture Overview

### 3.1 System Diagram

```
<ASCII box diagram showing major components and their connections.
Use clear labels and directional arrows.>

┌──────────┐     HTTPS      ┌──────────┐     TCP       ┌──────────┐
│  Client  │ ──────────────► │   API    │ ────────────► │ Database │
│  (Web)   │                 │ Gateway  │               │ (SQL)    │
└──────────┘                 └──────────┘               └──────────┘
```

### 3.2 Component Summary

| Component | Responsibility | Technology | Owner |
|-----------|---------------|------------|-------|
| <Component 1> | <What it does> | <Tech stack> | <Team> |
| <Component 2> | <What it does> | <Tech stack> | <Team> |

### 3.3 Component Interaction Matrix

| From \ To | Component A | Component B | Component C |
|-----------|-------------|-------------|-------------|
| Component A | — | <Protocol, frequency> | <Protocol, frequency> |
| Component B | <Protocol> | — | <Protocol> |

---

## 4. Data Flow

### 4.1 Primary Data Flows

```
<Diagram showing how data moves through the system.
Include message rates/frequencies where applicable.>
```

### 4.2 Data Flow Table

| Flow | Source | Destination | Protocol | Frequency | Payload |
|------|--------|-------------|----------|-----------|---------|
| <Flow 1> | <Source> | <Dest> | <HTTP/WS/MQTT> | <rate> | <brief description> |

---

## 5. Security Architecture

### 5.1 Authentication & Authorization

<Describe auth approach: OAuth2/OIDC, JWT, API keys, etc.>

### 5.2 Multi-Tenancy (if applicable)

<How tenant isolation is enforced at each layer.>

### 5.3 Data Protection

<Encryption at rest, in transit. Key management approach.>

---

## 6. Infrastructure & Deployment

### 6.1 Deployment Architecture

```
<Diagram showing deployment topology: regions, availability zones,
container orchestration, CDN, etc.>
```

### 6.2 Environment Summary

| Environment | Purpose | URL |
|-------------|---------|-----|
| Development | Local development | localhost |
| Staging | Pre-production testing | <staging URL> |
| Production | Live system | <production URL> |

---

## 7. Cross-Cutting Concerns

### 7.1 Observability

| Signal | Tool | Retention |
|--------|------|-----------|
| Metrics | <e.g., Prometheus, CloudWatch> | <retention period> |
| Logs | <e.g., ELK, CloudWatch Logs> | <retention period> |
| Traces | <e.g., Jaeger, X-Ray> | <retention period> |

### 7.2 Performance Budgets

| Metric | Target | Measurement |
|--------|--------|-------------|
| API response time (p95) | <target> | <how measured> |
| Page load time | <target> | <how measured> |
| Database query time (p95) | <target> | <how measured> |

### 7.3 Cost Targets (if applicable)

| Resource | Target | At Scale |
|----------|--------|----------|
| <Resource> | <$/month> | <at N users> |

---

## 8. Constraints & Limitations

| Constraint | Impact | Mitigation |
|------------|--------|------------|
| <Technical constraint> | <What it limits> | <How to work within it> |

---

## 9. Critical Invariants

<List non-negotiable system properties that must hold at all times.
These are referenced by LLDs (§9) and enforced by review agents.>

1. **<Invariant 1>:** <description>
2. **<Invariant 2>:** <description>

---

## 10. Appendices

### 10.1 Glossary

| Term | Definition |
|------|-----------|
| <Term> | <Definition> |

### 10.2 References

- <Link to related documents>
