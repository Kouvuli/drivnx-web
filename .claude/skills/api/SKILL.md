---
name: api
description: Designs and changes HTTP/REST, GraphQL or gRPC APIs - resource and route naming, methods and status codes, request validation, error format, pagination, idempotency, versioning and backward compatibility - consistent with the project's existing API and its contract (OpenAPI, schema, proto). Use when adding or changing an endpoint, operation or API contract, or reviewing an API design.
license: MIT
---

# API design

## Purpose
APIs are promises to other code. Make new endpoints consistent with the existing ones, hard to misuse, and safe to evolve.

## When to use
- Adding or changing an endpoint, query, mutation or RPC; designing request/response shapes; changing an OpenAPI/GraphQL/proto contract.

## When NOT to use
- Framework mechanics (controllers, DI, serialization config): the framework pack (`java-spring`, `nextjs`, ...).
- A security audit of an endpoint: `security`. Database query speed: `performance`/`postgres`.

## Procedure
1. **Learn the existing conventions first**: the contract (`.ai/knowledge/api/`, OpenAPI/GraphQL/proto files) and 2-3 existing endpoints. Match their URL style, casing, error shape, auth, pagination and envelope. Consistency beats your preference.
2. **Model the operation**:
   - REST: nouns for resources, methods by semantics (GET safe, PUT/DELETE idempotent, POST create/actions), sub-resources for ownership (`/orders/{id}/cancel` for actions that are not CRUD).
   - Status codes: 200/201 (with `Location`)/204; 400 validation, 401 unauthenticated, 403 forbidden, 404 not found (also for resources the caller may not see), 409 conflict/invalid state, 422 if the project uses it, 429 rate limit. Never 200 with an error body.
3. **Validate input at the boundary** (types, ranges, required fields) and return a consistent error body (`code`, `message`, field errors).
4. **Collections**: paginate (cursor or limit/offset per project convention) with a maximum page size; never return an unbounded list.
5. **Retries and state**: make unsafe operations idempotent where clients retry (idempotency key or natural idempotency); define behaviour for invalid state transitions.
6. **Compatibility**: adding optional fields is safe; removing/renaming fields, changing types or meanings is breaking. For breaking changes, version or deprecate with a migration path.
7. **Update the contract and tests together**: OpenAPI/schema/proto, examples, and tests for success, validation errors, auth and not-found.

## Output
The endpoint definition (method, path, request, responses with status codes), the contract change, compatibility notes, and tests.

## Failure modes
- New endpoint that ignores the existing error format, naming or auth.
- Unbounded list endpoints; returning internal models (password hashes, internal ids) directly.
- Breaking a response field silently; contract and implementation drifting apart.
- Inventing fields or behaviours the requirement did not ask for.
