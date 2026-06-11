# Skill 03 — Prompt Factory

## Purpose

This skill makes GPT responsible for producing high-quality Claude Code prompts.

Poor prompts create poor execution.
Vague prompts create vague assets.
Uncontrolled prompts create duplicate truth.

## Core Rule

All Claude Code task instructions must be created inside GPT first.

The user should not type free-form work instructions directly into Claude Code.

## Allowed Direct Claude Code Inputs

Normally allowed direct commands:

- commit
- push
- shutdown
- short confirmation requested by Claude Code
- emergency stop

All task instructions should come from GPT.

## Prompt Types

GPT should create different prompt types depending on the stage.

### 1. Discovery Prompt

Used before implementation.

Purpose:
Find existing assets, duplicate risk, data sources, related files, workflow exports, PostgreSQL objects, and evidence.

### 2. Validation Prompt

Used to test whether output is correct.

Purpose:
Check row counts, file existence, duplicate keys, evidence links, naming, metadata, and queryability.

### 3. Implementation Prompt

Used only after discovery.

Purpose:
Create or modify approved files, workflows, scripts, SQL, documentation, or assets.

### 4. Documentation Prompt

Used to create queryable explanation.

Purpose:
Explain what was created, why it exists, how it works, source evidence, limitations, owner, validator, and next step.

### 5. Closure Prompt

Used at the end.

Purpose:
Create handover, evidence summary, pass/fail status, Git path, and unknown developer readiness.

## Standard Prompt Template

Use this structure:

```text
You are working inside [project/repository/folder].

Objective:
[clear objective]

Context:
[why this work exists]

Before doing anything:
Search existing assets first across:
- GitHub/repo
- PostgreSQL objects if available
- n8n/OpenFlow workflow exports
- documentation
- skill files
- capability files
- evidence packs
- standards
- agents/memory assets

Scope:
You may:
[allowed actions]

You must not:
[forbidden actions]

Tasks:
1. [task]
2. [task]
3. [task]

Evidence required:
- [evidence item]
- [evidence item]

Output format:
| Finding | Evidence | Risk | Recommendation |

Stop conditions:
Stop and report if duplicate risk, source-of-truth conflict, production risk, missing evidence, or unclear business logic is found.

Pass/fail:
PASS if [measurable rule].
FAIL if [measurable rule].
```

## Prompt Quality Checklist

Before sending to Claude, GPT must check:

- Is the objective clear?
- Is discovery required?
- Are existing n8n assets included where relevant?
- Are existing GitHub assets included?
- Are existing PostgreSQL assets included?
- Is scope clear?
- Are forbidden actions clear?
- Is evidence required?
- Is output format clear?
- Is pass/fail measurable?
- Are stop conditions included?

## Failure Modes

| Prompt Failure | Likely Damage |
|---|---|
| No discovery instruction | Duplicate asset risk |
| No evidence requirement | Unproven work |
| No forbidden scope | Uncontrolled changes |
| No pass/fail rule | Cannot verify completion |
| No output format | Hard to review |
| No stop condition | Claude may continue when it should pause |

## Pass / Fail Rule

PASS if Claude Code receives a structured GPT-generated prompt with discovery, evidence, scope, output format, and pass/fail rule.

FAIL if Claude Code receives free-form task instructions directly from the user.
