---
name: Database Schema Review
use_case: Review or design a database schema for correctness, performance, and maintainability
variables:
  - SCHEMA: the schema (DDL, entity descriptions, or ORM models)
  - DATABASE: database system (PostgreSQL, MySQL, SQLite, MongoDB, etc.)
  - USE_CASE: what queries this schema primarily serves
  - SCALE: current and expected data volume (rows per table, query frequency)
---

# Database Schema Review Prompt

## Prompt

```
Review this database schema for a {{DATABASE}} database. Focus on correctness, query performance, and long-term maintainability.

**Primary use case:** {{USE_CASE}}
**Scale:** {{SCALE}}

**Schema:**
{{SCHEMA}}

Review:

### Data Integrity
Missing NOT NULL constraints, missing UNIQUE constraints, wrong data types, missing foreign key constraints. For each issue: specific table/column and recommended fix.

### Normalization
Is the schema appropriately normalized for its use case? Are there repeating groups, transitive dependencies, or update anomalies? (Don't over-normalize — denormalization is appropriate for read-heavy workloads.)

### Indexes
What indexes are missing that would make the primary use case queries fast? What indexes are probably unnecessary? Include: exact index definition and the query it supports.

### Naming and Conventions
Are names consistent? Are there naming patterns that will cause confusion as the schema grows?

### Migration Hazards
What would be hard to change later? Which decisions are difficult to reverse once data is in production?

### Missing Tables or Fields
Based on the use case, what's likely missing from this schema that will need to be added soon?

### Query Performance Landmines
Any design decisions that will cause slow queries as the table grows? (N+1 query traps, missing pagination support, unbounded text search, etc.)
```

## Worked Example

**Schema:** An e-commerce schema with users, products, orders, and order_items

**What a good response looks like:** Data integrity flags `order_items.price` as DECIMAL(10,2) (correct) but `products.price` as FLOAT (wrong — floating point for currency causes rounding errors); missing index on `orders.user_id` (all user order history queries will full-scan); migration hazard is that `products.description` is a TEXT field with no length limit, making it hard to add search indexes later; missing `order_items.product_snapshot` (what happens to historical orders when a product's price changes?).

## Tuning Notes

- For schemas with existing data: add "this schema is in production with N million rows in the largest table" — the model will adjust migration recommendations.
- Ask for: "What are the top 3 queries this schema will struggle with as it scales?" to think about future problems early.
- For NoSQL schemas: replace the integrity/normalization sections with "document structure trade-offs" and "embedding vs. referencing decisions."
- Run this review before the first migration that adds data — changes are cheap when there's no data to migrate.
