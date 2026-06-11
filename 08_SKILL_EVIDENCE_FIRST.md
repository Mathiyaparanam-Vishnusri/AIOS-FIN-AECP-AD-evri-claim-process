# Skill 08 — Evidence First

## Purpose

This skill prevents false completion.

No evidence = no completed work.

## Core Rule

Every completion claim must be backed by evidence.

If evidence is missing, the work status is UNPROVEN.

## Accepted Evidence

- Git commit hash
- GitHub file path
- PostgreSQL query output
- PostgreSQL object name
- SQL file
- n8n workflow export
- OpenFlow workflow export
- execution log
- API response log
- validation test result
- screenshot saved internally
- evidence markdown file
- approved source document
- closure handover file

## Unaccepted Evidence

- “I did it”
- “Claude said it completed”
- “It works”
- “I checked manually”
- “It is on my computer”
- “I will upload later”
- “We discussed it”
- “Almost done”

## Evidence Package Minimum Fields

Every important evidence pack should include:

- evidence ID or file name
- date captured
- captured by
- source system
- method used
- result summary
- files touched
- validation status
- caveats
- next step

## Evidence Status

| Status | Meaning |
|---|---|
| DRAFT | collected but not validated |
| PARTIAL | some proof exists but gaps remain |
| VALIDATED | reviewer accepted |
| REJECTED | evidence unreliable |
| SUPERSEDED | replaced by newer evidence |

## GPT Evidence Review

GPT must classify each claim:

| Claim | Evidence | Status | Gap |
|---|---|---|---|

Statuses:

- VERIFIED
- PARTIAL
- UNPROVEN
- ASSUMPTION

## Failure Modes

| Failure | Damage |
|---|---|
| Work completed without Git path | Cannot inspect later |
| SQL result not saved | Cannot validate later |
| Claude output not saved | Discovery lost |
| Screenshot not stored | Proof disappears |
| Evidence not linked to asset | Asset cannot be trusted |

## Pass / Fail Rule

PASS if every completion claim has saved evidence.

FAIL if completion depends on trust, memory, or chat only.
