---
name: docker
description: Writing and reviewing Dockerfiles and Docker Compose setups - small multi-stage images, layer caching, non-root users, pinned base images, healthchecks, secrets handling, and local development stacks. Use when a project has a Dockerfile or compose file, or a task involves containerizing, building images, or running services locally with Compose.
license: MIT
---

# Docker and Compose

Applies on top of the core skills. Follow the project's existing images, registries and naming.

## Orientation (do first)
- Read the existing `Dockerfile`s, compose files and CI build steps before changing anything; the image may be built differently in CI than locally.
- Identify which services are for local development only and which images ship to production.

## Dockerfile
- Multi-stage builds: build tools in a builder stage, only the runtime artifact in the final stage.
- Pin base images to a specific version tag (and digest for production); never `latest`.
- Order layers for caching: copy dependency manifests and install dependencies before copying source.
- Run as a non-root user (`USER`), with only the files the process needs. Add a `.dockerignore` (exclude `.git`, `node_modules`, build output, `.env`).
- One process per container; use exec-form `CMD`/`ENTRYPOINT` so signals reach the app and it shuts down gracefully.
- Add a `HEALTHCHECK` (or the orchestrator's equivalent) that checks the app, not just the port.

## Secrets
- Never bake secrets into images (`ENV`, `ARG`, copied `.env` files all end up in layers or history). Use build secrets (`--secret`) for build-time credentials and runtime environment or secret stores for runtime.
- Compose: reference an untracked `.env` file; commit a `.env.example` with placeholder values only.

## Compose (local stacks)
- Pin service image versions to match production (same Postgres major version, etc.).
- Use `depends_on` with `condition: service_healthy` rather than sleeps.
- Named volumes for data that should survive restarts; bind mounts only for source code in development.
- Expose only the ports developers need; bind databases to `127.0.0.1`.

## Verification
- Build the image and run it: `docker build` then `docker run` (or `docker compose up --build`) and hit the healthcheck.
- Check size and user: `docker image ls`, `docker run --rm <img> id`.
- Scan the image with the project's scanner (e.g. Trivy, Grype) if one is configured.

## Failure modes
- `latest` tags causing unreproducible builds.
- Copying the whole context before installing dependencies, so every change invalidates the cache.
- Running as root; secrets visible in `docker history`.
- Shell-form `CMD` swallowing SIGTERM, so containers are killed rather than stopped.
