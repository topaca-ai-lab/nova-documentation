# workflow-skills

## 1. Overview

`@topaca/workflow-skills` is the declarative workflow execution package for Nova/Edgent.

It provides:

- typed workflow definitions and validation,
- deterministic workflow execution semantics,
- built-in dispatchers for tool/decision/memory/transform/finish steps,
- runtime persistence and event publication integration,
- orchestration bridge integration for cron/heartbeat scheduled runs,
- safety policies for runtime, payload, and tool-action constraints,
- observability diagnostics for metrics, health, and failure context.

Current maturity marker exported by the package is **phase-7** (`WORKFLOW_SKILLS_PHASE = "phase-7"`).  
Phase 8 is focused on packaging and release readiness.

---

## 2. Package Location and Entrypoints

Source path:

- `Nova-Packages/workflow-skills/src`

Primary package entry:

- `src/index.ts`

Core modules:

- `types.ts`
- `validator.ts`
- `state-machine.ts`
- `executor.ts`
- `dispatchers.ts`
- `store.ts`
- `events.ts`
- `runtime.ts`
- `orchestration-bridge.ts`
- `safety.ts`
- `observability.ts`

---

## 3. Workflow Definition Model

### 3.1 Top-level contract

`WorkflowDefinition` contains:

- `schemaVersion`,
- `id`, `name`, `version`,
- `entryStepId`,
- `steps`,
- `edges`.

### 3.2 Step kinds

Supported `WorkflowStepKind` values:

- `tool`
- `decision`
- `memory`
- `transform`
- `finish`

### 3.3 Graph constraints

MVP execution mode is deterministic DAG-style execution with explicit edges and validated decision targets.  
Validation ensures that topology and branching are valid before runtime.

---

## 4. Validation Layer

`validateWorkflowDefinition()` and `getWorkflowValidationIssues()` provide structural and semantic checks, including:

- duplicate step IDs,
- invalid entry step,
- invalid edge references,
- cycles,
- unreachable steps,
- missing reachable finish step,
- invalid decision branches/default targets,
- invalid transition topology for finish/decision steps.

Invalid definitions raise `WorkflowValidationError` with typed issue codes.

---

## 5. Runtime State Machine

Run lifecycle:

- `queued -> running -> succeeded|failed|canceled`

Step lifecycle:

- `queued -> running -> succeeded|failed|canceled`
- retry envelope path: `failed -> queued`

Transition guards are centralized in `state-machine.ts` and prevent invalid state mutations.

---

## 6. Executor Semantics

`executeWorkflow()` is the deterministic core executor:

- validates workflow before execution,
- maintains shared execution context,
- executes steps in deterministic order,
- supports retries (`maxRetries`) with bounded exponential backoff,
- supports per-step timeout (`timeoutMs`),
- supports cancellation via `AbortSignal`,
- enforces global step cap (`maxTotalSteps`).

`executeWorkflowWithDispatchers()` provides a convenience path using the built-in dispatcher factory.

---

## 7. Built-in Dispatchers

`createDefaultStepHandler()` includes implementations for:

- `tool`:
  - invokes `toolInvoker` abstraction,
  - maps dependency/tool failures to typed execution errors.
- `decision`:
  - evaluates typed branch conditions,
  - returns a deterministic `nextStepId`.
- `memory`:
  - delegates `read/write/query` to `memoryDispatcher`.
- `transform`:
  - maps structured payload values without dynamic evaluation.
- `finish`:
  - returns terminal payload shaping.

---

## 8. Persistence and Events

### 8.1 Store

`WorkflowStore` abstracts definition and run snapshot persistence:

- `upsert/get/list/delete` for definitions,
- `upsert/get/list/delete` for run snapshots,
- `health()`.

Reference adapter:

- `InMemoryWorkflowStore`.

### 8.2 Event sink

`WorkflowEventSink` handles lifecycle events:

- `run_started`,
- `step_recorded`,
- `run_finished`.

Reference adapter:

- `InMemoryWorkflowEventSink` with bounded retention, subscriptions, snapshots, and health reporting.

---

## 9. Runtime Integration API

`executeWorkflowRuntime()` combines:

- execution,
- optional definition persistence,
- ordered event publishing,
- optional run snapshot persistence.

This API is the recommended entrypoint when operational traceability is required.

---

## 10. Orchestration Bridge

`orchestration-bridge.ts` connects workflow execution to `@topaca/orchestration-core`.

Main helpers:

- `createWorkflowScheduleJobDefinition()`
- `createWorkflowJobId()`
- `parseWorkflowJobId()`
- `createWorkflowRunIntent()`
- `createScheduledWorkflowJobHandler()`
- `registerScheduledWorkflow()`

Supported triggers:

- `cron`
- `heartbeat`

---

## 11. Safety and Hardening

`WorkflowSafetyPolicy` supports:

- tool action policy:
  - global allow/deny lists,
  - per-step allow/deny lists.
- runtime budget:
  - `maxRuntimeMs`.
- payload budgets:
  - `maxInitialInputBytes`,
  - `maxStepInputBytes`,
  - `maxStepOutputBytes`,
  - `maxStoredStepOutputsBytes`,
  - `maxFinalOutputBytes`.

Typed errors:

- `WorkflowPolicyDeniedError`
- `WorkflowMaxRuntimeExceededError`
- `WorkflowPayloadBudgetExceededError`

---

## 12. Observability and Diagnostics

### 12.1 Metrics

- `computeWorkflowMetrics()`
- `computeWorkflowMetricsFromSnapshots()`

Metrics include:

- run counts by status,
- success/failure/cancel rates,
- average run latency,
- average step latency,
- average step count,
- runs with retries,
- per-workflow breakdown.

### 12.2 Health snapshot

`getWorkflowHealthSnapshot()` merges:

- store health,
- event sink health,
- current metrics,
- warning list.

### 12.3 Failure context

`getWorkflowFailureContext()` returns structured failure/cancel context with:

- run identity and status,
- last error,
- failed step details,
- step timeline.

---

## 13. Public API Summary

Primary exports:

- workflow contracts/types,
- validator API,
- state-machine utilities,
- executor API,
- dispatcher API,
- store and event adapters,
- runtime integration API,
- orchestration bridge API,
- safety API,
- observability API,
- typed execution/validation errors.

---

## 14. Testing Strategy

Current suite covers:

- validation rules,
- state-machine transitions,
- executor (success/retry/timeout/cancel),
- dispatcher behavior and integration,
- store/event/runtime integration,
- orchestration bridge mapping/handler flow,
- safety policy enforcement,
- observability metrics/health/failure context.

Recommended release gate:

```bash
npm run check
PATH=/Users/markusertel/.nvm/versions/node/v24.14.1/bin:/opt/homebrew/bin:$PATH npx tsx --test test/*.test.ts
```

---

## 15. Release Metadata

- package: `@topaca/workflow-skills`
- version: `0.0.1` (current in package manifest)
- license: `AGPL-3.0-only`
- repository: `https://github.com/topaca-ai-lab/nova-packages` (directory: `workflow-skills`)

For release steps, see:

- `Nova-Packages/workflow-skills/RELEASING.md`
