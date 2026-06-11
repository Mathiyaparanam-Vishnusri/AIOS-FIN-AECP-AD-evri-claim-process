━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ PROJECT CONTEXT DOCUMENT & DEVELOPER GUIDE ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ Project Code:    AECP Project Name:    FIN-AECP-AD-evri claim processVersion:         1.0 Last Updated:    29 May 2026 Lead Developer:  Jennani Status:          Active ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

---

## Section 1 — Project Overview

The Hermes Claims Form Automation project processes PDF invoices stored in a Google Drive folder and populates a Hermes claims sheet automatically. Previously, the postage team had to manually read each invoice, extract customer details, tracking numbers, and product values, and type them into the claims spreadsheet — a time-consuming and error-prone process. This workflow reads every PDF in the source folder, parses the invoice content using LlamaIndex (OCR/parsing) and a Groq LLM, cleans and normalises the extracted data, and appends one row per product line to the "Claims" tab in the Hermes Claims Google Sheet. The primary user of the output is Jennani (Postage Team). The workflow is manually triggered by a developer each time a new batch of invoices needs processing.

---

## Section 2 — Platform Map

Platform:    N8N

Role:        Orchestration — triggers the run, loops through files, coordinates all steps

Why used:    Primary automation platform

Resource:   FIN-AECP-AD-evri claim process

Credential:  N/A (internal)

Platform:    Google Drive (Source — Invoice PDFs)

Role:        Source folder containing the PDF invoices to be processed

Why used:    Invoices are stored here by the postage team before processing

Resource:    Folder ID: \[DRIVE\_FOLDER\_IDi\]

         (All PDF files inside this folder are processed each run)

Credential:  GISHOR (googleDriveOAuth2Api)

Platform:    Google Sheets (Output — Hermes Claims)

Role:        Receives one row per invoice product line after each run

Why used:    Hermes claims submission format — postage team reads and submits from here

Resource:    Sheet name: new \- Hermes Claims Forms

         Sheet ID:   \[SHEET\_ID — new \- Hermes Claims Forms\]

         Tab name:   Claims (\[tab gid\])

         Tab range cleared before write: A2:M1000

Credential:  Gishor (googleSheetsOAuth2Api)

Platform:    LlamaIndex Cloud (PDF Parsing API)

Role:        Converts each PDF invoice into Markdown text for downstream LLM extraction

Why used:    N8N cannot natively parse PDF content; LlamaIndex provides accurate OCR

         and structured Markdown output, especially for complex invoice layouts

