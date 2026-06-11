# Project Runbook

**LEDs One · Automation Team** *Project Code: FIN-AECP · Version 1.0 · June 2026*

---

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PROJECT RUNBOOK

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Project Code:    FIN-AECP

Project Name:    Evri Claims Processing Automation

Workflow Name:   FIN-AECP-AD-evri claim process

Version:         1.0

Last Updated:    11 Jun 2026

Lead Developer:  Gishor  
Status:          Active — Pending Production Credential Swap (Node 13\)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

---

## Section 1 — Infrastructure Overview

N8N-only — no external VM, no Python service, no local server required.

All processing happens inside N8N. The only external dependencies are cloud APIs accessed via HTTP from N8N.

External services used:

Type:        External Cloud API

Name/ID:     LlamaIndex Cloud Parsing API

Location:    SaaS — api.cloud.llamaindex.ai

Purpose:     Converts invoice PDFs to Markdown text for AI extraction

Must be on:  Available on demand — called during each run

Managed by:  LlamaIndex (third-party SaaS) — API key held by team lead

Type:        Google Drive folder

Name/ID:     Evri Claims — Invoice Inbox folder

Location:    Google Drive — shared team drive

Purpose:     Staging folder where Jennani places invoice PDFs before each run

Must be on:  Available on demand — read during each run

Managed by:  Gishor account (Drive OAuth2 credential)

Type:        Google Sheet

Name/ID:     new \- Hermes Claims Forms

Location:    Google Drive — shared

Purpose:     Output destination — Claims tab receives the extracted rows

Must be on:  Available on demand — written to during each run

Managed by:  Gishor account (Sheets OAuth2 credential)

---

## Section 2 — Credential Setup

Complete all credentials before attempting a test run. A missing credential will cause a node failure that may look like a workflow logic error.

---

Credential Name:  Google Sheets OAuth2 — Gishor

Type:             OAuth2 (Google Sheets)

Used by:          Node 02 (Clear Claims Sheet), Node 15 (Append Claims Row)

How to obtain:

  1\. Request access from team lead — a Gishor Google account login is required

  2\. Confirm the account has edit access to the "new \- Hermes Claims Forms" sheet

  3\. Credentials (email/password) provided via team password manager

How to connect in N8N:

  1\. Go to N8N → Settings → Credentials → New

  2\. Select type: Google Sheets OAuth2 API

  3\. Click Connect — a browser window opens

  4\. Sign in with the Gishor Google account

  5\. Approve the permissions prompt (Sheets access)

  6\. Click Save

Expected test result:  "Connection successful"

Who to contact if stuck:  Team lead

---

Credential Name:  Google Drive OAuth2 — Gishor

Type:             OAuth2 (Google Drive)

Used by:          Node 03 (List Folder Files), Node 05 (Download File)

How to obtain:

  1\. Same Gishor Google account as Sheets credential above

  2\. Confirm the account has read access to the Evri Claims invoice folder

How to connect in N8N:

  1\. Go to N8N → Settings → Credentials → New

  2\. Select type: Google Drive OAuth2 API

  3\. Click Connect — sign in with Gishor Google account

  4\. Approve the permissions prompt (Drive access)

  5\. Click Save

Expected test result:  "Connection successful"

Who to contact if stuck:  Team lead

---

Credential Name:  LlamaIndex API — Bearer Token

Type:             HTTP Header Auth

Used by:          Node 07 (Upload File), Node 08 (Check Job Status), Node 11 (Fetch Markdown)

How to obtain:

  1\. Request the LlamaIndex API key from the team lead

  2\. The key is stored in the team password manager under "LlamaIndex Cloud"

  3\. Confirm the key is active — log in to cloud.llamaindex.ai and check account status

How to connect in N8N:

  ⚠ This credential is embedded directly in the HTTP Request node headers, not in

  N8N Credentials. The Bearer token is entered as the value of the Authorization

  header in Nodes 07, 08, and 11\.

  1\. Open Node 07 in the workflow

  2\. Under Headers → find the Authorization parameter

  3\. Set value to: Bearer \[token from password manager\]

  4\. Repeat for Node 08 and Node 11

  5\. Do not paste the token value into this document

Expected test result:  Node 07 returns a JSON object with an "id" field (LlamaIndex job ID)

Who to contact if stuck:  Team lead

---

Credential Name:  accounts team1 (OpenAI) — TEMPORARY

Type:             OpenAI API (used for Ollama compatibility endpoint)

Used by:          Node 13 (OpenAI Chat Model — connected to Node 12 AI Agent)

How to obtain:

  1\. Request the accounts team OpenAI credential from the team lead

  2\. This credential points to the local Ollama endpoint, not OpenAI directly

  3\. The base URL and API key are configured in the N8N credential

How to connect in N8N:

  1\. Go to N8N → Settings → Credentials → New

  2\. Select type: OpenAI API

  3\. API Key: \[value from team password manager — accounts team1\]

  4\. Base URL: \[Ollama server URL — from team lead\]

  5\. Click Save

Expected test result:  Node 12 runs and returns a JSON array in its output

