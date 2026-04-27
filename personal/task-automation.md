# Task Automation

## Overview

Task automation in Nova turns repeated personal workflows into reliable, scheduled agent runs.

Common examples:

- daily planning summaries,
- inbox triage preparation,
- weekly project status digests,
- reminder and follow-up loops,
- periodic maintenance checks (RAG, memory, connectors).

The goal is not “maximum automation”, but *controlled automation with predictable outcomes*.

---

## Automation Modes

Nova supports two useful patterns:

### 1) Thread heartbeat automation

Use when you want continuity in the same conversation/thread.

Best for:

- iterative tasks,
- ongoing investigations,
- recurring progress checks.

### 2) Cron-style recurring jobs

Use when you want independent scheduled runs.

Best for:

- routine reports,
- system checks,
- maintenance tasks with stable outputs.

---

## Personal Automation Design Principles

Follow these principles for reliable outcomes:

- define one clear objective per automation,
- keep prompts deterministic and concise,
- constrain output format,
- separate data collection from side effects,
- add explicit failure handling.

Avoid broad prompts like “manage my day.”
Prefer concrete instructions like:

- “Generate a 10-item daily priority list from tagged project notes.”

---

## Automation Blueprint

Each automation should define:

1. trigger schedule,
2. input sources,
3. model/backend profile,
4. execution constraints,
5. output target,
6. fallback behavior.

### Trigger schedule

Examples:

- every weekday at 08:00,
- every 30 minutes during working hours,
- hourly during incident windows.

### Input sources

Use explicit sources only:

- selected folders,
- tagged notes,
- specific connectors.

### Model/backend profile

Choose per task class:

- low-cost local model for routine summarization,
- stronger model only for complex reasoning tasks.

### Execution constraints

Set:

- timeout limits,
- retry limits,
- maximum context/input size,
- tool/action allowlists.

### Output target

Define exactly where results go:

- conversation output,
- markdown report file,
- message gateway notification.

### Fallback behavior

On failure:

- retry within policy,
- produce compact error report,
- avoid silent failure.

---

## High-Value Personal Automations

### Daily Start Pack

At the start of your day:

- summarize open tasks,
- collect upcoming events,
- list top priorities by project impact.

### Inbox Prep

Before mail processing:

- cluster messages by urgency,
- extract required actions,
- generate draft response intents.

### Weekly Review

Once per week:

- summarize completed work,
- list blocked items,
- produce next-week focus plan.

### Knowledge Hygiene

Periodic maintenance:

- detect stale notes,
- flag duplicates,
- suggest merge/archive actions.

### System Health Check

Regular diagnostic routine:

- run local doctor checks,
- inspect connector health,
- verify memory/RAG integrity metrics.

---

## Safety Controls for Automation

Always apply layered safeguards.

Minimum baseline:

- read-only by default,
- explicit permission for write/send/delete actions,
- scoped connector credentials,
- per-automation allowlists for tools/actions,
- budget limits for runtime and payload size.

For sensitive automations:

- require confirmation step before side effects,
- enforce stricter retry and timeout policies,
- keep detailed run history for audit.

---

## Prompt Template Guidance

A good automation prompt includes:

- objective,
- exact input scope,
- decision criteria,
- output schema,
- do-not-do constraints.

Template:

1. Task objective
2. Data sources
3. Required output format
4. Quality constraints
5. Safety constraints

This reduces drift and improves reproducibility across model changes.

---

## Monitoring and Operations

Track every recurring automation with:

- run count,
- success/failure/cancel rates,
- average runtime,
- average retry count,
- error class distribution.

Add alerting for:

- repeated failures,
- unusual latency spikes,
- empty or malformed outputs.

Weekly review is enough for personal setups, unless the workflow is business-critical.

---

## Incident Handling

When an automation misbehaves:

1. pause the automation,
2. inspect last successful run and first failed run,
3. identify whether failure is model, connector, data, or policy related,
4. patch one variable at a time,
5. resume with close monitoring.

Keep a simple rollback version of every critical automation prompt/config.

---

## Integration with Skills and RAG

Automation quality depends on upstream data quality.

Recommended chain:

1. stable skills,
2. clean RAG ingestion and retrieval,
3. automation workflows on top.

Do not automate unstable foundations.
If retrieval or connector behavior is noisy, fix that first.

---

## Example Rollout Plan

Week 1:

- deploy one read-only daily summary automation.

Week 2:

- add weekly review automation.

Week 3:

- add one side-effect automation with confirmations (for example reminder gateway).

Week 4:

- harden monitoring and failure alerts.

This staged rollout prevents cascading failures and keeps trust in the system high.

---

## Best Practices Recap

- Keep automations narrow and explicit.
- Use least privilege everywhere.
- Track metrics and failure context from day one.
- Treat prompt/version changes like code changes.
- Scale complexity only after stability is proven.

---

## Related Documentation

- Skills setup: [install-skills.md](./install-skills.md)
- Document grounding and retrieval: [rag-documents.md](./rag-documents.md)