Resource:    Upload endpoint:  POST [https://api.cloud.llamaindex.ai/api/parsing/upload](https://api.cloud.llamaindex.ai/api/parsing/upload)

         Status endpoint:  GET  https://api.cloud.llamaindex.ai/api/parsing/job/{id}

         Result endpoint:  GET  https://api.cloud.llamaindex.ai/api/parsing/job/{id}/result/markdown

         API Key (hardcoded in HTTP nodes): \[LLAMAINDEX\_API\_KEY — stored in N8N credential store\]

Credential:  Hardcoded Bearer token in HTTP Request nodes — not an N8N credential object

Platform:    Groq API (LLM — llama-3.3-70b-versatile)

Role:        Extracts structured invoice fields from the Markdown text returned by LlamaIndex

Why used:    Fast, cost-effective LLM for structured JSON extraction from invoice text

Resource:    Model: llama-3.3-70b-versatile

Credential:  Groq account 3 (groqApi)

Platform:    TCH-SYSLOG-Append Execution Log-WF01

Role:        Shared execution logger — logs each successful run to Sheet 6

Why used:    Team standard — all workflows must log via TCH-SYSLOG

Resource:    Workflow ID: \[TCH-SYSLOG workflow ID\]

Credential:  N/A (internal sub-workflow call)

---

## Section 3 — External Scripts and Code

None — this project uses N8N nodes only. No Google Apps Script, Python, or shell scripts.

---

## Section 4 — Workflow Map

PST-HMSCLAIM   →  Processes all PDF invoices in the source Drive folder, parses each

               via LlamaIndex \\+ Groq LLM, normalises the data, and appends rows

               to the Hermes Claims sheet. Single workflow — no sub-steps.

TCH-SYSLOG-Append Execution Log-WF01   →  \[SHARED\] Logs every execution — not project-specific

---

## Section 5 — Data Flow

\[Manual Trigger\]

→ Developer clicks Run in N8N to start the batch

\[Google Sheets — Claims tab\]

→ Range A2:M1000 is cleared before any writes

→ Ensures no duplicate rows from previous runs

\[Google Drive — Source Folder ID: \[DRIVE\_FOLDER\_ID — confirm with Vithusali\]\]

→ All files in the folder are listed via Drive API

→ Files are sorted by the numeric part of their filename (ascending order)

→ Each file is downloaded as binary data one at a time inside a loop

\[LlamaIndex Cloud — PDF Parsing\]

→ Binary PDF is uploaded to LlamaIndex via POST /api/parsing/upload

  (premium\\\_mode=true for higher accuracy)

→ Job ID is returned; status is polled via GET /api/parsing/job/{id}

→ If status \\= PENDING, workflow waits 10 seconds and polls again

→ If status \\= SUCCESS, Markdown text is fetched via GET /api/parsing/job/{id}/result/markdown

\[Groq LLM — llama-3.3-70b-versatile\]

→ Markdown invoice text is sent to the AI Agent node

→ System prompt instructs extraction of 22 structured fields per product line

→ Output must be a JSON array (one object per product line)

→ Special rules applied in prompt: order\\\_number extraction from Repla-XX-XXXXX-XXXXX format,

  tracking number extraction from parentheses in shipment text,

  customer\\\_address concatenated as a single comma-separated string,

  brand\\\_name hardcoded to "Ledsone",

  subtotal\\\_items constrained between 40 and 60

\[Code Node — Clean & Normalise\]

→ Strips JSON markdown fences from LLM output

→ Ensures brand\\\_name \\= "Ledsone" for every row

→ Normalises customer\\\_address (array or newline → comma-separated string)

→ Extracts UK postcode from customer\\\_address via regex

→ Subtracts 5 days from invoice\\\_date (date adjustment rule — see Section 6\\)

→ Extracts tracking number from shipment\\\_details parentheses if not already present

→ Detects currency from field values (£→GBP, $→USD, €→EUR); defaults to GBP

→ Converts all numeric fields to floats

→ Clamps subtotal\\\_items between 40 and 60

→ Wraps any value containing commas, quotes, or newlines in CSV-safe quoting

\[Google Sheets — Claims tab\]

→ One row appended per product line with these columns:

  CLIENT REFERNCE NO, BRAND NAME, HERMES BARCODE, ORDER NO,

  CUSTOMER NAME, POSTCODE, PARCEL VALUE, PRODUCT DESCRIPTION

\[TCH-SYSLOG\]

→ Node E01 builds success payload; Node E02 calls TCH-SYSLOG sub-workflow

→ Execution logged to Sheet 6 with trigger\\\_type \\= 'Form-Trigger', run\\\_by \\= 'Jennani'

---

## Section 6 — BLOS Keys Used

None — this project does not use BLOS. All thresholds and business rules are hardcoded in the AI Agent system prompt and the Code node. If the subtotal\_items clamp range (40–60) or the invoice\_date offset (−5 days) need to change, they must be edited directly in the relevant node.

**Pending:** Consider requesting BLOS keys for `pst_hmsclaim_subtotal_floor`, `pst_hmsclaim_subtotal_ceiling`, and `pst_hmsclaim_date_offset_days` if these values change frequently. Contact Vithusali to create them.

---

## Section 7 — Reference Data Scope

N/A — this project is not channel or account scoped via Account\_Mappings.

All scope is determined by which PDF files are present in the Drive source folder.

---

## Section 8 — Data Mapping

### 8.1 LlamaIndex Cloud API

API:         LlamaIndex Cloud Parsing API

Upload:      POST [https://api.cloud.llamaindex.ai/api/parsing/upload](https://api.cloud.llamaindex.ai/api/parsing/upload)

Status:      GET  [https://api.cloud.llamaindex.ai/api/parsing/job/{id}](https://api.cloud.llamaindex.ai/api/parsing/job/{id})

Result:      GET  [https://api.cloud.llamaindex.ai/api/parsing/job/{id}/result/markdown](https://api.cloud.llamaindex.ai/api/parsing/job/{id}/result/markdown)

Auth:        Bearer token hardcoded in HTTP nodes (see Section 2\)

Pagination:  None — one job per file

Rate limit:  Not documented; premium\_mode=true used for accuracy

Used by:     Node6-HTTP Request(LlamaIndex – Upload File)

         Node7-HTTP Request(LlamaIndex – Check Job Status)

         Node10-HTTP Request(LlamaIndex – Fetch Markdown)

**Upload request — fields sent:**

| Parameter | Type | Required | Description |
| :---- | :---- | :---- | :---- |
| file | binary | Yes | The PDF invoice binary downloaded from Google Drive |
| premium\_mode | string | Yes | Set to `"true"` for higher OCR accuracy on complex invoice layouts |

**Status response — fields used:**

| Field | Type | Description |
| :---- | :---- | :---- |
| id | string | Job ID used to poll status and fetch result |
| status | string | `SUCCESS` or `PENDING` — drives the Switch routing node |

**Markdown result response — fields used:**

| Field | Type | Description |
| :---- | :---- | :---- |
| markdown | string | Full invoice text as Markdown; passed to AI Agent node as `$json.markdown` |

---

### 8.2 Groq LLM — Extracted Invoice Fields

The AI Agent node sends the Markdown invoice text to llama-3.3-70b-versatile with a structured extraction prompt. The model returns a JSON array. Each object in the array represents one product line with the following fields:

| Field | Type | Source / Rule |
| :---- | :---- | :---- |
| invoice\_number | string | Extracted from invoice header |
| invoice\_date | string | Extracted as-is from invoice; Code node subtracts 5 days and formats as YYYY-MM-DD |
| customer\_name | string | Full name of recipient |
| customer\_address | string | All address lines concatenated with commas |
| country | string | Country extracted from address |
| order\_number | string | If format is `Repla-XX-XXXXX-XXXXX`, only the last segment is used (e.g. `68179`) |
| shipment\_details | string | Carrier and service info; parenthesised tracking code removed from here |
| tracking\_number | string | Extracted from parentheses in shipment\_details (e.g. `T062XA0024814417`) |
| product\_name | string | Description of the product on this line |
| brand\_name | string | Always `"Ledsone"` — hardcoded regardless of invoice content |
| postal\_code | string | Extracted by Code node via UK postcode regex from customer\_address |
| quantity | float | Number of units on this line |
| unit\_price | float | Price per unit (currency symbols stripped) |
| total\_price | float | Line total (currency symbols stripped) |
| subtotal\_items | float | Clamped to range 40–60 by Code node |
| discount | float | Discount applied (0 if none) |
| subtotal | float | Invoice subtotal before VAT |
| vat | float | VAT amount |
| shipping\_cost | float | Shipping charge |
| total | float | Invoice grand total |
| paid | float | Amount paid |
| currency | string | Detected from field values: `£`→`GBP`, `$`→`USD`, `€`→`EUR`. Defaults to `GBP` |

---

### 8.3 Google Sheet — new \- Hermes Claims Forms (Output)

Sheet Name:    new \- Hermes Claims Forms

Sheet ID:      \[SHEET\_ID — new \- Hermes Claims Forms\]

Tab Name:      Claims

Tab gid:       \[tab gid\]

Operation:     CLEAR range A2:M1000 first, then APPEND one row per product line

Used by:       Node6-GSheets – Clear Claims Sheet

           Node7-GSheets – Append Claims Row

Maintained by: Auto-written by N8N — Jennani owns the sheet

Credential:    Gishor (googleSheetsOAuth2Api)

**Columns written by this workflow (mapped from extracted invoice data):**

| Sheet Column Header | Source Field | Description |
| :---- | :---- | :---- |
| CLIENT REFERNCE NO | order\_number | Order reference number from invoice |
| BRAND NAME | brand\_name | Always "Ledsone" |
| HERMES BARCODE | tracking\_number | Parcel tracking barcode |
| ORDER NO | order\_number | Same as CLIENT REFERNCE NO |
| CUSTOMER NAME | customer\_name | Recipient full name |
| POSTCODE | postal\_code | UK postcode extracted from address |
| PARCEL VALUE | subtotal | Invoice subtotal (pre-VAT) |
| PRODUCT DESCRIPTION | product\_name | Description of the product |

**Columns present in sheet but NOT written by this workflow (left blank):**

| Sheet Column Header | Notes |
| :---- | :---- |
| DOR LETTER DATE | Not populated — filled manually by postage team |
| CLIENT OR CUSTOMER COMMENTS | Not populated — filled manually |
| PRODUCT CATEGORY | Not populated — filled manually |
| CARRIAGE | Not populated — filled manually |
| LOST OR DAMAGED | Not populated — filled manually |

---

### 8.4 Google Drive — Source Folder

Platform:      Google Drive

Folder ID:     \[DRIVE\_FOLDER\_ID — confirm with Vithusali\]

Operation:     LIST all files, then DOWNLOAD each file as binary

Used by:       Node2-GDrive – List Folder Files

           Download file

Maintained by: Postage team uploads invoices here before triggering the workflow

Credential:    GISHOR (googleDriveOAuth2Api)

**File listing response fields used:**

| Field | Type | Description |
| :---- | :---- | :---- |
| id | string | Drive file ID — used to download the file |
| name | string | Filename — numeric part extracted for sort ordering |

---

## Section 9 — Replication Guide

To replicate this project for a different carrier (e.g. DPD Claims or Royal Mail Claims):

CHANGE:

Source Drive folder:    Folder ID \[DRIVE\_FOLDER\_ID — confirm with Vithusali\]  →  New carrier folder ID

Output Sheet ID:        \[SHEET\_ID — new \- Hermes Claims Forms\]  →  Carrier-specific claims sheet

Output Tab name:        "Claims"  →  Carrier-specific tab name

Column mapping:         Node7-GSheets – Append Claims Row mapping  →  Carrier's required column headers

AI prompt:              order\_number extraction rule (Repla-XX format) may differ per carrier

Workflow name:          PST-HMSCLAIM  →  PST-\[CARRIERNAME\]CLAIM

run\_by in Node E01:     'Jennani'  →  Whoever triggers the new workflow

trigger\_type in E01:    'Form-Trigger'  →  Confirm correct value for the new workflow

KEEP THE SAME:

LlamaIndex parsing flow:    Same PDF parsing logic applies to any carrier invoice

Groq LLM and prompt:        Core extraction logic is carrier-agnostic (adjust order\_number rule only)

Code node normalisation:    Postcode extraction, currency detection, CSV safety wrapping all reusable

File sort logic:            Numeric sort on filename — reusable

TCH-SYSLOG logging:         Same shared sub-workflow — no changes

LlamaIndex API key:

Currently hardcoded in all three HTTP nodes. If the key rotates, update all three nodes.

Consider storing in an N8N credential or config node for future replications.

Apps Script changes:   None — no Apps Script

Python changes:        None — no Python

Estimated developer time:  2–3 hours (column mapping and prompt tuning are the main variables)

---

## Section 10 — Known Constraints and Dependencies

\- LlamaIndex API key is hardcoded in three HTTP Request nodes (Upload, Status, Result).

If the API key rotates or expires, all three nodes must be updated.

Nodes affected: Node6-HTTP Request(LlamaIndex – Upload File),

              Node7-HTTP Request(LlamaIndex – Check Job Status),

              Node10-HTTP Request(LlamaIndex – Fetch Markdown).

\- invoice\_date adjustment: the Code node subtracts 5 days from every invoice date.

This is a business rule from the postage team. If this rule changes, update the

date offset in Node5-Code(Format – Clean & Normalise Invoice Data).

No BLOS key exists for this — it is a hardcoded value.

\- subtotal\_items clamp: the Code node enforces a range of 40–60 for subtotal\_items.

This is separate from the AI prompt instruction. Both exist to double-enforce the rule.

If the range changes, update the Code node (not just the AI prompt).

\- LlamaIndex job polling: the workflow polls job status every 10 seconds via a Wait node.

For very large or complex PDFs, more iterations may be needed. No timeout exists —

if a job never completes, the workflow loop will poll indefinitely.

Monitor execution time for unusually large files.

\- Google Drive folder scope: the workflow processes ALL files currently in folder

ID \[DRIVE\_FOLDER\_ID — confirm with Vithusali\] on every run. If old files are not removed

between runs, they will be re-processed and duplicate rows will appear in the sheet.

The postage team must clear or archive processed files before triggering a new run.

\- Claims sheet cleared on every run: range A2:M1000 is cleared at the start of each run

before any rows are written. Any manually entered data in that range will be lost.

Columns that the team fills manually (DOR LETTER DATE, COMMENTS etc.) must be

completed AFTER the workflow has run, not before.

\- TCH-SYSLOG dependency: if TCH-SYSLOG-Append Execution Log-WF01 (ID: \[TCH-SYSLOG workflow ID\])

is archived or deactivated, Node E02 will fail.

Do not archive TCH-SYSLOG without checking all dependent workflows first.

\- Groq credential: uses "Groq account 3" .

If the team's Groq account changes, update the Groq Chat Model node.

\- Single endpoint only: execution logger (E01/E02) is connected after

Node7-GSheets – Append Claims Row. If a second output path is added in future,

E01/E02 must also be added to that path.

---

## Section 11 — Ownership

Project Owner:        Jennani

Lead Developer:       Gishor

Maintainer:           Gishor

Business Requester:   Jennani  
Operations Contact:   Vithusali

---

*LEDs One · Automation Team · Project Context Document & Developer Guide v1.0 · May 2026* *Follows PCD Developer Guide v2.1 format · Questions: Slack \#automation-team*  
