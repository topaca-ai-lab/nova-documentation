# Upstream Porting Report: Phase D

Date: 2026-04-26  
Target: Nova (`/nova`)  
Reference upstream range: `pi-mono v0.69.0 -> v0.70.2`

## Goal

Phase D focused on low-risk, high-value ports after the core reliability work from earlier phases.  
Selection criteria:

- no behavioral regressions for existing Nova local workflows,
- small, isolated diffs,
- clear correctness or UX improvement,
- easy validation via existing `npm run check`.

## Ported Changes

### 1) Windows package manager shell hardening

Upstream reference: `7bd9f650`  
Status: Ported

What was done:

- Added command-aware shell decision (`shouldUseWindowsShell`) in package manager process spawning.
- Replaced broad `shell: process.platform === "win32"` with selective shell usage for npm/pnpm/yarn/corepack and `.cmd`/`.bat`.

Files:

- `nova/packages/coding-agent/src/core/package-manager.ts`

Impact:

- Reduces Windows command execution edge cases during extension/package install/update.
- Keeps non-shell binaries safer and more predictable.

### 2) Google Vertex custom base URL forwarding

Upstream reference: `65a6472b`  
Status: Ported

What was done:

- Added `buildHttpOptions()` for Vertex client construction.
- Forwarded custom `model.baseUrl` to `@google/genai` `httpOptions.baseUrl`.
- Added `baseUrlResourceScope = COLLECTION`.
- If base URL already includes an API version path (`/v1`, `/v1beta`, etc.), clears appended API version to avoid malformed URLs.

Files:

- `nova/packages/ai/src/providers/google-vertex.ts`

Impact:

- Improves compatibility with Vertex-compatible proxies and custom gateway endpoints.

### 3) Scoped model selector height consistency

Upstream reference: `27c05b7f`  
Status: Ported

What was done:

- Reduced `/scoped-models` list height from `15` to `8` visible rows.

Files:

- `nova/packages/coding-agent/src/modes/interactive/components/scoped-models-selector.ts`

Impact:

- Aligns interaction density with `/model` selector.
- Better fit on smaller terminals.

### 4) Login/selector UI consistency polish

Upstream reference: `d4d3c2fd` (applied where still relevant)  
Status: Ported

What was done:

- Standardized accent/bold title rendering in selector-style dialogs.
- Improved row padding/alignment and muted text rendering for non-selected providers.

Files:

- `nova/packages/coding-agent/src/modes/interactive/components/oauth-selector.ts`
- `nova/packages/coding-agent/src/modes/interactive/components/extension-selector.ts`

Impact:

- More consistent visual hierarchy in `/login` and related selectors.

### 5) First user message spacing fix

Upstream reference: `c091aa73`  
Status: Ported

What was done:

- Removed fragile `isFirstUserMessage` flag logic.
- Spacer insertion now checks `chatContainer.children.length > 0`.

Files:

- `nova/packages/coding-agent/src/modes/interactive/interactive-mode.ts`

Impact:

- Correct spacing after startup notices and restored sessions.
- Simpler, less stateful rendering logic.

### 6) GPT-5.5 context metadata correction

Upstream reference: `c96c2fcd`  
Status: Ported

What was done:

- Updated OpenAI metadata correction rule to include `gpt-5.5`.
- Corrected generated model metadata entries for `gpt-5.5` to:
  - `contextWindow: 272000`
  - `maxTokens: 128000`

Files:

- `nova/packages/ai/scripts/generate-models.ts`
- `nova/packages/ai/src/models.generated.ts`

Impact:

- Prevents overestimated context/token assumptions for GPT-5.5 routing/planning.

## Verified Already in Nova (No Additional Patch Needed)

Upstream branding cleanup reference: `de8c9475`  
Result: already present in Nova.

Checked areas already aligned:

- `APP_NAME` usage in CLI process title,
- slash command text (`/quit`),
- project-local extension path uses configurable directory (`CONFIG_DIR_NAME`),
- terminal title uses `APP_TITLE`.

## Intentionally Not Included in Phase D

- Changelog-only/doc-only upstream commits.
- Large generated model refreshes unrelated to concrete bugfixes.
- Higher-risk behavioral changes without immediate relevance for Nova’s current local-runtime priority.

Those are candidates for a later, dedicated sync pass.

## Validation

Executed after Phase D ports:

- `cd /Users/markusertel/code/pi-local-core/nova && npm run check`

Result:

- Passed (biome, type checks, browser smoke checks, web-ui checks).

## Phase D Outcome

Phase D is complete: the selected low-risk deltas from `v0.70.2` are now integrated in Nova with clean checks and no detected regressions in the current development workflow.
