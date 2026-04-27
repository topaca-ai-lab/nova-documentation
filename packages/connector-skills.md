# connector-skills

## 1. Overview

`@topaca/connector-skills` is the integration package that connects Nova/Edgent to external systems.

It provides:

- a unified skill request/response envelope,
- strict, model-friendly contracts for tool inputs and outputs,
- one shared error taxonomy across all connector families,
- interface-first APIs for mocking, testing, and production adapters,
- a complete mock layer plus real adapters for all current connector families.

Current runtime maturity marker in code is `CONNECTOR_SKILLS_PHASE = "phase-4"`.

---

## 2. Package Location and Entrypoints

Source path:

- `Nova-Packages/connector-skills/src`

Main public entrypoint:

- `src/index.ts`

Package name and license:

- `@topaca/connector-skills`
- `AGPL-3.0-only`

---

## 3. Public API Surface

The package exports:

- envelope primitives: `SkillRequest`, `SkillResponse`, `SkillError`, `ConnectorCapabilityCheck`,
- helpers: `okResponse`, `errorResponse`,
- typed error classes: `ConnectorSkillError`, `ConnectorAuthError`, `ConnectorTimeoutError`, `ConnectorNotAvailableError`, `ConnectorValidationError`,
- domain types per family (mail, calendar, files, messaging, search, browser, ide, media),
- interface contracts (`MailConnector`, `CalendarConnector`, ...),
- mock adapters for deterministic local tests,
- real adapters for production integrations.

---

## 4. Unified Envelope Contract

All connector calls are normalized through one envelope.

`SkillRequest` fields:

- `skillId`: capability namespace (`mail`, `search`, `ide`, ...),
- `action`: operation identifier (`inbox.list`, `pageClick`, ...),
- `params`: flat argument object,
- `traceId`: optional correlation id,
- `signal`: optional `AbortSignal`.

`SkillResponse` fields:

- `skillId` and `action` echoed from request,
- `ok` boolean success flag,
- `result` payload when successful,
- `error` payload when failed,
- `durationMs` execution latency.

Design intent:

- predictable parsing for small local models,
- deterministic handling in repair loops,
- simple telemetry attachment.

---

## 5. Error Taxonomy

All adapters map runtime/provider failures into five normalized classes:

1. `ConnectorAuthError` for credentials/auth failures.
2. `ConnectorTimeoutError` for action timeouts.
3. `ConnectorNotAvailableError` for unavailable/offline backends.
4. `ConnectorValidationError` for invalid or unsafe parameters.
5. `ConnectorSkillError` as base/internal generic error.

This keeps retry/repair logic stable across heterogeneous backends.

---

## 6. Connector Families and Real Adapters

Supported families and default real adapters:

1. Mail: `ImapSmtpMailConnector` (`imapflow` + `nodemailer`)
2. Calendar: `CalDavCalendarConnector` (`tsdav`)
3. Files: `WebDavFilesConnector` (`webdav`)
4. Messaging: `TelegramMessagingConnector` (`grammY`)
5. Search: `FetchSearchConnector` (native fetch, providers: Brave/SearXNG/DuckDuckGo)
6. Browser: `PlaywrightBrowserConnector` (DI-based `Page` integration)
7. IDE: `LocalIdeConnector` (filesystem-backed local editing)
8. Media: `LocalMediaConnector` (local Whisper CLI + transcript fetch)

Each family also has an in-memory/mock connector for test and simulation workflows.

---

## 7. Capability Negotiation

Every connector implements:

- `check(): Promise<ConnectorCapabilityCheck>`

The orchestration layer should call `check()` before tool execution to validate:

1. backend reachability,
2. credential validity,
3. action-level capability availability.

This is a hard guardrail against tool hallucination and avoidable retries.

---

## 8. Security and Runtime Guarantees

Current hardening points implemented in code:

- `LocalIdeConnector` enforces workspace-root boundaries and rejects path traversal/escape attempts.
- `LocalIdeConnector` uses top-level imports only; no runtime inline import path.
- `PlaywrightBrowserConnector` propagates mapped runtime errors instead of returning silent fallback values.
- `TelegramMessagingConnector` reports `receiveMessages` capability based on polling mode and rejects receive calls when polling is disabled.

These constraints are required for safe local execution with mixed trust boundaries.

---

## 9. Testing Strategy

The package includes:

- unit tests for all mock adapters,
- regression tests for IDE path safety, Playwright error mapping, and Telegram polling behavior,
- deterministic local execution without external APIs for the core suite.

Useful commands from package root:

- `npm run check`
- `npm run test`

---

## 10. Operational Notes

Recommended usage pattern in Nova:

1. run capability probes during startup/profile selection,
2. route tool calls through envelope + typed errors,
3. collect per-action telemetry (`durationMs`, trace id),
4. keep connector payloads compact for small-model reliability.

---

## 11. Release Notes

Current published baseline:

- `@topaca/connector-skills@0.0.2`

Recent release includes:

- IDE path traversal protection,
- improved Playwright failure propagation,
- Telegram capability alignment,
- new regression tests and standardized test script.
