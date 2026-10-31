---
name: database-engineer
description: Designs and changes database schemas for any relational or document database - tables and keys, constraints that enforce business rules, indexes for the real queries, migrations that are safe on live data, transactions and data integrity - following the project's migration tool and naming. Use when adding or changing tables, columns, constraints or indexes, writing a migration, modelling how data is stored, or fixing data integrity problems.
license: MIT
---

# Database engineer

## Purpose
Data outlives code. Let the database enforce what must always be true, and change the schema without downtime or data loss.

## When to use
- New tables/collections, columns, relations, unique rules; migrations; integrity bugs (duplicates, orphans); index design.

## When NOT to use
- PostgreSQL-specific tuning, locking and EXPLAIN plans: `postgres` (this skill is engine-neutral; use both on Postgres).
- Business concepts and rules before storage: `domain-modeling`. A slow query whose cause is unknown: `performance` first.

## Procedure
1. **Read the current schema and conventions**: migrations (`.ai/knowledge/database/`, `ai-skills knowledge graph table:<name>`), ORM models, naming, the migration tool (Flyway, Liquibase, Alembic, Prisma, Rails, EF) and the next version number.
2. **Model the change**: keys (surrogate vs natural), relations with foreign keys, nullability, defaults, and the business rules the database must enforce (`UNIQUE`, `CHECK`, `NOT NULL`, FK `ON DELETE` behaviour). Prefer a constraint over application-only checks for rules that must never break.
3. **Index for actual queries**: list the queries that will run, index their filter/join/sort columns, and avoid duplicate or unused indexes. Unique constraints are indexes too.
4. **Write the migration safely for live data** (expand -> migrate -> contract): add nullable/defaulted columns first, backfill in batches, add constraints after data is valid, drop old columns in a later release. Large tables need non-blocking operations (see the engine-specific skill). Every migration is reviewed for a rollback path. Old and new application versions must both work during rollout; backfills and retried steps must be idempotent; the destructive contract step (dropping columns or tables) is a separate, explicitly approved change - never part of the same run.
5. **Keep it transactional**: multi-step writes in one transaction; think about concurrent writers (unique constraints beat check-then-insert races).
6. **Test**: the migration on a copy or test container with realistic data, plus tests for the new constraints (duplicate insert fails, cascade behaves).
7. **Update knowledge**: `ai-skills knowledge discover` refreshes the schema items.

## Output
Migration file(s) named by the project's convention, model/ORM changes, rollback notes, and tests.

## Failure modes
- Enforcing uniqueness only in code (races create duplicates).
- Editing an already-applied migration instead of adding a new one.
- `NOT NULL` without default or backfill on a populated table; dropping columns the running code still reads.
- Inventing table or column names that do not exist.
