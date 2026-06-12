# Worked Session: Code Review

**Template used:** `templates/code-review.md`

## Setup

```
LANGUAGE: TypeScript
CONTEXT: Express middleware that validates API keys for a B2B SaaS product. Each customer has an API key stored in Postgres. The middleware runs on every authenticated route.
FOCUS_AREAS: security, performance
```

## Prompt Sent

```
You are an experienced TypeScript engineer performing a code review. Be direct, specific, and actionable — no filler commentary.

Context: Express middleware that validates API keys for a B2B SaaS product. Each customer has an API key stored in Postgres. The middleware runs on every authenticated route.

Code to review:
```typescript
import { Request, Response, NextFunction } from 'express';
import { db } from '../db';

export async function validateApiKey(req: Request, res: Response, next: NextFunction) {
  const key = req.headers['x-api-key'];

  if (!key) {
    return res.status(401).json({ error: 'API key required' });
  }

  const customer = await db.query(
    `SELECT * FROM customers WHERE api_key = '${key}'`
  );

  if (customer.rows.length === 0) {
    return res.status(401).json({ error: 'Invalid API key' });
  }

  req.customer = customer.rows[0];
  next();
}
```

Review focus: security, performance
```

## Response Received

### Critical Issues (must fix before merge)

**Line 11 — SQL Injection vulnerability**
The API key is interpolated directly into the SQL string: `WHERE api_key = '${key}'`. An attacker who controls the `x-api-key` header can terminate the string and inject arbitrary SQL. Example payload: `' OR '1'='1` would return all customers and bypass authentication entirely.

**Fix:**
```typescript
const customer = await db.query(
  'SELECT * FROM customers WHERE api_key = $1',
  [key]
);
```

**Line 7 — No type narrowing on `key`**
`req.headers['x-api-key']` has type `string | string[] | undefined`. The undefined check on line 9 works, but if a client sends multiple `x-api-key` headers, `key` will be a `string[]` and your parameterized query will receive an array, likely causing a runtime error or silent failure depending on your DB driver.

**Fix:**
```typescript
const key = Array.isArray(req.headers['x-api-key'])
  ? req.headers['x-api-key'][0]
  : req.headers['x-api-key'];
```

### Improvements (should fix)

**Line 11 — `SELECT *` on authenticated hot path**
`SELECT *` fetches all customer columns on every request. When the customers table grows columns (as it will), this query gets slower and returns fields you don't need in `req.customer`. Select only what downstream handlers use.

**Fix:**
```typescript
const customer = await db.query(
  'SELECT id, name, plan, api_key_expires_at FROM customers WHERE api_key = $1',
  [key]
);
```

**No key expiry check**
If customers can have their keys rotated or expired, add a check: `WHERE api_key = $1 AND (api_key_expires_at IS NULL OR api_key_expires_at > NOW())`.

**No caching — every request hits the database**
For a B2B API that may receive hundreds of requests per second from the same customer, a short-lived in-memory or Redis cache keyed on the API key (TTL: 60-300s) would dramatically reduce DB load.

### Nits

- Rename `customer` to `result` and `customer.rows[0]` to `customer` to reduce confusion between the query result and the customer object.
- Consider adding a timing-safe comparison (`crypto.timingSafeEqual`) when comparing the API key to prevent timing attacks, though this is low-risk for server-side DB lookups.

### What's done well

- Clean early-return pattern for missing keys keeps the happy path readable.
- Attaching `req.customer` makes the customer object available downstream without additional queries — good design.

## What changed after the review

1. Fixed the SQL injection with parameterized query (5 minutes)
2. Added type narrowing for the header (5 minutes)
3. Added `SELECT` column list (2 minutes)
4. Added key expiry check to the query (5 minutes)
5. Filed a ticket for the caching improvement — deferred to next sprint

**Total time saved by catching the SQL injection in review vs. production: incalculable.**
