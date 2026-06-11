# Skill 05 — Duplicate Truth Prevention

## Purpose

This skill prevents the organization from creating multiple conflicting sources of truth.

Duplicate truth is more dangerous than missing truth because it makes future decisions unreliable.

## What Counts As Duplicate Truth

Duplicate truth can appear as:

- two documents saying different rules
- two PostgreSQL tables representing the same business concept
- two n8n workflows doing the same task
- two prompt files giving different instructions
- two capability files for the same capability
- dashboard logic that differs from PostgreSQL logic
- hardcoded rule inside code that differs from BLOS/configuration
- local file that differs from GitHub
- verbal rule that differs from documented rule

## Core Rule

One business rule should have one canonical source.

Other assets may reference it, but must not silently redefine it.

## Duplicate Detection Questions

GPT must ask:

1. Is this business rule already defined somewhere?
2. Is this workflow already built somewhere?
3. Is this data already represented somewhere?
4. Is this capability already captured somewhere?
5. Is this prompt already available somewhere?
6. Is this document creating a new source of truth?
7. Should this asset extend an existing source instead?

## Duplicate Risk Levels

| Level | Description | Decision |
|---|---|---|
| GREEN | No duplicate found | Continue |
| AMBER | Partial overlap found | Merge or extend |
| RED | Existing source already exists | Stop new creation |

## Required Claude Output

When duplicate risk exists, Claude must output:

| New Asset | Existing Asset | Overlap | Risk | Recommendation |

GPT must not allow Claude to decide deletion or merge without review.

## Merge / Extend Rule

If an existing asset partially covers the requirement, improve it unless there is a strong reason to create a new one.

Reasons to create new may include:

- different domain
- different grain
- different validated source
- different execution cadence
- different owner
- existing asset is deprecated and marked as such

Reasons must be documented.

## Failure Modes

| Failure | Consequence |
|---|---|
| Two standards for same process | Staff follow different rules |
| Two workflows for same automation | Double execution or conflict |
| Two tables for same metric | Different answers to same question |
| Two prompts for same Claude task | Inconsistent execution |
| Local truth not pushed to GitHub | GPT cannot query it later |

## Pass / Fail Rule

PASS if duplicate risk is checked and documented.

FAIL if a new source of truth is created without proving uniqueness.
