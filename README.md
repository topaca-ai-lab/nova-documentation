# Quickstart

## Goal

This section helps you get Nova running fast, with a backend setup that is stable for daily work.

Use this page as your entry point, then move to the detailed pages.

---

## Start Here

1. Read [Overview](./overview.md)
2. Choose your runtime mode
3. Validate with `nova local doctor`
4. Start `nova` in your project directory

---

## Recommended Runtime Order

For most users, this is the best order:

1. **LiteLLM + Ollama** (primary default)
2. **Ollama direct** (high-reliability fallback/reference)
3. **LM Studio** (local experimentation)

Why:

- LiteLLM + Ollama gives one stable endpoint and easier multi-model operations.
- Ollama direct tends to be very reliable for tool-heavy workflows.
- LM Studio is useful, but strict forced tool-calling may vary by model/runtime behavior.

---

## Minimal Setup Checklist

- Install Nova CLI
- Configure providers/models in `~/.nova/agent/models.json`
- (Optional) set defaults in `~/.nova/agent/settings.json`
- Run capability probe for each active backend

Validation command:

```bash
nova local doctor --json --base-url <BASE_URL> --model <MODEL_ID> --backend "<BACKEND_HINT>"
```

Inspect active profile:

```bash
nova local inspect --json
```

---

## Daily Usage

From your target repository:

```bash
cd /path/to/project
nova
```

Inside Nova:

- use `/model` to select provider/model explicitly,
- keep long sessions on known-good combinations,
- re-run `doctor` after backend/model changes.

---

## Common Capability Signals

When reading `doctor` output, prioritize:

- `connectivity`
- `streaming`
- `toolCallingAuto`
- `toolCallingForced`
- `recommendedProfile`

For strict tool workflows, `toolCallingForced=true` is a strong quality indicator.

---

## Where to Go Next

- [Overview](./overview.md)
- [Personal: Installation](../personal/installation.md)
- [Personal: LLM Selection](../personal/llm-selection.md)
- [Personal: Install Skills](../personal/install-skills.md)
- [Personal: Task Automation](../personal/task-automation.md)
- [Business: Server Deployment](../business/server-deployment.md)
- [API Reference: Edgent Bridge](../api-reference/edgent-bridge.md)
