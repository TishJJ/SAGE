# SAGE — Secure Adaptive Governance Engine

**Schema-first governance for CI/CD pipeline execution: not just security, but proof that a pipeline ran with the settings it was supposed to.**

## The Problem

Pipelines accumulate configuration from many sources — developer-supplied settings, organizational defaults, security requirements. Without enforcement, it becomes possible for a developer config (intentionally or not) to override a guardrail that should never be bypassed. There's no single trusted source of truth for "what actually ran."

## The Approach

SAGE is a **schema-first validation and governance model**. Rather than trusting configuration as given, every pipeline run passes through a structured gateway before anything executes.

### 1. The Gateway Phase
- The pipeline's environment is detected automatically.
- Three inputs are loaded: a user-supplied config, a restricted/secured config, and a master schema.
- Inputs are validated against the schema. Invalid configuration fails fast, before anything runs.
- If a setting exists in both the user config and the secure config, **the secure config always wins.** No override path exists for restricted settings.
- The result is a single, validated, immutable manifest — the one source of truth for that run.

### 2. The Distribution Phase
The validated manifest is split into scoped payloads, organized by job phase, following least-privilege:

- **Build** — compiler flags, runtime targets, build arguments
- **Test** — unit tests and other code-only tests; no infrastructure or deployment concerns
- **Scan** — security and other governance scans, run where applicable; outputs reports, SBOMs, and SARIF files for downstream consumption
- **Collect** — aggregates metadata from across the entire pipeline run (artifacts, build data, scan outputs, and other run-level data) for audit trail and decisioning
- **Deploy** — infrastructure targets, environment-specific parameters

Each downstream stage only receives the portion of the manifest relevant to it — nothing more.

Decisions about whether to proceed — to deployment, to further testing, or to the next phase — are made using the data collected across these phases, ensuring that code only advances when it adheres to policy, process, and governance requirements at each layer.

### 3. The Deployment Decision
Based on the validated manifest, the pipeline either:
- Deploys immediately using the deploy payload, or
- Stages the artifact and its validated deploy payload in a registry, ready to trigger later without re-running the full pipeline.

## Why It Matters

SAGE answers a different question than pure security scanning does: not "is this artifact safe" but **"did this pipeline run with the configuration it was supposed to, and can no one quietly subvert that?"**

It's the governance layer that a security verification engine (like SEVEN) can plug into — SAGE ensures the *inputs* and *execution path* are trustworthy; a security layer downstream verifies the *output* is trustworthy.

## Status

This is a conceptual architecture writeup describing a governance pattern, generalized from real design work. It is not tied to any specific organization's pipeline implementation.
