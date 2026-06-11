# Skill 09 — Queryability First

## Purpose

This skill ensures assets can be understood by GPT, Claude, and future users.

An asset that cannot be queried later is weak.

## Core Rule

Every important asset must be self-explanatory enough for a clean LLM to understand it later.

## Required Asset Questions

Every asset should answer:

1. What is this?
2. Why does it exist?
3. What problem does it solve?
4. What system or process does it belong to?
5. What source data or files does it use?
6. What evidence supports it?
7. What is the current status?
8. What are the known limits?
9. Who should review it?
10. What is the next step?

## Queryability Test

Ask GPT:

Using only this asset and its linked evidence, explain:

- purpose
- source
- method
- evidence
- status
- risk
- next step

If GPT cannot answer, the asset fails queryability.

## Self-Explanatory Asset Requirements

Every important file should include:

- title
- purpose
- context
- source
- evidence
- owner/reviewer
- status
- next step
- pass/fail rule

## PostgreSQL Queryability

PostgreSQL objects should have:

- business-readable name
- purpose
- source tables
- grain
- key fields
- refresh logic
- validation query
- limitations
- owner/reviewer

## Workflow Queryability

n8n/OpenFlow workflows should document:

- workflow name
- trigger
- input
- output
- business purpose
- owner
- evidence of last test/run
- failure handling
- duplicate check result

## Failure Modes

| Failure | Damage |
|---|---|
| File name vague | GPT cannot locate purpose |
| No evidence link | Output untrusted |
| No next step | Work stalls |
| No source | Business meaning unclear |
| No owner/reviewer | No accountability |
| No status | Unknown readiness |

## Pass / Fail Rule

PASS if a clean LLM can explain the asset without asking the creator.

FAIL if verbal explanation is required.
