# Worked Session: Debugging a Race Condition

**Template used:** `templates/debugging-protocol.md`

## Setup

```
SYMPTOM: Occasional duplicate entries in the `audit_events` table. Happens roughly 1 in 200 requests. No errors in logs.
CONTEXT: Node.js, PostgreSQL, high-traffic checkout flow (200 req/s)
WHAT_YOU_TRIED: Added logging around the insert — doesn't reproduce when logging is added (classic Heisenbug)
CODE_SNIPPET: the recordAuditEvent function and its callers
LOGS: No errors, but occasionally two rows with identical (user_id, event_type, occurred_at) appear within the same millisecond
```

## Prompt Sent

```
You are a senior engineer helping me debug a production issue. Think like a detective: form hypotheses from evidence, rank them by probability, and propose the cheapest test to rule each one out.

Symptom: Occasional duplicate entries in the audit_events table. Happens roughly 1 in 200 requests. No errors in logs.

Environment: Node.js, PostgreSQL, high-traffic checkout flow (200 req/s)

What I've already tried: Added logging around the insert — doesn't reproduce when logging is added (classic Heisenbug)

Relevant code:
```javascript
async function recordAuditEvent(userId, eventType, metadata) {
  const existing = await db.query(
    'SELECT id FROM audit_events WHERE user_id = $1 AND event_type = $2 AND occurred_at > NOW() - INTERVAL \'1 second\'',
    [userId, eventType]
  );

  if (existing.rows.length > 0) {
    return; // deduplicate
  }

  await db.query(
    'INSERT INTO audit_events (user_id, event_type, metadata, occurred_at) VALUES ($1, $2, $3, NOW())',
    [userId, eventType, JSON.stringify(metadata)]
  );
}
```

Logs/errors: No errors, but occasionally two rows with identical (user_id, event_type, occurred_at) appear within the same millisecond
```

## Response Received

### Root Cause Hypotheses

**1. TOCTOU race condition (most likely, ~85% confidence)**
The SELECT-then-INSERT pattern has a classic time-of-check-time-of-use race. Two concurrent requests for the same user can both execute the SELECT simultaneously, both find no existing row, both proceed to INSERT. Since Node.js is single-threaded but `await` yields to the event loop, two inflight requests can interleave at every `await` point. The "disappears with logging" behavior is the canonical signature — the added latency from logging creates enough time for the first INSERT to commit before the second SELECT runs.

Evidence to confirm: two INSERTs with timestamps within the same millisecond for the same (user_id, event_type) pair.

**2. Multiple application instances without coordination (medium, ~12% confidence)**
If this runs behind a load balancer with multiple Node processes, each process has its own event loop and no shared memory. Requests can be routed to different instances simultaneously. The SELECT-INSERT race applies across processes with no mitigation.

Evidence to confirm: check if duplicates only occur under high load or after recent scaling events.

**3. Retry middleware double-submitting (low, ~3% confidence)**
Some HTTP middleware automatically retries on timeout. If the INSERT succeeds but the response is slow, the client (or proxy) retries and the deduplication window hasn't expired yet.

### Cheapest Diagnostic Step

Run this query against your production DB:

```sql
SELECT user_id, event_type, date_trunc('millisecond', occurred_at) as ts, count(*)
FROM audit_events
GROUP BY 1, 2, 3
HAVING count(*) > 1
ORDER BY ts DESC
LIMIT 20;
```

If duplicates cluster within the same millisecond for the same (user_id, event_type), hypothesis 1 is confirmed. If they're spread across milliseconds, look harder at hypothesis 2 or 3.

### Fix (for hypothesis 1 — TOCTOU)

Use a PostgreSQL `INSERT ... ON CONFLICT DO NOTHING` with a unique constraint instead of the application-level check:

```sql
-- Migration: add unique constraint
ALTER TABLE audit_events
  ADD CONSTRAINT uq_audit_events_dedup
  UNIQUE (user_id, event_type, date_trunc('second', occurred_at));
```

```javascript
async function recordAuditEvent(userId, eventType, metadata) {
  await db.query(
    `INSERT INTO audit_events (user_id, event_type, metadata, occurred_at)
     VALUES ($1, $2, $3, NOW())
     ON CONFLICT (user_id, event_type, date_trunc('second', occurred_at))
     DO NOTHING`,
    [userId, eventType, JSON.stringify(metadata)]
  );
}
```

This atomically deduplicates at the database level. No race condition possible.

### Blind Spots

- **Expression index syntax:** `date_trunc('second', occurred_at)` in a unique constraint requires PostgreSQL 14+. Verify your version.
- **Backfill:** existing duplicates in the table will need to be cleaned up before you can add the unique constraint.
- **`DO NOTHING` vs `DO UPDATE`:** with `DO NOTHING`, the returned row count will be 0 on conflict — make sure callers don't treat this as an error.

## What Changed

1. Ran the diagnostic query — confirmed millisecond-level duplicates (hypothesis 1 confirmed)
2. Cleaned up existing duplicates: `DELETE FROM audit_events WHERE id NOT IN (SELECT MIN(id) FROM audit_events GROUP BY user_id, event_type, date_trunc('second', occurred_at))`
3. Added unique constraint
4. Replaced SELECT-then-INSERT with `INSERT ... ON CONFLICT DO NOTHING`
5. Deployed — zero duplicates in 48 hours of production traffic at 200 req/s

**Diagnosis-to-fix time: 45 minutes. Previous debugging time: 3 days.**
