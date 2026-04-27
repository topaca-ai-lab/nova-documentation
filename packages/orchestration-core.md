# orchestration-core

## 1. Overview

`@topaca/orchestration-core` is the runtime package responsible for deterministic orchestration in Nova.

It provides:

- job scheduling (`cron` and `heartbeat`),
- execution lifecycle control (`queued -> running -> succeeded|failed|canceled`),
- retry and timeout behavior,
- pluggable persistence (in-memory and SQLite),
- retention/compaction policies,
- observability primitives (events, metrics, health),
- event sink policies (retry/backoff + dead-letter + replay).

Current package maturity in the codebase is **phase-8** (`ORCHESTRATION_CORE_PHASE = "phase-8"`).

---

## 2. Package Location and Entrypoints

Source path:

- `Nova-Packages/orchestration-core/src`

Main public entrypoint:

- `src/index.ts`

Primary export groups:

- domain types (`JobDefinition`, `RunRecord`, `OrchestrationEvent`, ...),
- scheduler/state-machine/runner utilities,
- stores (`InMemoryOrchestrationStore`, `SqliteOrchestrationStore`),
- orchestrator runtime (`Orchestrator`),
- retention worker (`RetentionCompactionWorker`),
- event sinks (`InMemoryOrchestrationEventSink`, `InMemoryOrchestrationDeadLetterSink`),
- error taxonomy (`JobNotFoundError`, `JobAlreadyRunningError`, ...).

---

## 3. Core Domain Contracts

### 3.1 JobDefinition

A job includes:

- `id`: stable identifier,
- `name`: operator-facing label,
- `trigger`: schedule definition,
- `retry`: retry policy.

### 3.2 Trigger

Two trigger modes:

- `cron`: standard 5-field expression (`minute hour day month weekday`),
- `heartbeat`: fixed interval in milliseconds.

### 3.3 RunRecord

Execution record fields:

- run identity (`runId`, `jobId`),
- state (`status`),
- retry attempt counter,
- queue/start/finish timestamps,
- optional error text.

### 3.4 OrchestrationEvent

Lifecycle events emitted by runner/orchestrator:

- `job_registered`,
- `run_queued`,
- `run_started`,
- `run_succeeded`,
- `run_failed`,
- `run_canceled`.

---

## 4. Runtime Components

### 4.1 State Machine (`state-machine.ts`)

Provides strict run transition validation and helper APIs.

Allowed transitions are enforced centrally to prevent invalid status writes.

### 4.2 Scheduler (`scheduler.ts`)

Provides:

- `validateTrigger(trigger)`,
- `getNextRunAt(trigger, fromDate)`.

Behavior notes:

- cron resolution is UTC-based and deterministic,
- cron supports ranges/lists/steps,
- day-of-month/day-of-week matching follows cron semantics,
- heartbeat computes next run via fixed interval.

### 4.3 Runner (`runner.ts`)

`runJob(job, handler, options)` handles:

- queue/start/success/failure/cancel events,
- retry loop with bounded exponential backoff,
- timeout enforcement,
- cancellation propagation and cancellation classification.

Key errors:

- `JobRunTimeoutError`,
- `JobRunCanceledError`.

### 4.4 Orchestrator (`orchestrator.ts`)

High-level orchestration API for registration, scheduling, execution, and telemetry.

Key methods:

- `registerJob(job, handler)`,
- `start()` / `stop()`,
- `runNow(jobId, options)`,
- `cancel(jobId)`,
- `getJob`, `listJobs`, `listRuns`,
- `onEvent(listener)`,
- `getMetrics()` / `resetMetrics()`,
- `getHealth()`,
- `replayDeadLetters(options)`.

---

## 5. Persistence Adapters

### 5.1 OrchestrationStore Interface

Required store operations include:

- job CRUD-like operations,
- run upsert/list/get/delete operations,
- retention compaction (`pruneRuns`),
- stats snapshot (`getStats`).

### 5.2 InMemoryOrchestrationStore

Reference implementation for tests and local runtime.

