---
name: postgres
description: PostgreSQL schema design, migrations, queries, indexing, transactions and safe production changes - zero-downtime migrations, query plans with EXPLAIN, connection pooling, and testing against a real Postgres. Use when a project uses PostgreSQL (driver, JDBC URL, compose image, or SQL migrations) or a task touches its schema, queries or performance.
license: MIT
---

# PostgreSQL

Applies on top of the core skills and the project's framework pack. The project's existing migration tool and conventions win over these defaults.

## Orientation (do first)
- Find the Postgres major version (compose image, Terraform, docs) and the migration tool (Flyway, Liquibase, Prisma, Alembic, Rails, plain SQL). Never change the schema outside it.
- Read the current schema from migrations or `.ai/knowledge/database/` before writing queries; do not guess column names or types.

## Schema
- Every table has a primary key; prefer `bigint generated always as identity` or UUID per project convention.
- Use the narrowest correct type: `timestamptz` (not `timestamp`) for instants, `numeric` for money, `text` over `varchar(n)` unless a limit is a business rule, `jsonb` only for genuinely schemaless data.
- Declare constraints in the database: `not null`, foreign keys, `unique`, `check`. Application validation does not replace them.
- Index foreign keys used in joins and columns used in frequent `where`/`order by`. Each index costs writes; justify it with a query.

## Migrations (production-safe)
- One logical change per migration; never edit a migration that has run anywhere shared.
- Expand -> migrate -> contract for renames and type changes: add the new column, backfill in batches, switch code, drop the old column in a later release.
- `create index concurrently` on large tables (outside a transaction). Adding `not null` or a foreign key to a big table: add as `not valid`, then `validate constraint` separately.
- Assume old and new application versions run against the schema at the same time during deploys.

## Queries
- Parameterized queries only; never concatenate input into SQL.
- Check plans with `EXPLAIN (ANALYZE, BUFFERS)` on realistic data before claiming a query is fast. Look for sequential scans on large tables and row-estimate mismatches.
- Paginate with keyset (`where id > $1 order by id limit n`) for large or infinite lists; `offset` degrades linearly.
- Keep transactions short; never hold one open across network calls. Pick the isolation level deliberately and handle serialization failures with retries.

## Operations
- Use a connection pool sized to the database's `max_connections`, not to the app's thread count.
- Credentials come from the environment or a secret store, never from committed config.

## Testing
- Test against real Postgres (Testcontainers, compose service) rather than H2/SQLite, since types, locking and SQL dialect differ.
- Every migration is exercised by the test suite starting from an empty database.

## Failure modes
- `timestamp without time zone` for instants; floating point for money.
- A migration that locks a hot table (plain `create index`, `alter column type`) during business hours.
- N+1 queries hidden behind an ORM; missing index on a foreign key.
- Claiming a performance fix without a before/after `EXPLAIN ANALYZE`.
