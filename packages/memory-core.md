# memory-core

## 1. Overview

`@topaca/memory-core` is the reusable memory runtime package for Nova/Edgent.

It provides:

- a provider-agnostic memory data model,
- pluggable store adapters (in-memory, SQLite),
- optional vector retrieval integration,
- markdown ingestion with deterministic chunk IDs,
- wiki-style curated memory pages,
- policy and safety controls,
- observability wrappers (events, metrics, health snapshots).

Current package maturity marker exported in code is **phase-7** (`MEMORY_CORE_PHASE = "phase-7"`), with release hardening and publish workflow already implemented.

---

## 2. Package Location and Entrypoints

Source path:

- `Nova-Packages/memory-core/src`

Main public entrypoint:

- `src/index.ts`

Primary export groups:

- domain types (`MemoryEntry`, `MemoryQuery`, `MemoryFilter`, ...),
- interfaces (`MemoryStore`, `EmbeddingProvider`, `VectorIndex`, `MemoryPolicy`),
- stores (`InMemoryMemoryStore`, `SqliteMemoryStore`),
- vector adapter (`InMemoryVectorIndex`),
- markdown ingestion helpers,
- wiki memory helpers,
- policy implementations (`DefaultMemoryPolicy`, `AllowAllMemoryPolicy`),
- wrappers (`PolicyAwareMemoryStore`, `ObservableMemoryStore`),
- typed policy errors (`MemoryPolicyViolationError`).

---

## 3. Core Domain Model

### 3.1 MemoryEntry

Every memory record includes:

- identity (`id`, `namespace`),
- memory classification (`kind`),
- payload (`content.text`, optional `content.structured`),
- tags and provenance metadata,
- lifecycle timestamps (`createdAt`, `updatedAt`, optional `expiresAt`, `deletedAt`),
- optimistic version counter (`version`).

### 3.2 MemoryKind

Supported semantic categories:

- `working`,
- `episodic`,
- `semantic`,
- `fact`.

### 3.3 Query Contracts

`MemoryQuery` supports:

- optional text query,
- namespace/kind/tag/time filters,
- retrieval profile (`lexical`, `vector`, `hybrid`),
- scoring weights (`lexical`, `vector`, `recency`),
- cursor pagination,
- optional diagnostics.

Result shape:

- ranked `hits`,
- optional `nextCursor`,
- optional diagnostics including fallback reason and vector usage.

---

## 4. Architecture Layers

`memory-core` is intentionally layered:

1. **Domain + contracts** (`types.ts`, interface files),
2. **Reference stores** (in-memory + SQLite),
3. **Retrieval logic** (lexical/vector/hybrid ranking),
4. **Ingestion/wikification** (markdown + wiki helpers),
5. **Policy/safety wrappers**,
6. **Observability wrappers**.

This keeps the core package usable in local-only deployments while allowing production hardening through wrappers.

---

## 5. Store Adapters

### 5.1 MemoryStore Interface

All adapters implement:

- `upsert` / `upsertMany`,
- `getById`,
- `query`,
- `remove`,
- `compact`,
- `health`.

### 5.2 InMemoryMemoryStore

Reference runtime adapter for local development and deterministic testing.

Highlights:

- deterministic ranking and pagination,
- soft delete and hard delete support,
- compaction by age/count,
- optional vector integration through injected `EmbeddingProvider` + `VectorIndex`.

### 5.3 SqliteMemoryStore

Persistent adapter based on `node:sqlite`.

Highlights:

- schema migration system with metadata versioning,
- equivalent query/remove/compact semantics to in-memory adapter,
- same vector integration model through injected providers,
- explicit `close()` for lifecycle management.

---

## 6. Retrieval Model

### 6.1 Lexical Mode

Token overlap between query text and entry text plus recency factor.

### 6.2 Vector Mode

Uses query embedding (or forced `queryVector`) and vector index hits.
Falls back gracefully when vector dependencies are missing.

### 6.3 Hybrid Mode

Combines lexical, vector, and recency signals with configurable weights.

Default weighting intent:

- lexical-first reliability,
- vector semantic boost,
- small recency bias to prioritize newer updates.

---

## 7. Vector Integration

### 7.1 EmbeddingProvider

Contract for text -> vector generation:

- `embed({ texts, model?, signal? })`.

### 7.2 VectorIndex

Contract for index storage/retrieval:

- `upsert(vectors)`,
- `remove(entryIds, namespace?)`,
- `search({ namespace?, vector, limit, filter? })`,
- `health()`.

### 7.3 InMemoryVectorIndex

Reference vector adapter using cosine similarity.
Useful for local testing and CI without external vector databases.

---

## 8. Markdown Ingestion

`ingestMarkdownDocument(store, options)` converts markdown sources into deterministic memory chunks.

Pipeline behavior:

1. normalize markdown,
2. paragraph-based chunking with optional overlap,
3. deterministic chunk ID generation using source ID + chunk index + content hash,
4. provenance tagging (`source = markdown`, `sourceRef = <source>#chunk-n`),
5. bulk upsert via store API.

Important property:

- re-ingesting unchanged markdown produces the same chunk IDs, enabling idempotent updates.

---

## 9. Wiki Memory

Wiki helpers provide curated long-lived memory pages:

- `upsertWikiPage`,
- `getWikiPage`,
- `listWikiPages`,
- `wikiPageId`.

Design notes:

- stable IDs based on slug (`wiki:<slug>`),
- markdown-like rendered text plus structured wiki metadata,
- source attribution (`source = wiki`, `sourceRef = slug`),
- tag convention includes `wiki-page`.

---

## 10. Policy and Safety

### 10.1 DefaultMemoryPolicy

Supports:

- allow-list namespaces,
- block-list namespaces,
- optional redaction hook for sensitive content,
- optional max text length control.

### 10.2 PolicyAwareMemoryStore

Wrapper that applies policy checks before delegating to underlying store.

Adds:

- policy decision callbacks,
- optional retention compaction triggers after writes.

### 10.3 Typed Violations

Policy violations throw `MemoryPolicyViolationError` with machine-readable violation codes.

---

## 11. Observability and Operations

### 11.1 ObservableMemoryStore

Wrapper that records operation-level runtime telemetry:

- success/failure events,
- latency per operation,
- query hit/zero-hit counters,
- aggregate metrics snapshots.

### 11.2 Event Sinks

`MemoryEventSink` allows external consumption of runtime events.
`InMemoryMemoryEventSink` provides bounded in-process storage for tests/diagnostics.

### 11.3 Health Snapshot

`getHealthSnapshot()` combines:

- backend health (`store.health()`),
- runtime metrics snapshot,
- recent event counts.

This gives operators a machine-readable operational health view for local and server environments.

---

## 12. Testing Strategy

The package test suite includes:

- store behavior tests (in-memory),
- cross-adapter contract tests (in-memory + SQLite),
- vector/hybrid retrieval tests,
- markdown ingestion determinism tests,
- wiki memory tests,
- policy/safety tests,
- observability tests.

Current outcome in project execution:

- all package tests pass under the current phase-8 implementation.

---

## 13. Release and Packaging

Package metadata includes:

- `AGPL-3.0-only` license,
- `exports` map for typed/default entry,
- public publish configuration,
- repository/homepage/bugs links,
- Node engine requirement (`>=22.5.0`).

Release process is documented in:

- `Nova-Packages/memory-core/RELEASING.md`

The package is published under:

- `@topaca/memory-core`
- current documented package version: `0.1.1`
- license: `AGPL-3.0-only`