Who to contact if stuck:  Team lead

⚠ PRODUCTION NOTE: This credential is temporary. Before production sign-off,

replace Node 13 with the local Ollama qwen25-seo:14b model and the correct

N8N Ollama credential. Do not use the accounts team OpenAI credential permanently.

---

## Section 3 — Infrastructure Setup (First Time Only)

No VM or server setup required. This project is N8N-only.

Steps for a new developer:

1. Obtain all four credentials listed in Section 2 from the team lead  
2. Connect all four credentials in N8N following Section 2  
3. Import the workflow if not already present in the N8N instance:  
   - Workflow name: FIN-AECP-AD-evri claim process  
   - Ask the team lead for the workflow JSON export if required  
4. In the imported workflow, update the LlamaIndex Bearer token in Nodes 07, 08, and 11 (HTTP header values)  
5. Verify the Google Drive folder ID in Node 03 points to the correct invoice inbox folder  
6. Verify the Google Sheet ID in Nodes 02 and 15 points to "new \- Hermes Claims Forms"  
7. Confirm Node 15 column mapping matches the actual column headers in the Claims tab (check for the CLIENT REFERNCE NO typo — header must match exactly)  
8. Run the pre-run verification checklist in Section 4 before the first test run

---

## Section 4 — Pre-Run Verification Checklist

Run this checklist before every manual test run, and on your first day picking up this project.

#### 4.1 Infrastructure Check

