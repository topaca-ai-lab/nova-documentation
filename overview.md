# Nova Quickstart Overview

## Purpose

This document gives a practical, implementation-oriented overview of Nova.
It is intentionally general and should remain valid across different local and hybrid runtime setups.

It focuses on:

- how Nova is structured,
- which runtime options are available,
- how to choose the right backend profile,
- how to validate capabilities before daily use.

---

## What Nova Is

Nova is a coding agent platform for terminal-first engineering workflows.
It combines an interactive CLI agent with a flexible model/provider abstraction so you can run the same workflows across different inference backends.

Core characteristics:

- local-first operation,
- backend flexibility (OpenAI-compatible runtimes),
- tool-driven agent execution,
- reproducible diagnostics and capability checks.

---

## Core Runtime Modes

Nova can be operated with multiple backend topologies.
The following are common and field-tested options.

### 1) LiteLLM + Ollama (recommended default)

Nova calls a LiteLLM OpenAI-compatible endpoint, and LiteLLM routes requests to one or multiple Ollama models.

Why this is recommended as the default:

- one stable endpoint for Nova,
- model routing and policy control in one place,
- easier multi-model management,
- better long-term maintainability for personal and team environments.

Use this when you want operational flexibility and centralized control.

### 2) Ollama directly

Nova calls Ollama's OpenAI-compatible endpoint directly.

Why this is a strong option:

- fewer components,
- simple setup,
- very reliable tool-use behavior in many practical setups.

For local/self-hosted tool-heavy workflows, Ollama direct is generally preferred over LM Studio.

### 3) LM Studio endpoint

Nova calls the LM Studio OpenAI-compatible endpoint.

When to use it:

- local experimentation,
- workstation-centric model testing,
- ad-hoc model comparison.

Important note:

- Tool-use behavior can be less strict than Ollama in some setups (especially under forced tool-call scenarios).

---

## Recommendation Summary

If you need a single practical recommendation:

1. Start with **LiteLLM + Ollama** as your primary runtime.
2. Keep **Ollama direct** as a high-reliability fallback and benchmark reference.
3. Use **LM Studio** mainly for local experimentation, not as first choice for strict tool orchestration.

---

## Configuration Model

Use dedicated providers per runtime in:

- `~/.nova/agent/models.json`

Do not overload one provider for multiple backend types.
Separate providers improve:

- predictability,
- easier troubleshooting,
- safer model switching.

### Typical provider segmentation

- one provider for LiteLLM-routed models,
- one provider for direct Ollama models,
- one provider for LM Studio models.

---

## Important Config Files

### `~/.nova/agent/models.json`

Provider and model registry.
Use this for endpoint definitions, model IDs, and runtime-specific provider separation.

### `~/.nova/agent/settings.json`

Global defaults and behavior preferences.
Use this for default provider/model and general runtime/UI settings.

### `.nova/settings.json` (project-local)

Project-scoped overrides.
Use this for repository-specific defaults and team reproducibility.

---

## Validation Workflow (Required)

Before using a new runtime/model combination in daily work:

```bash
nova local doctor --json --base-url <BASE_URL> --model <MODEL_ID> --backend "<BACKEND_HINT>"
```

Then inspect profile resolution:

```bash
nova local inspect --json
```

For baseline quality comparisons:

```bash
nova local bench run --json
```

---

## How to Read Doctor Output

Key fields:

- `connectivity` - endpoint reachability
- `streaming` - streaming response support
- `toolCallingAuto` - tool calls in normal auto mode
- `toolCallingForced` - reliability under forced tool calls
- `jsonSchema` - structured output contract support
- `recommendedProfile` - capability-based runtime profile suggestion

Interpretation rule:

- For tool-heavy coding agent workflows, `toolCallingForced=true` is a strong quality signal.

---

## Operational Best Practices

- Keep runtime providers explicit and isolated.
- Keep model IDs exact as returned by `/v1/models`.
- Validate after every backend or model change.
- Prefer known-good provider/model pairs for longer sessions.
- Use benchmark artifacts to compare runtime quality over time.

---

## Troubleshooting Quick Map

### `command not found: nova`

- verify global install in active Node environment,
- verify PATH includes global npm bin,
- run `hash -r`.

### Model not found

- verify exact model ID from backend model list,
- verify provider entry in `models.json`,
- reselect via `/model`.

### Wrong model appears active

- verify provider separation,
- check default provider/model settings,
- explicitly switch via `/model`.

### Tool-use instability

- compare `toolCallingAuto` vs `toolCallingForced`,
- move critical workflows to Ollama direct or LiteLLM+Ollama,
- keep LM Studio for exploratory/local-only runs when needed.

---

## Security and Governance Notes

- Do not store sensitive production credentials in shared docs or repositories.
- Prefer least-privilege secrets.
- Keep personal, lab, and production runtime configs clearly separated.

---

## Licensing Context

Nova uses dual licensing:

- AGPL-3.0 for open-source/community use,
- commercial licensing for proprietary/closed-source deployment.

Plan compliance boundaries early for business deployments.

---

## Suggested Next Reading

1. `quickstart/README.md`
2. `personal/installation.md`
3. `personal/rag-documents.md`
4. `business/server-deployment.md`
5. `api-reference/edgent-bridge.md`

---

## Command Reference

```bash
# Start Nova in current project
nova

# Capability probe
nova local doctor --json --base-url <BASE_URL> --model <MODEL_ID> --backend "<BACKEND_HINT>"

# Inspect resolved profile
nova local inspect --json

# Run benchmark suite
nova local bench run --json
```
