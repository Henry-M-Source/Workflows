# Document Intake

Automated PDF intake pipeline that receives documents via webhook, validates format, extracts content with AI, and logs submissions to Airtable — notifying the sender at every step.

---

## Overview

This workflow listens for incoming document submissions through a webhook. It validates that the file is a PDF, routes invalid submissions back to the sender, and processes valid ones through an AI-powered extraction and summarization pipeline before creating a record in Airtable and confirming receipt to the client.

---

## Workflow

```
Webhook → Code (JS) → HTTP Request → If (PDF check)
                                         │
                              ┌──────────┴──────────┐
                           [true]                [false]
                              │                     │
                     Extract from File         Send an Email
                     (Extract from PDF)        (resubmit notice)
                              │
                          Edit Fields
                          (manual mapping)
                              │
                         Flowise API
                         (AI summary)
                              │
                        Create a Record
                        (Airtable)
                              │
                        Send an Email
                        (confirmation)
```

---

## Nodes

| Node | Type | Purpose |
|------|------|---------|
| Webhook | Trigger | Listens for incoming POST requests with an attached document |
| Code in JavaScript | Transform | Parses and prepares the submission payload |
| HTTP Request | Action | Fetches or forwards the document for validation |
| If | Logic | Checks whether the submitted file is a PDF |
| Extract from File | Action | Extracts raw text content from the PDF |
| Edit Fields | Transform | Maps sender details and extracted content to structured fields |
| Flowise API | AI | Generates an AI summary of the extracted document content |
| Create a record | Action | Logs the submission, sender info, and summary to Airtable |
| Send an Email | Notification | Confirms receipt to the client after a successful submission |
| Send an Email | Notification | Notifies the sender to resubmit if the format is not a PDF |

---

## How It Works

1. **Submission received** — a client submits a document via the webhook endpoint (POST request)
2. **Format validation** — the workflow checks whether the file is a PDF
   - If **not a PDF**, the sender receives an email asking them to resubmit in the correct format
   - If **valid**, the workflow continues
3. **Text extraction** — the PDF content is extracted using the Extract from File node
4. **AI summarization** — the extracted text is sent to Flowise, which returns an AI-generated summary
5. **Record creation** — sender details and the AI summary are populated as a new record in Airtable
6. **Confirmation sent** — the client receives an email confirming their submission was received and processed

---

## Requirements

- **n8n** self-hosted or cloud instance
- **Flowise** instance with a configured AI flow (update the endpoint URL in the Flowise API node)
- **Airtable** account with a base and table set up to receive submissions
- **SMTP / email credentials** configured in n8n for the Send an Email nodes

---

## Setup

1. Import `Document Intake.json` into your n8n instance
2. Configure credentials:
   - Email (SMTP) — used for both notification and confirmation emails
   - Airtable — connect your base and select the target table
   - Flowise — update the API endpoint URL in the Flowise API node
3. Activate the workflow and copy the webhook URL
4. Point your intake form or client-facing endpoint to the webhook URL

---

## Notes

- Ensure the webhook accepts `multipart/form-data` if clients are submitting via a file upload form
- The Edit Fields node may need to be updated to match your Airtable column names
- Flowise endpoint URL is hardcoded in the node — update it after import
