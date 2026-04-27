# nova-status

## 1. Overview

`@topaca/nova-status` is the deterministic status package for Nova/Edgent.

It provides runtime visibility for:

- agent activity state,
- heartbeat/cron scheduler health,
- internal and extended diagnostics,
- dependency health across core Nova packages.

The package is designed to stay reliable on very small local models by keeping status evaluation fully LLM-independent.

---

## 2. Package Location and Entrypoints

Source path:

- `Nova-Packages/nova-status/src`

Public entrypoint:

- `src/index.ts`

Core modules:

- `types.ts`
- `rules.ts`
- `collectors.ts`
- `renderers.ts`
- `watch.ts`
- `integrations.ts`
- `snapshot-store.ts`

---

## 3. Core Domain Contracts

Main status shape:

- `NovaStatusSnapshot`
- `NovaStatusSeverity` (`green|yellow|red|unknown`)
- `NovaStatusIssue` + typed `NovaStatusIssueCode`

Domain statuses:

- `AgentStatus`
- `SchedulerStatus`
- `DiagnosticsStatus`
- `DependencyStatus`

---

## 4. Deterministic Rule Engine

`rules.ts` includes:

- `evaluateNovaStatus(...)`
- `determineOverallSeverity(...)`
- `countSeverities(...)`
- `buildNovaStatusSnapshot(...)`

Policy highlights:

- red dominates,
- yellow degrades overall status,
- unknown signals are treated as degraded (mapped to yellow at overall level),
- issues are sorted deterministically.

---

## 5. Collector Orchestration

`collectNovaStatus(...)` runs domain collectors with timeout-safe behavior.

Fallback behavior:

- missing collectors -> `unknown_state`,
- timed out collectors -> `collector_timeout`,
- failed collectors -> `collector_failed`.

The package includes deterministic helper collectors:

- `createStaticCollector(...)`
- `createFailingCollector(...)`

---

## 6. Rendering APIs

`renderers.ts` provides:

- `renderNovaStatusText(...)`
  - `compact` mode
  - `verbose` mode
- `renderNovaStatusJson(...)`

These outputs are deterministic and suitable for CLI/TUI and automation pipelines.

---

## 7. Watch Contract

`watch.ts` defines refresh behavior without runtime side effects:

- `createNovaStatusWatchContract(...)`
- `computeRefreshDelay(...)`
- `computeNextRefreshAt(...)`

Supports bounded exponential backoff for repeated refresh failures.

---

## 8. Integration Hooks

`integrations.ts` maps external runtime signals into normalized status:

- `mapSchedulerSignalsToStatus(...)`
- `mapDiagnosticProbesToStatus(...)`
- `mapDependencySignalsToStatus(...)`

It also derives domain-specific issues:

- `deriveIssuesFromAgentStatus(...)`
- `deriveIssuesFromSchedulerStatus(...)`
- `deriveIssuesFromDiagnosticsStatus(...)`
- `deriveIssuesFromDependencyStatus(...)`

---

## 9. Snapshot Store

`snapshot-store.ts` provides in-memory persistence:

- `InMemoryNovaStatusSnapshotStore`
- `createInMemoryNovaStatusSnapshotStore(...)`

Supported operations:

- `upsert`
- `getByGeneratedAt`
- `list` (optional severity filter / limit)
- `prune`
- `health`

---

## 10. Testing and Hardening

Current test coverage includes:

- rules,
- collectors,
- renderers,
- watch contract,
- integration mappings,
- snapshot store,
- deterministic degraded-state behavior.

Recommended package validation:

```bash
npm run check
PATH=/Users/markusertel/.nvm/versions/node/v24.14.1/bin:/opt/homebrew/bin:$PATH npx tsx --test test/*.test.ts
```

---

## 11. Release Metadata

- package: `@topaca/nova-status`
- maturity marker: `phase-5` (MVP baseline completed)
- license: `AGPL-3.0-only`
- repository: `https://github.com/topaca-ai-lab/nova-packages` (`nova-status/`)
