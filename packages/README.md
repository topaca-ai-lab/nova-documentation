# Nova Package Documentation

This section contains package-level technical documentation for Nova components that are built as reusable modules.

## Available Packages

- [`connector-skills`](./connector-skills.md) - unified connector contracts for tool execution, capability checks, typed errors, mock adapters, and production adapters across mail/calendar/files/messaging/search/browser/ide/media.
- [`orchestration-core`](./orchestration-core.md) - scheduling, execution lifecycle, persistence adapters, retention, telemetry, event sinks, and dead-letter replay.
- [`memory-core`](./memory-core.md) - unified memory model, store adapters, vector/hybrid retrieval, markdown/wiki ingestion, policy/safety, and observability wrappers.
- [`memory-core-operations`](./memory-core-operations.md) - production operations guide covering monitoring, SLOs, alerting, incident runbooks, and troubleshooting.
- [`workflow-skills`](./workflow-skills.md) - declarative workflow definitions, deterministic executor, dispatcher runtime, safety policy enforcement, orchestration bridge integration, and observability diagnostics.
- [`nova-status`](./nova-status.md) - deterministic runtime status layer with health rules, collector orchestration, scheduler/diagnostics integration mapping, renderers, watch refresh contracts, and snapshot storage.
- [`upstream-porting-phase-d`](./upstream-porting-phase-d.md) - low-risk upstream sync report (`pi-mono v0.69.0 -> v0.70.2`) with ported deltas, skipped items, and validation summary.

## Documentation Scope

Each package document should cover:

- overview and package purpose,
- package location/entrypoints and public API surface,
- core contracts and runtime behavior,
- security/error model and operational guidance,
- testing strategy,
- release notes (package name, version, license).
