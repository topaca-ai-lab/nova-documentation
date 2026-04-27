# Install Skills

## Overview

In Nova, a *skill* is a capability module that gives the agent controlled access to external systems or specialized workflows.

Examples:

- search and web extraction,
- calendar and task workflows,
- mail and messaging integrations,
- local transcription and media processing,
- repository and editor helpers.

For personal setups, the recommended approach is:

1. start with local-first skills,
2. add only the connectors you actually use,
3. keep permissions minimal,
4. test each skill in isolation before combining them.

---

## Skill Architecture in Nova

A skill should be treated as a contract between:

- **agent intent** (what the model wants to do),
- **execution layer** (what is technically allowed),
- **external system** (what actually gets called).

This matters because reliable local agents depend more on strict boundaries than on model size.

Core principles:

- deterministic inputs and outputs,
- explicit error classes,
- capability probing before productive use,
- clear audit trail for side effects.

---

## Prerequisites

Before installing skills, verify:

- Nova CLI is installed and starts correctly,
- local model configuration works (`/login` with your chosen endpoint),
- required runtime dependencies are available,
- secrets are managed outside source files.

Minimal validation flow:

1. run Nova,
2. complete local login,
3. run a basic non-skill task,
4. install one skill,
5. execute one deterministic skill action.

---

## Installation Strategy

Use a phased rollout.

### Phase 1: Foundation Skills

Start with low-risk, high-value skills:

- local filesystem read/organize helpers,
- markdown/document utilities,
- lightweight local automation helpers.

Goal:

- confirm stable invocation format,
- confirm output parsing quality on your default model.

### Phase 2: Communication and Scheduling

Add daily productivity integrations:

- calendar skill (open standards preferred),
- task scheduling bridges,
- notification channels.

Goal:

- validate recurring workflows and side-effect safety.

### Phase 3: External Connectors

Add networked connectors only when needed:

- messaging gateways,
- search providers,
- cloud storage bridges.

Goal:

- maintain minimal attack surface and manageable key rotation.

---

## Recommended Personal Skill Baseline

For most users, this baseline is sufficient:

- `filesystem-helper` (structured read/write in allowed folders),
- `calendar-open` (CalDAV-based flows),
- `mail-open` (IMAP/SMTP draft + send with confirmations),
- `markdown-memory` (local knowledge capture),
- `scheduler-bridge` (cron/heartbeat task execution),
- `local-transcription` (Whisper-local style pipelines),
- `notification-gateway` (Telegram or equivalent).

---

## Configuration Guidelines

### 1) Keep skill configs explicit

Each skill should define:

- allowed actions,
- allowed targets,
- timeout limits,
- retry policy,
- output schema expectation.

### 2) Separate secrets from behavior

Store API keys/tokens in dedicated secret storage or environment variables.
Do not hardcode credentials into markdown config files.

### 3) Define least-privilege scopes

If a connector supports read-only mode, start there.
Upgrade privileges only when your workflow requires write actions.

### 4) Enforce side-effect confirmations

For actions that modify external state, require explicit confirmation gates.

Examples:

- “send email now”,
- “delete calendar event”,
- “overwrite file”.

---

## Verification Checklist After Installation

For each installed skill:

1. capability probe passes,
2. sample command succeeds in deterministic mode,
3. invalid-input path returns typed error,
4. timeout handling works,
5. logs/events are visible and readable.

If one item fails, fix it before installing the next skill.

---

## Security and Privacy Notes

Personal setups often mix private documents, messages, and account data.
Treat skill deployment as a security task.

Minimum controls:

- encrypt local disks,
- isolate credentials by skill,
- rotate tokens periodically,
- monitor unusual outbound calls,
- keep clear allowlists for paths and hosts.

For shared machines:

- use separate user profiles,
- avoid globally accessible token files,
- disable risky write-capable skills when not needed.

---

## Troubleshooting

### Skill is installed but never selected

Possible causes:

- model is not producing valid tool-call format,
- skill capability was not registered,
- routing policy excludes that skill.

What to check:

- tool-calling diagnostics via `nova local doctor`,
- skill registration logs,
- action-policy allow/deny configuration.

### Skill triggers but output is ignored

Possible causes:

- output schema mismatch,
- parser limitations on your model/backend profile.

What to check:

- raw skill output payload,
- expected output schema,
- fallback parser behavior.

### Frequent timeouts

Possible causes:

- connector latency,
- too small timeout budget,
- retry policy too aggressive.

What to check:

- timeout policy settings,
- network round-trip time,
- connector-side rate limits.

---

## Operational Best Practices

- Add one skill at a time.
- Keep a “known-good profile” snapshot.
- Maintain a short rollback procedure per skill.
- Document each new skill with:
  - purpose,
  - permissions,
  - risk level,
  - owner,
  - test command.

This discipline prevents silent degradation and makes Nova dependable on smaller local models.

---

## Next Steps

After skills are installed and validated:

1. set up document ingestion and retrieval workflows ([rag-documents.md](./rag-documents.md)),
2. define repeatable automations ([task-automation.md](./task-automation.md)),
3. measure reliability under your smallest supported model before scaling up.
