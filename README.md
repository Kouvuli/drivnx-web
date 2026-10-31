# conflux-web

The Conflux Phase 1 product: a local Python runtime with a browser UI that connects unfamiliar data and turns it into
evidence-backed, human-reviewed understanding (Data Profile → Data Dictionary → Data Contract).

- **Source of truth:** [../docs/Conflux_Phase1_Development_Direction.md](../docs/Conflux_Phase1_Development_Direction.md)
  (approved); open findings in [../docs/Conflux_Phase1_Review_Notes.md](../docs/Conflux_Phase1_Review_Notes.md)
- **Status:** skeleton only, no code yet. Start with the first vertical slice in §12 of the direction doc.
- **Stack (§4):** Python 3.12, FastAPI, Pydantic v2, DuckDB, SQLite · React + TypeScript + Vite, TanStack, Radix/shadcn ·
  uv and pnpm with committed lockfiles · pytest, Vitest, Playwright, Docker Compose PostgreSQL fixture

## Layout (§10)

```text
apps/local-api      FastAPI runtime: sessions, jobs, files, credentials, DuckDB, SQLite, AI providers; serves the UI
apps/web            React + TypeScript + Vite browser UI
packages/           domain, connectors, profiling, artifacts, ai, contracts
tests/              golden, integration, e2e
testdata/           synthetic datasets only
docs/               adr, threat-model, product-research
.ai/                AI Skill OS: manifest, lock, knowledge
```

## AI skills

Set up with `../ai-skills/conflux-skills.sh` (the shared Conflux baseline plus this repo's stack skills); the list is in
`.ai/manifest.yaml`.
