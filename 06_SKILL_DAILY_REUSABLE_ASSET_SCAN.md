# Skill 06 — Daily Reusable Asset Scan

## Purpose

This skill ensures each day produces reusable organizational value, not only activity.

A staff member can be busy all day and still create no reusable value.

The GPT brain must repeatedly scan for reusable assets created or improved during the day.

## Core Rule

Every working day should create or improve at least one reusable, evidence-backed, queryable asset.

## Reusable Asset Types

Examples:

- capability file
- skill file
- reusable prompt
- validation rule
- evidence pack
- GitHub documentation
- PostgreSQL metadata proposal
- PostgreSQL view/table/function where approved
- n8n/OpenFlow workflow export
- decision framework
- data map
- duplicate-risk report
- query pack
- closure note
- SOP improvement
- testing checklist
- agent definition
- exception handling rule

## Scan Frequency

Run this scan:

- before lunch
- after major Claude output
- before commit
- before closure
- before shutdown

## Mandatory Questions

1. What reusable asset was created today?
2. What reusable asset was improved today?
3. What capability was discovered today?
4. What repeatable method was discovered today?
5. What evidence proves the asset exists?
6. Where is the asset stored?
7. Can GPT query it tomorrow?
8. Can an unknown developer use it tomorrow?
9. What should be promoted to a capability file?
10. What should be added to GitHub?

## Asset Quality Scoring

| Criterion | Points |
|---|---:|
| Clear purpose | 20 |
| Evidence attached | 20 |
| GitHub/approved location saved | 20 |
| Queryable by GPT | 20 |
| Reusable by unknown developer | 20 |

GREEN = 80–100
AMBER = 50–79
RED = below 50

## Daily Failure Condition

If no reusable asset was created or improved, the day is not fully successful.

Learning without capture is weak.
Discussion without capture is weak.
Claude output without saved asset is weak.
Local-only output is weak.

## Capability Extraction Trigger

If the task reveals a repeatable process, GPT must ask:

Should this become a reusable capability?

Examples:

- a better validation method
- a better prompt format
- a better workflow pattern
- a better evidence structure
- a better duplicate-check method
- a better closure format

## Required Daily Closure Output

At the end of the day, GPT should produce:

| Asset | Type | Location | Evidence | Reusable? | Next Step |
|---|---|---|---|---|---|

## Pass / Fail Rule

PASS if at least one reusable asset is created or improved, evidence exists, and location is recorded.

FAIL if the day ends with only chat, verbal explanation, or unsaved Claude output.
