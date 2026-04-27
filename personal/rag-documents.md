# RAG Documents

## Overview

RAG in Nova means: retrieve relevant document context at runtime, then generate grounded answers or actions.

For personal usage, good RAG design is less about maximum scale and more about:

- consistency,
- source traceability,
- low-latency retrieval,
- predictable behavior on small local models.

---

## What “Good Personal RAG” Looks Like

A strong setup has these properties:

- clear document boundaries (projects, topics, private vs shared),
- deterministic ingestion rules,
- stable chunk identifiers,
- metadata that can drive filtering,
- transparent retrieval diagnostics.

If those are in place, model quality improves significantly even on compact models.

---

## Suggested Folder Strategy

Use a strict source layout, for example:

- `knowledge/projects/`
- `knowledge/personal/`
- `knowledge/reference/`
- `knowledge/archive/`

Tag each document with:

- source type,
- topic,
- date window,
- sensitivity level.

This enables precise filtering and reduces irrelevant context.

---

## Ingestion Workflow

Recommended pipeline:

1. normalize document format,
2. split into stable chunks,
3. attach metadata,
4. write to memory store,
5. optionally index vectors,
6. run ingestion validation.

### Format normalization

Convert mixed sources into predictable text-first representations.

Examples:

- markdown stays markdown,
- PDFs converted with section boundaries,
- transcripts cleaned but speaker labels preserved.

### Chunking rules

Prefer semantic chunking with bounded size and slight overlap.

Goals:

- keep each chunk self-contained,
- avoid splitting key statements across chunk borders,
- preserve heading context.

### Deterministic IDs

Chunk IDs should be reproducible from source + chunk position + content hash.
This allows idempotent re-ingestion and clean updates.

---

## Metadata Design

Use metadata that helps retrieval and governance.

Recommended fields:

- `sourceId`,
- `sourceType`,
- `namespace`,
- `tags`,
- `createdAt`,
- `updatedAt`,
- `owner`,
- `sensitivity`.

For project documentation, add:

- `project`,
- `component`,
- `status` (draft/final/deprecated).

---

## Retrieval Profiles

Nova should support multiple retrieval strategies.

### Lexical-first profile

Use when:

- documents are structured and terminology is stable,
- model is small and you need predictable hit quality.

### Vector-first profile

Use when:

- queries are semantic and less keyword-bound,
- embeddings are reliable for your language/domain.

### Hybrid profile

Use when:

- you want the best balance of precision and recall.

For personal environments, hybrid with lexical priority is usually the best default.

---

## Context Budgeting

RAG quality collapses if too much noisy context is injected.

Use explicit budgets:

- max retrieved chunks,
- max total tokens/bytes,
- source diversity rules,
- recency weighting limits.

Practical guidance:

- return fewer, higher-confidence chunks,
- include citation metadata,
- avoid dumping full long documents into context.

---

## Grounding and Citations

Every generated answer using RAG should preserve provenance.

Minimum citation payload:

- source reference,
- chunk identifier,
- short quoted evidence span.

Benefits:

- user trust,
- fast correction of wrong retrieval,
- better debugging of hallucination vs ingestion defects.

---

## Update and Re-Ingestion Policy

Define how updates are handled:

- full re-index,
- incremental update by changed sources,
- scheduled compaction and cleanup.

For personal use:

- incremental updates daily,
- full consistency rebuild weekly or after schema changes.

Archive old chunks instead of hard deleting when possible.

---

## Quality Checks

Run recurring checks:

1. ingestion success rate,
2. chunk count drift by source,
3. retrieval hit quality on fixed benchmark prompts,
4. citation presence rate,
5. latency percentile for retrieval requests.

Track these over time, especially when changing chunking or embeddings.

---

## Privacy and Compliance for Personal Data

Even personal RAG can contain sensitive data.

Minimum controls:

- namespace isolation for private documents,
- sensitive-tag filtering by default,
- optional redaction before indexing,
- encrypted storage at rest,
- strict backup handling.

If you sync across devices, secure transport and endpoint access are mandatory.

---

## Troubleshooting

### Retrieval returns irrelevant chunks

Likely reasons:

- missing metadata filters,
- oversized chunks,
- weak lexical matching.

Actions:

- tighten namespace/tag filters,
- reduce chunk size,
- increase lexical weighting.

### Good chunks exist but are not retrieved

Likely reasons:

- indexing pipeline did not update,
- vector mismatch,
- query normalization issues.

Actions:

- re-run ingestion diff,
- inspect vector index health,
- compare normalized query terms.

### Answers hallucinate despite retrieval hits

Likely reasons:

- too much context noise,
- model ignores citations under overload,
- prompt template lacks grounding constraints.

Actions:

- reduce chunk count,
- enforce citation-required response pattern,
- add strict “answer only from provided context” instruction.

---

## Recommended Starter Workflow

1. ingest one small project corpus,
2. validate with 20 fixed benchmark questions,
3. tune retrieval profile and context budgets,
4. expand to broader personal document sets,
5. automate nightly ingestion checks.

This sequence keeps reliability high while complexity grows.

---

## Next Step

When RAG is stable, automate recurring document workflows and reminders using Nova task automation:

- [task-automation.md](./task-automation.md)