| Check | Action | Expected Result |
| :---- | :---- | :---- |
| LlamaIndex API is accessible | Open browser → [https://api.cloud.llamaindex.ai](https://api.cloud.llamaindex.ai) | No connection error |
| Google Drive folder is accessible | Open Gishor Drive → Evri Claims — Invoice Inbox | Folder opens, shows uploaded PDFs |
| Claims sheet is accessible | Open Google Drive → new \- Hermes Claims Forms → Claims tab | Sheet opens without permission error |
| Invoice PDFs are in the folder | Check folder contents in Drive | Only the current batch PDFs are present — no old files |

#### 4.2 Credential Check

| Credential | How to verify | Expected Result |
| :---- | :---- | :---- |
| Google Sheets OAuth2 — Gishor | N8N → Settings → Credentials → find credential → Test | Connection successful |
| Google Drive OAuth2 — Gishor | N8N → Settings → Credentials → find credential → Test | Connection successful |
| LlamaIndex Bearer Token | Open Node 07 → verify Authorization header value is present | Value begins with "Bearer llx-" |
| accounts team1 OpenAI (temp) | N8N → Settings → Credentials → accounts team1 → Test | Connection successful |

#### 4.3 N8N Workflow Check

| Check | Action | Expected |
| :---- | :---- | :---- |
| Workflow exists | N8N → search: FIN-AECP-AD-evri claim process | Workflow visible |
| TCH-SYSLOG-WF01 is active | N8N → search TCH-SYSLOG-WF01 | Status: Active |

---

## Section 5 — Running the Project

#### 5.1 Scheduled Run (Normal Operation)

Not applicable — this project has no scheduled trigger. Every run is manual.

#### 5.2 Manual Trigger

Before triggering — confirm:

  1\. Jennani has placed all invoice PDFs in the Evri Claims — Invoice Inbox folder

  2\. No old/unrelated PDFs are in the folder

  3\. Section 4 checklist is complete

Steps to trigger:

  1\. Open N8N

  2\. Search for workflow: FIN-AECP-AD-evri claim process

  3\. Click "Execute Workflow" (top right button)

  4\. No test parameters needed — the workflow reads whatever is in the folder

  5\. Watch execution:

     \- Node 02: should clear quickly (no error)

     \- Node 03: should return a list of files

     \- Node 06 loop: iterates once per PDF — this is the slow part

     \- Node 09: will show PENDING → wait loops for each file before SUCCESS

     \- Node 15: appends rows — watch for green checkmarks

  6\. Once all iterations complete, inform Jennani the run is done

  7\. Verify the output: open Claims tab → confirm rows are present with today's run data

**Expected runtime:** 3–8 minutes depending on the number of invoices (approximately 30–60 seconds per invoice including LlamaIndex parsing time)

**What a successful run looks like:**

- Claims tab contains new rows — one per product line across all invoices  
- No red error nodes in the N8N execution view  
- BRAND NAME column shows "Ledsone" in every row  
- HERMES BARCODE column is populated for most rows (blank only if not found on invoice)

---

## Section 6 — Service Management

Not applicable — N8N-only project. There is no external service or systemd process to manage.

If N8N itself is unavailable, contact whoever manages the N8N instance.

---

## Section 7 — Troubleshooting

---

Symptom:    Node 07 (LlamaIndex Upload) fails with a network or 401 error

Cause:      LlamaIndex Bearer token is expired or incorrect

Confirm:    Open Node 07 → check the Authorization header value is present and begins with "Bearer llx-"

            Try opening https://api.cloud.llamaindex.ai in a browser — confirm it responds

Fix:

  1\. Request a fresh API token from the team lead

  2\. Update the Authorization header value in Nodes 07, 08, and 11

  3\. Re-run the workflow to confirm

---

Symptom:    Node 03 (List Folder Files) returns 0 files

Cause:      Either the folder is empty, or the folder ID in the node is incorrect

Confirm:    Open the Evri Claims — Invoice Inbox folder in Drive — confirm PDFs are present

Fix:

  Option A — folder is empty: Ask Jennani to upload the invoice PDFs first

  Option B — folder ID mismatch: Open Node 03 in N8N → check the folder query parameter

             Ask the team lead for the correct folder ID and update the node

---

Symptom:    Node 02 (Clear Claims Sheet) fails with a permissions error

Cause:      Gishor Google Sheets credential has lost access or expired

Confirm:    N8N → Settings → Credentials → Google Sheets OAuth2 — Gishor → Test

            If test fails: credential needs to be reconnected

Fix:

  1\. Go to N8N → Settings → Credentials → Google Sheets OAuth2 — Gishor

  2\. Click Reconnect → sign in with Gishor Google account

  3\. Approve permissions

  4\. Save and test again

  5\. Re-run the workflow

---

Symptom:    Node 09 Switch stays in PENDING loop for more than 5 minutes on one file

Cause:      LlamaIndex is slow to parse a specific PDF, or the PDF is malformed

Confirm:    Watch the loop counter in N8N — if it cycles more than 30 times on one file,

            the parsing has stalled

Fix:

  1\. Stop the N8N execution

  2\. Remove the problematic PDF from the folder

  3\. Re-run with the remaining PDFs

  4\. Process the problematic PDF separately and inspect it manually

  5\. If LlamaIndex consistently fails on it, the PDF may need to be re-exported from the source

---

Symptom:    Node 12 (AI Agent) returns an error or the output is not a JSON array

Cause:      Node 13 (AI model) credential is not working, or model returned malformed output

Confirm:    Click Node 12 in the execution view → check the output field → is "output" a valid JSON array?

            If not: check Node 13 credential status

Fix:

  1\. N8N → Settings → Credentials → accounts team1 → Test

  2\. If test fails: request updated credential from team lead

  3\. If credential is fine but output is malformed: re-run once — AI models occasionally produce

     bad output on the first attempt; the code node (Node 14\) will skip malformed items

---

Symptom:    Node 15 (Append Claims Row) fails with "column not found" or writes to wrong columns

Cause:      Column headers in the Claims sheet do not match the mapping in Node 15

Confirm:    Open Claims tab → check column headers in row 1

            Compare to Node 15 mapping in N8N — particularly "CLIENT REFERNCE NO" (note typo)

Fix:

  1\. If headers were accidentally renamed: restore the original header names in the sheet

  2\. If the sheet was recreated: ensure all required column headers are present in row 1,

     with exact names matching Node 15 (including the CLIENT REFERNCE NO typo)

  3\. Do not correct the typo in N8N without also correcting it in the sheet simultaneously

---

Symptom:    Claims tab still shows data from the previous run after a new run completes

Cause:      Node 02 (Clear Claims Sheet) failed or was skipped

Confirm:    Check the N8N execution history — did Node 02 complete successfully?

Fix:

  1\. If Node 02 failed: see Sheets credential troubleshooting above

  2\. If Node 02 was skipped: check for an accidental connection change in the workflow

  3\. Manually clear the Claims tab (A2:M1000) and re-run

---

## Section 8 — Updating and Redeploying

This project contains no external scripts or services to deploy. All logic is inside N8N nodes.

**To update the AI extraction prompt (Node 12 system message):**

1\. Open N8N → FIN-AECP-AD-evri claim process

2\. Click Node 12 (AI – Parse Invoice Data)

3\. Edit the system message in the prompt options

4\. Save the node

5\. Run a test with a sample invoice to confirm the updated prompt extracts correctly

6\. Update this Runbook if the change affects any documented behaviour

**To swap Node 13 to local Ollama (production requirement):**

1\. Confirm qwen25-seo:14b is running on the local Ollama server

2\. In N8N → Settings → Credentials → create a new Ollama credential pointing to the local endpoint

3\. Open Node 13 in the workflow → change the credential to the new Ollama credential

4\. Update the model name to qwen25-seo:14b

5\. Disconnect the accounts team1 OpenAI credential from Node 13

6\. Run a test with 3 sample invoices to confirm extraction quality matches previous results

7\. Update Section 2 of this Runbook to replace the accounts team1 entry with the Ollama credential

8\. Remove the ⚠ TEMPORARY note from the Node 13 credential section

---

## Section 9 — Ownership

Lead Developer:   Gishor

Maintainer:       Automation Team (ongoing changes and prompt updates)

Infrastructure:   Not applicable — N8N-only project

Credentials:      Team lead — holds LlamaIndex API token and Gishor account access

Business contact: Jennani (Finance Team) — invoice batch preparation and output review

---

*LEDs One · Automation Team · FIN-AECP Project Runbook v1.0 · Jun 2026*  
