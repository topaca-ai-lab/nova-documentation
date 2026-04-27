# Personal Installation

## Purpose

This guide explains how to install and validate Nova for personal use.
It is optimized for developers running Nova from a laptop/workstation with local or LAN-hosted model backends.

This page covers:

- prerequisites,
- global CLI installation,
- runtime model/provider configuration,
- validation commands,
- common failure modes and fixes.

---

## Target Audience

Use this guide if you are:

- a single user,
- operating Nova on your own machine,
- connecting to local or self-hosted OpenAI-compatible endpoints,
- not yet setting up multi-user governance or enterprise controls.

For server/multi-user scenarios, continue with:

- `business/server-deployment.md`
- `business/user-management.md`

---

## Prerequisites

## Operating System

Recommended:

- macOS (Apple Silicon or Intel)
- Linux
- WSL (with standard caveats for terminal behavior)

## Required Runtime

- Node.js `>=20`
- npm `>=10`

Check:

```bash
node -v
npm -v
```

---

## Installation Strategy

Nova is typically installed globally for personal usage.
That allows calling `nova` from any project directory.

Install:

```bash
npm install -g @topaca/nova-coding-agent
```

Verify:

```bash
nova --version
nova --help
```

If `nova` is not found, see Troubleshooting section "CLI not found".

---

## Recommended First Runtime Topology

For personal setups, the practical runtime order is:

1. LiteLLM + Ollama (primary default)
2. Ollama direct (high-reliability fallback)
3. LM Studio endpoint (local experimentation)

Reasoning:

- LiteLLM + Ollama provides a stable, centralized endpoint and flexible multi-model routing.
- Ollama direct is simple and often very reliable for strict tool-use behavior.
- LM Studio is useful but may vary in forced tool-call behavior depending on model/runtime wrapper.

---

## File Locations

Nova personal config files:

- Global model/provider registry: `~/.nova/agent/models.json`
- Global behavior defaults: `~/.nova/agent/settings.json`
- Project-local overrides: `<project>/.nova/settings.json`

Create global config directory if needed:

```bash
mkdir -p ~/.nova/agent
```

---

## Configure Providers and Models

Define dedicated providers per runtime class.
Do not mix unrelated runtime topologies into one provider.

Example `~/.nova/agent/models.json`:

```json
{
  "providers": {
    "litellm-remote": {
      "api": "openai-completions",
      "baseUrl": "http://<SERVER_IP>:4000/v1",
      "apiKey": "test",
      "models": [
        { "id": "qwen3.6-35b", "name": "qwen3.6-35b" }
      ]
    },
    "ollama-remote": {
      "api": "openai-completions",
      "baseUrl": "http://<SERVER_IP>:11434/v1",
      "apiKey": "test",
      "models": [
        { "id": "qwen3.6:35b", "name": "qwen3.6:35b" }
      ]
    },
    "lmstudio-local": {
      "api": "openai-completions",
      "baseUrl": "http://127.0.0.1:1234/v1",
      "apiKey": "lm-studio",
      "models": [
        { "id": "google/gemma-4-e4b", "name": "google/gemma-4-e4b" }
      ]
    }
  }
}
```

Notes:

- Keep model IDs exact as returned by backend `/v1/models`.
- For Ollama model IDs, `:` is normal (for example `qwen3.6:35b`).
- For OpenAI-compatible runtimes, use `.../v1` base URL.

---

## Optional: Set Personal Defaults

You can preselect default provider and model in `~/.nova/agent/settings.json`.

Example:

```json
{
  "defaultProvider": "litellm-remote",
  "defaultModel": "qwen3.6-35b"
}
```

This is optional but useful for predictable startup behavior.

---

## Validate Each Backend

Run capability probe per provider/model combo before normal usage.

## LiteLLM + Ollama

```bash
nova local doctor --json \
  --base-url http://<SERVER_IP>:4000 \
  --model qwen3.6-35b \
  --backend "LiteLLM+Ollama"
```

## Ollama direct

```bash
nova local doctor --json \
  --base-url http://<SERVER_IP>:11434 \
  --model qwen3.6:35b \
  --backend "Ollama"
```

## LM Studio

```bash
nova local doctor --json \
  --base-url http://127.0.0.1:1234 \
  --model google/gemma-4-e4b \
  --backend "LM-Studio"
```

---

## Interpret Doctor Output

Focus first on:

- `connectivity`
- `streaming`
- `toolCallingAuto`
- `toolCallingForced`
- `recommendedProfile`

Practical rule:

- For tool-heavy coding workflows, prioritize runtime/model combinations where `toolCallingForced=true`.

---

## Start Nova in a Project

Run Nova from the project you want to work in:

```bash
cd /path/to/project
nova
```

Inside Nova:

- use `/model` to explicitly select provider/model,
- keep long sessions on known-good combinations,
- re-run `doctor` after runtime changes.

---

## Minimal Personal Workflow

1. Start Nova in project directory.
2. Select provider/model via `/model`.
3. Confirm backend health with `doctor` whenever runtime changes.
4. Continue coding workflow.

Optional quality baseline:

```bash
nova local inspect --json
nova local bench run --json
```

---

## Common Problems and Fixes

## 1) `nova: command not found`

Cause:

- global package installed in a different Node environment,
- PATH missing active npm global bin.

Fix:

```bash
npm config get prefix
npm list -g --depth=0
hash -r
```

Ensure your shell loads the same Node environment you used for install.

## 2) Model not found

Cause:

- model ID mismatch,
- wrong provider selected,
- stale config.

Fix:

- verify `/v1/models` on backend,
- copy exact model ID,
- update `models.json`,
- restart Nova and reselect via `/model`.

## 3) Wrong model/provider active

Cause:

- default provider/model differs from expectation,
- previous session context persisted.

Fix:

- manually select via `/model`,
- review `settings.json` defaults,
- keep provider names explicit and unique.

## 4) Tool-calling instability

Cause:

- runtime/model capability differences.

Fix:

- compare `toolCallingAuto` and `toolCallingForced` across backends,
- move strict tool workflows to stronger combinations (often Ollama direct or LiteLLM+Ollama).

## 5) Endpoint reachable from server but not from workstation

Cause:

- service binding to localhost only,
- firewall or network path issues.

Fix:

- ensure backend binds to LAN interface (`0.0.0.0` when required),
- verify firewall rules,
- test from workstation with `curl`.

---

## Best Practices for Personal Setups

- Use dedicated provider names per backend class.
- Keep one known-good fallback model always configured.
- Avoid changing provider topology during long coding sessions.
- Document your known-good combos in personal notes.
- Re-validate after model updates or runtime upgrades.

---

## Security Notes

For personal setups:

- avoid committing private tokens to repositories,
- use low-privilege keys where possible,
- separate personal/lab credentials from business credentials.

---

## Next Documents

After installation, continue with:

- `quickstart/overview.md`
- `personal/llm-selection.md`
- `personal/install-skills.md`
- `personal/task-automation.md`