Characteristics:

- zero external dependencies,
- fast and deterministic for unit tests,
- data lifetime tied to process lifetime.

### 5.3 SqliteOrchestrationStore

Persistent adapter using `node:sqlite`.

Features:

- migration-based schema bootstrap,
- foreign key run cleanup on job delete,
- compaction history table,
- operational stats (`schemaVersion`, sqlite version, db path).

Migration model:

- schema metadata table tracks version,
- migrations run transactionally in ascending order.

---

## 6. Retention and Compaction

### 6.1 RunRetentionPolicy

Supported retention controls:

- `maxRunsPerJob` (count-based cap),
- `maxAgeMs` (time-based expiration).

Retention can be applied in two ways:

- inline after run persistence (orchestrator `runRetention` option),
- background/periodic compaction worker.

### 6.2 RetentionCompactionWorker

`RetentionCompactionWorker` scans jobs and applies `pruneRuns`.

Supports:

- `runOnce()` for explicit maintenance,
- `start()` / `stop()` for periodic compaction,
- cycle callback with summary.

---

## 7. Event Delivery Architecture

### 7.1 Event Sink Interface

`OrchestrationEventSink` is a simple push contract:

- `publish(event)` (sync or async).

The orchestrator can use:

- one sink (`eventSink`), or
- multiple sinks (`eventSinks`).

### 7.2 InMemoryOrchestrationEventSink

Useful for testing and diagnostics:

- stores recent events with bounded memory (`maxEvents`),
- supports subscribers (`subscribe`),
- supports pull snapshots with filters:
  - `jobId`,
  - event `types`,
  - `limit`.

---

## 8. Event Sink Policy (Retry/Backoff/DLQ)

Configured via `eventSinkPolicy` in `OrchestratorOptions`.

Controls:

- `maxAttempts`,
- `baseDelayMs`,
- `maxDelayMs`,
- optional `deadLetterSink`.

Behavior:

1. publish to sink,
2. on failure retry with bounded exponential backoff,
3. on final failure optionally write dead-letter entry.

### 8.1 Dead-Letter Model

`EventSinkDeadLetterEntry` contains:

- stable `deadLetterId`,
- original `event`,
- `sinkIndex`,
- total `attempts`,
- `failedAt`,
- `errorMessage`.

### 8.2 InMemoryOrchestrationDeadLetterSink

Replay-capable dead-letter sink with:

- `snapshot(limit?)`,
- `ack(deadLetterIds)`,
- `size()`,
- bounded memory (`maxEntries`).

---

## 9. Dead-Letter Replay API

`orchestrator.replayDeadLetters(options)` supports controlled redelivery.

Replay guard options:

- `limit`: global cap,
- `maxReplayPerRun`: guardrail per replay invocation,
- `jobId`: replay only entries for one job,
- `sinkIndex`: replay only entries bound to one sink slot.

Replay flow:

1. load DLQ snapshot,
2. apply filters and replay limits,
3. republish to original sink index,
4. acknowledge successful entries,
5. return replay summary.

`DeadLetterReplaySummary` returns:

- `scanned`,
- `attempted`,
- `succeeded`,
- `failed`,
- `acked`,
- `remaining`.

Important behavior:

- replay attempts do **not** recursively dead-letter on replay failure,
- replay requires a replay-capable dead-letter sink implementation.

---

## 10. Concurrency Model

### 10.1 Per-job concurrency

`runNow` rejects if the same job is already running:

- `JobAlreadyRunningError`.

### 10.2 Global concurrency

`maxConcurrentRuns` defines process-wide active run limit.

If exceeded:

- `GlobalConcurrencyLimitExceededError` is thrown.

Scheduled runs encountering limits are skipped for that tick and retried by next schedule cycle.

---

## 11. Error Taxonomy

Core orchestration errors:

- `JobNotFoundError`,
- `JobHandlerNotRegisteredError`,
- `JobAlreadyRunningError`,
- `GlobalConcurrencyLimitExceededError`,
- `InvalidRunTransitionError`,
- `InvalidTriggerError`,
- `SchedulerRangeError`,
- `JobRunTimeoutError`,
- `JobRunCanceledError`.

