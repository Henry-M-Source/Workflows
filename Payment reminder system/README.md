# Payment Reminder System

Automated payment reminder pipeline that runs daily, checks invoice due dates, and sends escalating email reminders — updating the record status at each stage to prevent duplicate notifications.

---

## Overview

This workflow runs on a schedule and queries Airtable for unpaid invoices. It evaluates each record's due date and routes it through an escalating reminder sequence: 7 days out, 3 days out, due today, and overdue. Each stage sends a tailored email to the client and updates the invoice status in Airtable, ensuring every record is only contacted at the correct stage on each run.

---

## Workflow

```
Schedule Trigger → Search Records → If (7 day check)
                                         │
                              ┌──────────┴──────────┐
                           [true]                [false]
                              │                     │
                        Send an Email        3 Day Check (If)
                        (7-day reminder)          │
                              │           ┌────────┼────────┐
                       7 days status   [true]  [middle]  [false]
                         switch           │       │          │
                      (update record)     │       │          │
                                    Send an   Send an     If1
                                    Email1    Email2        │
                                  (3-day)   (due today) ┌──┴──┐
                                     │          │    [true] [false]
                               3 days status  Today        │
                               switch         status   Send an
                             (update record)  switch    Email3
                                           (update)   (overdue)
                                                          │
                                                     Overdue Switch
                                                     (update record)
```

---

## Nodes

| Node | Type | Purpose |
|------|------|---------|
| Schedule Trigger | Trigger | Runs the workflow on a daily schedule |
| Search Records | Action | Queries Airtable for unpaid invoice records |
| If | Logic | Checks whether the invoice due date is 7 days away |
| Send an Email | Notification | Sends the 7-day payment reminder to the client |
| 7 days status switch | Action | Updates the Airtable record status to "7-day reminder sent" |
| 3 Day Check (If) | Logic | Checks whether the due date is 3 days away or today |
| Send an Email1 | Notification | Sends the 3-day payment reminder |
| 3 days status switch | Action | Updates the record status to "3-day reminder sent" |
| Send an Email2 | Notification | Sends the due-today payment reminder |
| Today status switch | Action | Updates the record status to "due today reminder sent" |
| If1 | Logic | Checks whether the invoice is past its due date |
| Send an Email3 | Notification | Sends the overdue payment notice |
| Overdue Switch | Action | Updates the record status to "overdue" |

---

## How It Works

1. **Daily run** — the workflow triggers on a schedule (e.g. every morning) and searches Airtable for all unpaid invoice records
2. **7-day check** — if the due date is 7 days away, a first reminder email is sent and the record is updated
3. **3-day check** — if the due date is 3 days away, a follow-up reminder is sent and the record is updated
4. **Due today** — if the invoice is due today, an urgent reminder is sent and the record is updated
5. **Overdue check** — if the due date has passed, an overdue notice is sent and the record is marked overdue
6. **Status tracking** — each stage updates the Airtable record status so the invoice is only contacted at the correct stage on the next daily run — no duplicate emails

---

## Reminder Stages

| Stage | Trigger | Email Tone | Status Set |
|-------|---------|------------|------------|
| 7-day reminder | 7 days before due date | Friendly heads-up | 7-day reminder sent |
| 3-day reminder | 3 days before due date | Gentle follow-up | 3-day reminder sent |
| Due today | Day of due date | Urgent reminder | Due today reminder sent |
| Overdue | Past due date | Formal overdue notice | Overdue |

---

## Requirements

- **n8n** self-hosted or cloud instance
- **Airtable** base with an invoices/payments table containing at minimum:
  - Client email address
  - Invoice due date
  - Payment status field
- **SMTP / email credentials** configured in n8n for all Send an Email nodes

---

## Setup

1. Import `Payment Reminder System.json` into your n8n instance
2. Configure credentials:
   - Airtable — connect your base and select the invoices table in the Search Records node
   - Email (SMTP) — used across all four reminder email nodes
3. Update each If node's date condition to match your Airtable due date field name
4. Customize the email content in each Send an Email node to match your tone and branding
5. Set the schedule in the Schedule Trigger node (daily morning run recommended)
6. Activate the workflow

---

## Notes

- The status field in Airtable is critical — it prevents the same invoice from receiving duplicate reminders on subsequent daily runs. Ensure the field is being correctly read and written by each status switch node
- If an invoice skips a stage (e.g. added when already 1 day from due), it will be picked up at the correct stage on the next run
- Email content for the overdue stage should be reviewed with your team — tone and escalation policy may vary by client
