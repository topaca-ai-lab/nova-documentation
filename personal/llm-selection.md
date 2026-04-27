# LLM Selection for Nova

## Purpose

This guide explains how to choose the right LLMs and runtime backends for Nova.
It focuses on practical engineering criteria, not model hype.

Use this document to:

- define selection criteria,
- compare candidate models consistently,
- assign primary and fallback models,
- avoid unstable tool-use setups.

---

## Why LLM Selection Matters in Nova

Nova is not a plain chat interface.
It is a tool-using coding agent workflow.

That means model selection must optimize for:

- reliable tool invocation,
- predictable behavior under constraints,
- reproducibility in multi-step tasks,
- stable output formats for automation paths.

A model that is "good at chat" is not automatically good for Nova agent execution.

---

## Core Selection Principles

## 1) Prefer capability stability over benchmark headlines

For Nova, stable behavior under repeated tool/action cycles is more important than peak benchmark numbers.

## 2) Validate in your real runtime path

Always test the exact combination:

- model,
- backend runtime,
- endpoint stack,
- provider profile.

Model behavior can change significantly between LM Studio, direct Ollama, and LiteLLM-routed setups.

## 3) Keep explicit fallback models

Always maintain:

- one primary model,
- one fallback model with known-good tool behavior.

---

## Mandatory Evaluation Criteria

These are non-negotiable for production-like Nova workflows.

## Tool Calling Reliability

Most important criterion for coding-agent usage.

Use `nova local doctor` and inspect:

- `toolCallingAuto`
- `toolCallingForced`

Guidance:

- `toolCallingAuto=true` is baseline,
- `toolCallingForced=true` is strongly preferred for strict orchestration workflows.

## Streaming Support

`streaming=true` should be present for responsive interactive usage.

## Structured Output Compatibility

`jsonSchema=true` is valuable for deterministic structured workflows.

## Role Handling

`systemRole` and `developerRole` support should be confirmed for robust instruction handling.

---

## Secondary Evaluation Criteria

These are important after mandatory checks pass.

## Reasoning Quality

Useful for:

- deeper multi-step planning,
- codebase analysis tasks,
- non-trivial refactoring guidance.

## Latency Profile

Measure practical response latency in your own environment.
A slightly weaker model with better latency can outperform in daily flow.

## Context Behavior

Evaluate long-context consistency under your project size and task complexity.

In Nova workflows, context window size should not be treated as a raw \"bigger is always better\" metric.
What matters is usable context quality over long agent sessions: stable retrieval of relevant prior turns, low drift in tool intent, and graceful degradation when nearing context limits.
When comparing models, include at least one long-session trial (multi-turn inspect/modify/execute cycle) and verify whether the model still follows constraints and tool semantics after context compaction.
This directly affects reliability in real coding sessions and should be part of the final model score.

## Hardware Fit and Throughput

Prefer models that match available VRAM/CPU constraints and expected concurrency.

---

## Runtime Stack Impact (Important)

Model quality is only one side.
Runtime path affects observed capability behavior.

## LiteLLM + Ollama

Strengths:

- one unified endpoint,
- easier multi-model routing,
- good operational flexibility.

Tradeoff:

- additional routing layer to operate.

## Ollama Direct

Strengths:

- simple topology,
- often strong tool-use reliability,
- straightforward diagnostics.

Tradeoff:

- less centralized routing/policy control than LiteLLM.

## LM Studio

Strengths:

- easy local experimentation,
- convenient workstation-centric testing.

Tradeoff:

- forced tool-calling behavior can be less consistent in some setups.

Practical recommendation:

- For strict tool-heavy workflows, prefer LiteLLM+Ollama or Ollama direct.

---

## Suggested Selection Workflow

## Step 1: Build candidate list

Create 3-6 realistic candidates per use case.
Do not over-expand the list.

## Step 2: Probe capabilities

Run for each candidate:

```bash
nova local doctor --json --base-url <BASE_URL> --model <MODEL_ID> --backend "<BACKEND_HINT>"
```

Store outputs for comparison.

## Step 3: Score with a fixed matrix

Use weighted criteria (example below).

## Step 4: Run short task trials

Test the model with representative Nova tasks:

- inspect + summarize,
- targeted edit,
- command execution with follow-up,
- multi-turn refinement.

## Step 5: Assign roles

Define explicitly:

- Primary model
- Fallback model
- Experimental model(s)

---

## Example Scorecard

Use a 1-5 scale and weighted score.

| Criterion | Weight | Notes |
| --- | ---: | --- |
| Tool calling auto | 20 | Must be reliable |
| Tool calling forced | 20 | Critical for strict workflows |
| Streaming quality | 10 | UX and interaction speed |
| JSON schema behavior | 10 | Structured workflows |
| Latency (practical) | 10 | Real environment, not claims |
| Reasoning quality | 10 | Multi-step coding tasks |
| Context consistency | 10 | Long sessions/repo work |
| Runtime stability | 10 | Error rate, retries, regressions |

Interpretation:

- >= 4.2 weighted average: strong primary candidate
- 3.6-4.1: strong fallback / secondary
- < 3.6: experimental only

---

## Recommended Model Portfolio Pattern

For personal/professional daily usage:

## Primary (default)

- Highest stability under tool use
- Good enough reasoning and latency

## Fallback

- Different architecture/runtime path than primary
- Known-good on tool reliability

## Specialist (optional)

- model selected for a niche, for example:
  - long-context heavy tasks,
  - faster low-cost interactive loops,
  - stronger reasoning passes.

---

## Decision Rules You Can Apply Immediately

- If `toolCallingForced=false`, do not use as sole primary for strict tool workflows.
- If two models are close, prefer the one with lower operational complexity.
- If runtime behavior differs by backend, treat them as different operational candidates even with same model family.
- Re-run capability probes after model updates/runtime upgrades.

---

## Common Selection Mistakes

## Choosing by raw model size only

Bigger is not always better for Nova workflows.
Reliability and latency matter more in practice.

## Ignoring runtime wrapper effects

Same model can behave differently across LiteLLM, Ollama direct, and LM Studio.

## No fallback strategy

Single-model setups increase downtime risk when behavior regresses.

## No periodic revalidation

Model/runtime updates can silently change tool behavior.

---

## Maintenance Cadence

Suggested routine:

- Weekly quick doctor checks for active models
- Monthly short scorecard refresh
- Revalidation after runtime/model upgrades

Keep a simple changelog of model decisions and observed regressions.

---

## Quick Commands

```bash
# Capability probe
nova local doctor --json --base-url <BASE_URL> --model <MODEL_ID> --backend "<BACKEND_HINT>"

# Inspect profile
nova local inspect --json

# Deterministic benchmark run
nova local bench run --json
```

---

## Practical Recommendation Snapshot

For most users running Nova today:

1. Use **LiteLLM + Ollama** as primary operational default.
2. Keep **Ollama direct** as robust fallback/reference.
3. Use **LM Studio** for local experimentation and rapid model trials.
4. Select final primary/fallback based on measured tool-use reliability, not model branding alone.
