# Skill 04 — Existing Asset First

## Purpose

This skill prevents duplicate assets, duplicate workflows, duplicate PostgreSQL objects, duplicate documentation, duplicate prompts, and duplicate business logic.

Creating new work before checking existing work is a governance failure.

## Core Rule

Before creating anything new, GPT must require an existing asset search.

This applies to:

- documentation
- workflows
- n8n automations
- OpenFlow automations
- PostgreSQL tables/views/functions
- APIs
- scripts
- prompts
- skill files
- capability files
- evidence packs
- registers
- dashboards
- agents
- decision rules
- validation rules
- SOPs

## Mandatory Search Areas

Claude Code prompts must instruct discovery across relevant areas:

1. GitHub repository
2. PostgreSQL schemas and metadata where approved
3. n8n workflow exports
4. OpenFlow workflow exports
5. local VM project folders
6. documentation folders
7. skill files
8. capability files
9. evidence folders
10. decision frameworks
11. validation assets
12. previous Claude outputs saved in files
13. existing agent definitions
14. memory assets
15. SOPs and standards

## Decision Order

After discovery, GPT must decide in this order:

1. Reuse existing asset
2. Extend existing asset
3. Merge with existing asset
4. Create new asset only if uniqueness is proven

Creation is the final option.

## Duplicate Risk Classification

| Level | Meaning | Action |
|---|---|---|
| GREEN | No relevant existing asset found | New asset may be created |
| AMBER | Similar asset exists | Extend or merge unless clear reason |
| RED | Existing asset already covers objective | Do not create new asset |

## Required Evidence For New Asset Creation

Before approving a new asset, Claude must provide:

- search locations checked
- matching files found
- matching workflows found
- matching PostgreSQL objects found
- overlap assessment
- reason why reuse/extend/merge is not enough

## n8n Specific Rule

Before creating any n8n workflow, search existing n8n assets first.

Check:

- existing workflow names
- workflow exports
- trigger type
- data source
- destination
- business purpose
- owner
- last run evidence
- duplicate or partial workflow

Do not create a new workflow if an existing one can be extended.

## PostgreSQL Specific Rule

Before creating any PostgreSQL object, check:

- existing tables
- existing views
- existing functions
- existing schemas
- existing metadata
- existing comments
- existing validation queries
- existing similar object names

Do not create parallel truth.

## Documentation Specific Rule

Before creating new documentation, check:

- existing README
- existing standards
- existing capability file
- existing skill file
- existing closure note
- existing handover file

Prefer improving the existing document.

## Failure Modes

| Failure | Damage |
|---|---|
| New n8n workflow without search | Duplicate automation |
| New table without schema search | Parallel data truth |
| New skill file without source check | Conflicting rules |
| New documentation without README check | Fragmented knowledge |
| New prompt library without checking existing prompts | Prompt duplication |

## Mandatory GPT Question

What existing asset already solves this fully or partially?

## Pass / Fail Rule

PASS if existing asset search is evidenced before creation.

FAIL if new assets are created without discovery evidence.
