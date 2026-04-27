# memory-core-operations

## 1. Purpose

This document describes how to operate `@topaca/memory-core` in real environments.

It focuses on:

- runtime observability,
- health and performance monitoring,
- incident handling,
- operational hardening.

---

## 2. Operational Architecture

Recommended production composition:

1. Base store (`SqliteMemoryStore` or another `MemoryStore` implementation)
2. Policy layer (`PolicyAwareMemoryStore`)
3. Observability layer (`ObservableMemoryStore`)
4. Event sink(s) for telemetry forwarding

Why this order:

- policy decisions are enforced before state changes,
- observability captures both successful and failed operations,
- metrics and snapshots remain backend-agnostic.

---

## 3. Key Signals to Monitor

### 3.1 Availability Signals

- `store.health().ok`
- `getHealthSnapshot().store.ok`
- backend-specific status message (`store.health().message`)

Alert if:

- health is `false` for more than 3 consecutive checks,
- health flips between `true/false` frequently (flapping).

### 3.2 Throughput and Error Signals

From `getMetricsSnapshot()`:

- `totalOperations`
- `errorCount`
- `perOperationCount`

Derived KPIs:

- error rate = `errorCount / totalOperations`
- operation mix = per-operation count distribution

Alert if:

- error rate exceeds baseline threshold (for example >2% sustained),
- `query` operations drop unexpectedly while traffic is expected.

### 3.3 Query Quality Proxy Signals

- `queryCount`
- `queryZeroHitCount`
- `queryHitCountTotal`

Derived KPI:

- zero-hit ratio = `queryZeroHitCount / queryCount`

Alert if:

- zero-hit ratio spikes above historical baseline.

Note:

- this is a proxy metric, not full retrieval quality.

### 3.4 Latency Signals

- `averageLatencyMsByOperation`

Track:

- `upsert` latency trend,
- `query` latency trend,
- `compact` latency trend.

Alert if:

- `query` latency increases sharply without traffic growth,
- `compact` latency grows after dataset expansion (possible retention tuning issue).

---

## 4. SLO Suggestions

Example starting SLOs (adapt to workload):

- Availability: memory health check success >= 99.9%
- Query latency: p95 query latency <= 250ms (local) / <= 600ms (server)
- Error rate: < 1% over rolling 24h
- Zero-hit ratio: <= baseline + tolerated delta

Important:

- use environment-specific SLOs (local dev vs server deployment differ significantly).

---

## 5. Alerting Rules (Practical)

### 5.1 Critical Alerts

- `store.ok == false` for >5 minutes
- error rate >5% for >10 minutes
- no successful writes + expected write traffic

### 5.2 Warning Alerts

- zero-hit ratio above baseline for >15 minutes
- query latency p95 regression >2x baseline
- repeated compaction removals far above normal (possible bad retention thresholds)

---

## 6. Event Stream Operations

`ObservableMemoryStore` emits structured runtime events.

Recommended actions:

- forward events to a centralized log/telemetry pipeline,
- retain recent events in-process via `InMemoryMemoryEventSink`,
- sample high-volume success events and keep full failure events.

Useful event fields:

- `operation`
- `success`
- `durationMs`
- `error`
- `namespace`
- `metadata`

---

## 7. Retention and Capacity Management

Use retention controls in `PolicyAwareMemoryStore`:

- `maxEntriesPerNamespace`
- `maxAgeMs`

Operational guidance:

- start conservatively,
- observe compaction effect and query quality,
- adjust namespace by namespace.

Risk pattern:

- over-aggressive retention can increase zero-hit ratio and degrade answer quality.

---

## 8. Incident Runbook

### 8.1 Elevated Error Rate

1. Read recent failed events (`getEventSnapshot({ success: false })`).
2. Classify failures by operation (`upsert`, `query`, `compact`, etc.).
3. Check store health and backend availability.
4. Verify policy violations vs infrastructure failures.
5. Roll back latest policy/retention config if needed.

### 8.2 Query Quality Drop

1. Check zero-hit ratio trend.
2. Check compaction activity and retention settings.
3. Validate vector integration status (`vectorUsed`, `fallbackReason`).
4. Confirm embedding provider/index health.
5. Re-run controlled retrieval checks with known reference queries.

### 8.3 Latency Regression

1. Inspect `averageLatencyMsByOperation`.
2. Identify dominant slow operation.
3. Correlate with dataset growth and compaction behavior.
4. Check vector backend latency if hybrid/vector mode is active.
5. Reduce compaction frequency or scope during peak load.

---

## 9. Troubleshooting Matrix

### 9.1 `fallbackReason = vector_index_unavailable`

Likely cause:

- vector index not wired or unavailable.

Action:

- verify `vectorIndex` injection and backend health.

### 9.2 `fallbackReason = embedding_provider_unavailable`

Likely cause:

- no embedding provider configured for vector/hybrid queries.

Action:

- provide embedding provider or pass explicit `queryVector`.

### 9.3 Frequent Policy Violations

Likely cause:

- namespace allow/block lists are too restrictive.

Action:

- audit policy config against actual namespace usage.

---

## 10. Security and Compliance Operations

Use policy layer for:

- namespace boundaries,
- optional text redaction before persistence.

Operational recommendations:

- audit redaction behavior in pre-prod,
- log policy decisions (`allow`, `deny`, `redact`) to audit sink,
- avoid storing raw secrets in `content.text`.

---

## 11. Deployment Recommendations

For local single-user setups:

- `InMemoryMemoryStore` + observability wrapper is sufficient.

For shared/server setups:

- `SqliteMemoryStore` (or external adapter),
- policy wrapper with explicit namespace governance,
- observability wrapper with exported events and metrics collection.

---

## 12. Operational Checklist

Before production rollout:

- health checks integrated,
- metrics ingestion integrated,
- alert thresholds defined,
- retention policy validated with realistic data,
- policy restrictions validated for all expected namespaces,
- incident runbook documented for the team.

---

## 13. Release Notes

Current package identity:

- `@topaca/memory-core`
- current documented package version: `0.1.1`
- license: `AGPL-3.0-only`
