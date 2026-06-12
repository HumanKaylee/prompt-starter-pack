# Worked Session: Architecture Decision Record

**Template used:** `templates/architecture-decision-record.md`

## Setup

```
DECISION_TITLE: Session Storage: Redis vs. Database-backed vs. JWT (stateless)
PROBLEM: We need to implement user sessions for a new web app. Currently using HTTP Basic Auth per-request which is hitting performance limits and making logout impossible.
CONTEXT: Node.js API, ~50k monthly active users, 3-engineer team, Kubernetes deployment, already running PostgreSQL but no Redis. Need to support: logout, session expiry, concurrent device limits (2 devices max per user).
OPTIONS: Redis sessions, DB-backed sessions (PostgreSQL), JWTs with refresh tokens
CHOSEN_OPTION: help me decide
```

## Response Received (ADR Document)

---
# ADR-007: Session Storage Strategy

**Date:** 2026-06-12
**Status:** Proposed

## Context and Problem Statement

The application currently authenticates every request with HTTP Basic Auth against the database. At 50k MAU with an average of 8 requests per session, this generates ~400k database authentication queries per day. More critically, stateless per-request auth makes logout semantically impossible — a user who logs out still has valid credentials until they change their password.

We need a session mechanism that supports logout, configurable expiry, and a 2-device-per-user limit.

## Decision Drivers

- **Must support real logout** (credentials must be invalidatable without password change)
- **Must enforce 2-device-per-user limit** (requires server-side session list)
- **Infrastructure simplicity** (3-engineer team; minimize new operational components)
- **Latency** (session validation on every request; target < 5ms)
- **No vendor lock-in** (self-hosted, Kubernetes)

## Considered Options

### Option A: Redis-backed Sessions
A dedicated Redis instance stores session data as key-value pairs. The session token is a random ID; all session data lives in Redis.

**Pros:** Sub-millisecond lookups; automatic TTL expiry; purpose-built for this use case; supports concurrent device list natively.
**Cons:** New infrastructure component to operate (deployment, backups, HA); one more thing to go down; added cost.

Decision driver score: Logout ✅ | Device limit ✅ | Simplicity ⚠️ (new infra) | Latency ✅ | Lock-in ✅
**Score: 4/5**

### Option B: Database-backed Sessions (PostgreSQL)
A `sessions` table in the existing PostgreSQL database stores session records. Session lookup is a parameterized query by session token.

**Pros:** No new infrastructure; PostgreSQL already HA with backups; joins with user data possible; supports arbitrary session metadata.
**Cons:** Database becomes a hot path for session validation (every request hits it); connection pool pressure at scale; ~5-20ms per lookup vs. Redis < 1ms.

Decision driver score: Logout ✅ | Device limit ✅ | Simplicity ✅ (no new infra) | Latency ⚠️ (DB on hot path) | Lock-in ✅
**Score: 4/5**

### Option C: JWTs with Refresh Tokens
Short-lived JWTs (15-minute expiry) validated server-side via signature. Refresh tokens stored in the database to enable logout.

**Pros:** No per-request database lookup for validation (stateless for 15 min); horizontally scalable; widely understood.
**Cons:** Logout only works after JWT expiry (15-minute window where a logged-out user can still use their JWT); device limit requires tracking refresh tokens only (gap in enforcement); complexity of token rotation increases attack surface.

Decision driver score: Logout ⚠️ (15-minute lag) | Device limit ⚠️ (partial) | Simplicity ⚠️ (token rotation complexity) | Latency ✅ | Lock-in ✅
**Score: 2.5/5**

## Decision Outcome

**Chosen option:** Option B — Database-backed sessions (PostgreSQL)

**Rationale:** At 50k MAU and a 3-engineer team, the operational overhead of running Redis HA outweighs the latency advantage. PostgreSQL can handle session lookups at this scale with a proper index — a single indexed query on `sessions.token` will run in < 5ms with connection pooling (pgBouncer is already in place). Option C's 15-minute logout lag is a product requirement failure given that real logout is an explicit decision driver.

Revisit this decision if: MAU exceeds 500k, session lookup latency P99 exceeds 20ms, or database CPU from session lookups exceeds 20% of total.

## Consequences

**Positive:** No new infrastructure; sessions are queryable (admin dashboard, audit logs); device limit enforced server-side.

**Negative:** Every API request now hits the database (but with pgBouncer and an indexed sessions table, this is manageable at current scale). Session data cannot survive a database outage.

**Risks:** If session lookup latency degrades under load, the database is the only place to tune. Mitigation: add a `GET /health` probe that measures session lookup latency and alerts if P99 > 10ms.

## Follow-up Actions

1. Create `sessions` table migration: `(id, user_id, token, device_id, created_at, expires_at, last_seen_at)`
2. Add unique index on `sessions.token` and composite index on `(user_id, expires_at)` for device-count queries
3. Implement `POST /sessions` (login), `DELETE /sessions/:id` (logout), `GET /sessions` (list devices)
4. Add session count check (max 2 per user) to `POST /sessions`, evicting the oldest session if limit exceeded
5. Add `session_lookup_latency_ms` metric to APM dashboard with P50/P95/P99 tracking
6. Set calendar reminder to re-evaluate at 150k MAU

---

## What Happened

The ADR clarified that JWT's logout lag was a blocker we'd handwaved. The "help me decide" scoring made Option B the clear winner once we accepted the latency trade-off. Implementation took 2 days. The ADR lives at `docs/decisions/ADR-007-session-storage.md` and has been referenced 4 times in subsequent discussions about session behavior.