---

## 12. Observability

### 12.1 Metrics (`OrchestratorMetrics`)

Covers:

- run lifecycle counts,
- scheduling preflight failures,
- retention compaction counters,
- sink delivery success/failure/retry counters,
- dead-letter replay counters.

### 12.2 Health (`getHealth()`)

Returns runtime + store snapshot:

- orchestrator status (`started`, running/scheduled counts),
- metrics snapshot,
- store stats.

### 12.3 Store Stats (`getStats()`)

Adapter-neutral fields:

- backend type,
- job/run totals,
- run distribution by status,
- last compaction timestamp per job,
- adapter metadata map.

SQLite metadata currently includes:

- schema version,
- database path,
- sqlite version,
- connection open state.

---

## 13. Production Usage Guidance

Recommended defaults:

- start with SQLite store for persistence,
- define explicit retention (`maxRunsPerJob`, `maxAgeMs`),
- set `maxConcurrentRuns` defensively,
- use event sinks with dead-letter enabled,
- run replay in controlled batches (`maxReplayPerRun`).

Operational suggestions:

- monitor sink failure/retry trends,
- alert on DLQ growth and replay failure counters,
- keep replay filterable by `jobId` for incident isolation,
- persist/recover DLQ externally if process restarts are frequent.

---

## 14. Example: Full Runtime Setup

```ts
import {
  InMemoryOrchestrationDeadLetterSink,
  InMemoryOrchestrationEventSink,
  Orchestrator,
  createSqliteOrchestrationStore,
} from "@topaca/orchestration-core";

const store = createSqliteOrchestrationStore({ path: "./orchestration.sqlite" });
const eventSink = new InMemoryOrchestrationEventSink({ maxEvents: 5000 });
const deadLetterSink = new InMemoryOrchestrationDeadLetterSink({ maxEntries: 10000 });

const orchestrator = new Orchestrator({
  store,
  maxConcurrentRuns: 4,
  runRetention: {
    maxRunsPerJob: 200,
    maxAgeMs: 1000 * 60 * 60 * 24 * 7,
  },
  eventSinks: [eventSink],
  eventSinkPolicy: {
    maxAttempts: 3,
    baseDelayMs: 25,
    maxDelayMs: 250,
    deadLetterSink,
  },
});

await orchestrator.registerJob(
  {
    id: "heartbeat:maintenance",
    name: "maintenance",
    trigger: { kind: "heartbeat", intervalMs: 60_000 },
    retry: { maxRetries: 2, baseDelayMs: 250, maxDelayMs: 2_000 },
  },
  async () => {
    // job logic
  },
);

await orchestrator.start();
```

---

## 15. Testing and Validation

Current package test coverage includes:

- state transitions,
- scheduler correctness,
- retry/timeout/cancel execution behavior,
- in-memory + SQLite store contracts,
- retention worker behavior,
- orchestrator lifecycle and limits,
- event sink delivery, retry, DLQ, and replay guards.

When changing orchestration behavior, always re-run the package suite and verify:

- lifecycle event ordering remains stable,
- metrics semantics remain backward-stable,
- replay behavior is deterministic under filters and limits.

---

## 16. Known Constraints

- SQLite support uses `node:sqlite` (experimental in Node 24 line),
- in-memory sinks/stores are process-local (not distributed),
- replay currently requires replay-capable dead-letter sink implementation,
- event sink publishing is asynchronous and may interleave across events/sinks.

---

## 17. Recommended Next Iteration

If this package is moved to production-critical usage, next high-value enhancements are:

- persistent dead-letter sink adapter (SQLite or queue-backed),
- replay idempotency key support,
- sink-level circuit breaker and cooldown windows,
- multi-node coordination/locking abstraction for distributed runners.

---

## 18. Release Notes

Current package identity:

- `@topaca/orchestration-core`
- current documented package version: `0.0.1`
- license: `AGPL-3.0-only`
