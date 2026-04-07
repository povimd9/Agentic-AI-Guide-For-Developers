# CLAUDE.md — TaskFlow API Repository

**First: Read `../CLAUDE.md` (workspace root) for shared standards.**

This file contains API-specific guidance only. Shared rules are NOT duplicated here.

---

## Quick Navigation

| Need to... | Go to |
|------------|-------|
| Run the API | `npm run dev` |
| Run tests | `npm run test` |
| Run migrations | `npx prisma migrate dev` |
| API spec | `../taskflow-specs/api/REST_ENDPOINTS.md` |
| WebSocket spec | `../taskflow-specs/events/WEBSOCKET_EVENTS.md` |

---

## Repository Boundaries

**NEVER modify code outside of this repository.**

- **NEVER** touch files in `../taskflow-web/` — that is the Frontend team's code
- **NEVER** touch files in `../taskflow-mobile/` — that is the Mobile team's code
- **NEVER** touch files in `../taskflow-infra/` — that is the Platform team's code

If you find a bug in another repository:
1. **STOP** — do not attempt to fix it.
2. **DOCUMENT** the issue with full details.
3. **REPORT** to the user.

---

## Architecture Overview

```
Client (React SPA / React Native)
    │
    ▼
API Gateway (Express.js)
    ├── Auth Middleware (JWT validation via Keycloak)
    ├── Tenant Middleware (extracts tenant_id, sets RLS context)
    ├── Rate Limiter (Redis-backed, per-tenant)
    │
    ├── REST Routes (/api/v1/*)
    │   ├── /tasks     — CRUD, assignment, status transitions
    │   ├── /projects  — Workspace/project management
    │   ├── /users     — Profile, preferences
    │   └── /teams     — Team membership, roles
    │
    ├── WebSocket Server (Socket.io)
    │   ├── task:updated    — Real-time task changes
    │   ├── notification:*  — Push notifications
    │   └── presence:*      — User online status
    │
    └── Background Jobs (BullMQ + Redis)
        ├── email:send      — Email notifications
        ├── webhook:deliver  — Webhook delivery with retries
        └── cleanup:expired  — Expired invitation cleanup
```

### Database

- **PostgreSQL 16** with Prisma ORM
- **Row-Level Security** enforced — every query scoped to tenant
- **Redis 7.2** for caching, rate limiting, and job queues

---

## Multi-Tenant Rules

Every database operation MUST be tenant-scoped. The middleware sets the PostgreSQL session variable before every request:

```typescript
// CORRECT — tenant context set by middleware
await prisma.$executeRaw`SELECT set_config('app.current_tenant', ${tenantId}, true)`;

// WRONG — query without tenant context
const tasks = await prisma.task.findMany(); // FORBIDDEN — no tenant filter
```

**Never bypass RLS.** If you need cross-tenant access (admin operations), use a dedicated service account with explicit documentation.

---

## Spec Compliance

Before writing ANY API endpoint or WebSocket event handler:

1. **Read the spec first:** `../taskflow-specs/api/REST_ENDPOINTS.md`
2. **Match field names exactly** — copy from spec, never guess
3. **Add spec reference comment** in code:
   ```typescript
   // Per taskflow-specs/api/REST_ENDPOINTS.md lines 45-62:
   // POST /api/v1/tasks — title (required), description, assignee_id, due_date
   ```

---

## Error Handling

```typescript
// CORRECT — fail fast with specific error
if (!process.env.DATABASE_URL) {
  throw new Error("DATABASE_URL environment variable is required");
}

// CORRECT — propagate with context
try {
  await prisma.task.create({ data: taskData });
} catch (error) {
  logger.error("Failed to create task", { error, taskData });
  throw error; // Propagate to error handler
}

// WRONG — silent fallback
const dbUrl = process.env.DATABASE_URL || "postgres://localhost:5432/default"; // FORBIDDEN

// WRONG — swallowed error
try {
  await riskyOperation();
} catch (error) {
  // silently ignored — FORBIDDEN
}
```

---

## Testing

```bash
# All tests
npm run test

# Unit tests only
npm run test:unit

# Integration tests (requires running PostgreSQL + Redis)
npm run test:integration

# Specific test file
npx jest src/routes/tasks.test.ts

# Database seed for manual testing
npx prisma db seed
```

### Test Database

Integration tests use a separate database (`taskflow_test`). The test harness:
1. Runs migrations
2. Seeds test tenants
3. Runs tests with tenant context
4. Rolls back after each test

---

## Deployment

```bash
# Build
npm run build

# Docker
docker build -t taskflow-api .
docker compose up -d

# Verify
curl http://localhost:3000/health
# Expected: {"status":"ok","version":"0.5.0"}
```

---

## Workflow

- Work on epics 1 story at a time to completion.
- Build and test after every story.
- Max 1-2 stories per session.
- If a decision is needed: STOP and ASK the user.
