# Threat register — Phase 1

The one-page register required by the direction doc (§7, §12). **No code exists yet, so every mitigation below is planned
and none is verified.** Update a row when its test ships. The product claims in §7 stay off-limits until then.

## Threats

| Threat | Mitigation | Decided in | Status |
|---|---|---|---|
| Malicious web page calls the local API | Loopback binding; exact Host and Origin checks; per-launch token in an `HttpOnly` cookie; GET routes have no side effects | §7, ADR 0001 | Planned; spike 6 |
| Database or API key leaks | OS credential store; masking; never in SQLite, URLs, browser storage, logs or exports | §7 | Planned |
| Sensitive data sent to AI unintentionally | AI off by default; context preview; explicit approval; audit record | §6, §7 | Planned (M8) |
| Workspace copied or stolen | Owner-only permissions; no credentials in the workspace; raw data not copied by default | §7, ADR 0002 | Planned; see R3 and R4 |
| Huge or malformed input exhausts resources | Sampling; size warnings; limits; cancellation | §5, §7 | Planned (M2, M3) |
| AI produces incorrect documentation | Inferred state; human review | §3, §12 | Planned (M1, M8) |

## Residual risks

| # | Risk | Position | Owner of next step |
|---|---|---|---|
| R1 | **The session cookie reaches other local servers.** Cookies aren't isolated by port, so any server on `127.0.0.1` receives it for paths under `/api` and could replay it. | Accepted for Phase 1: a program running as the same user can already read the workspace. Weakest on shared machines. | Confirm in spike 6 |
| R2 | **Windows permissions.** No ACL code in Phase 1. Owner-only access relies on the workspace being under the user's profile, which is the default location. | Accepted for Phase 1 (ADR 0002). | Revisit at hardening (M9) |
| R3 | **Values taken from source data are stored in the workspace:** min/max, frequent values, pattern examples, dictionary examples, contract accepted values, and their backups and SQLite free pages. This conflicts with "raw data not copied by default". | **Open.** Needs a rule before M3 (ADR 0002, ADR 0003). | Domain model and security review |
| R4 | **DuckDB spill files can hold source data** for the length of a job, or until the next start after a crash. They stay inside the workspace (ADR 0002 point 7). | Accepted for Phase 1, once the setting is in place. | Test in M3 |
| R5 | **Workspaces in sync folders or on network drives.** A sync service could copy the live database, and WAL isn't safe on network filesystems. The runtime warns about known sync folders. Network filesystems aren't detected. | Accepted for Phase 1 (ADR 0002). | Document for users |
