---
name: python
description: Conventions for Python projects, including FastAPI, Django and Flask - project layout and the existing toolchain (venv/uv/poetry, ruff/black, mypy, pytest or unittest), type hints, Decimal for money, explicit error handling, framework idioms (FastAPI dependencies and Pydantic models, Django ORM and migrations, Flask blueprints), and stdlib-first solutions. Use when writing or reviewing Python code or a Python web service.
license: MIT
---

# Python

## Purpose
Python that matches the project's own tools and style, is typed where it helps, and stays correct with money, time and errors.

## When to use
- Any Python code: scripts, libraries, FastAPI/Django/Flask services, CLIs, tests.

## When NOT to use
- API design decisions (routes, status codes, pagination): `api`. Schema and migrations design: `database-engineer`.

## Procedure
1. **Orient first**: Python version (`pyproject.toml`, `.python-version`), dependency manager (uv, poetry, pip + requirements), formatter/linter (ruff, black), type checker (mypy, pyright), test runner (pytest or unittest) and the existing package layout. Use what is there; add no tools without asking.
2. **Style**: follow the formatter; type hints on public functions; dataclasses or Pydantic models instead of loose dicts at boundaries; `pathlib` for paths; f-strings; no mutable default arguments.
3. **Correctness traps**: `Decimal` (with explicit rounding) for money, never float arithmetic for currency; timezone-aware datetimes (`datetime.now(timezone.utc)`); context managers for files, locks and transactions.
4. **Errors**: raise specific exceptions; never `except Exception: pass`; log with context; return proper HTTP errors at the edge only.
5. **Frameworks**:
   - FastAPI: Pydantic request/response models, dependencies (`Depends`) for auth/db sessions, `async` only with async I/O libraries.
   - Django: model methods and managers for business rules, `select_related`/`prefetch_related` against N+1, migrations via `makemigrations`, forms/serializers for validation.
   - Flask: blueprints, app factory, request validation before use.
6. **Prefer the standard library** (argparse, csv, json, pathlib, logging, unittest) before adding a dependency.
7. **Test** with the project's runner and naming, then run it (plus linters and type checks the project uses) and report real results.

## Output
Code and tests in the project's style, and the commands run with their results.

## Failure modes
- pytest-style tests in a unittest project (or vice versa); new tools the project does not use.
- float money, naive datetimes, bare `except`.
- Adding a dependency for something the standard library does.
