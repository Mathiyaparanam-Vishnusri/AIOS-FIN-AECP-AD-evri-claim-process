# Skill 11 — Continuous Drift Prevention

## Purpose

This skill prevents work from drifting away from the original objective.

Drift is expensive because it creates activity without value.

## Core Rule

Run drift checks repeatedly throughout the day.

Recommended times:

- before starting work
- after Claude discovery
- before implementation
- before creating new assets
- before creating new n8n workflows
- before creating PostgreSQL objects
- before lunch
- mid-afternoon
- before closure
- before commit
- before push
- before shutdown

## Drift Types

| Drift Type | Example |
|---|---|
| Objective drift | working on documentation instead of solving original problem |
| Asset drift | creating new file instead of improving existing file |
| Claude drift | Claude expands scope beyond prompt |
| Evidence drift | claims progress without proof |
| Queryability drift | output cannot be understood later |
| ROI drift | effort does not affect business value |
| Duplication drift | similar asset already exists |
| Closure drift | no handover or next step |

## Mandatory Drift Questions

1. Are we still solving the original problem?
2. What business question are we answering?
3. What evidence proves progress?
4. What existing asset was checked first?
5. What duplication risk exists?
6. What reusable asset was created or improved?
7. What capability was extracted?
8. Can an unknown developer continue?
9. Is GPT still directing Claude Code?
10. Should we continue, correct, or stop?

## Drift Score

Score 0–10:

| Area | Score | Evidence | Gap |
|---|---:|---|---|
| Objective Alignment | | | |
| Existing Asset Discovery | | | |
| Duplicate Prevention | | | |
| Evidence Quality | | | |
| Reusability | | | |
| Queryability | | | |
| Unknown Developer Readiness | | | |
| GPT Control of Claude | | | |
| ROI / Business Value | | | |
| Closure Quality | | | |

GREEN = 8–10
AMBER = 5–7
RED = below 5

## Recovery Actions

If AMBER:

- identify missing evidence
- check existing assets
- update asset documentation
- create closure note
- paste Claude output into GPT
- generate correction prompt

If RED:

- stop current direction
- return to original objective
- perform discovery
- identify duplicate risks
- create a recovery prompt

## Pass / Fail Rule

PASS if drift is detected early and corrected.

FAIL if drift continues into file creation, commit, push, or closure.
