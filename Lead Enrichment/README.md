# Lead Enrichment Flow

Automated lead qualification pipeline that enriches, scores, and routes incoming form submissions — sending hot leads to Slack instantly and logging warm leads to a CRM for follow-up.

---

## Overview

When a lead submits a form, this workflow checks whether a website and LinkedIn profile were provided. If so, it enriches the lead using TChecker's publicly available data before scoring. Leads without those fields are scored immediately on raw form data alone. Every lead is then analyzed by AI, scored, and routed — hot leads hit a Slack channel for immediate sales action, warm leads are created as a CRM record for nurture follow-up.

---

## Workflow

```
On form submission → Web Enrichment (If)
                          │
               ┌──────────┴──────────┐
            [true]                [false]
         Has website               No website
         + LinkedIn                or LinkedIn
              │                       │
        TChecker API                  │
              │                       │
       Merge Enrichment               │
              │                       │
              └──────────┬────────────┘
                         │
                     Lead Score
                         │
               Message AI Analysis
                         │
                        If
                         │
              ┌──────────┴──────────┐
           [true]               [false]
          Hot Leads            Warm Leads
        (Slack message)      (CRM record)
```

---

## Nodes

| Node | Type | Purpose |
|------|------|---------|
| On form submission | Trigger | Captures incoming lead data from a form |
| Web Enrichment | Logic | Checks whether the lead provided a website and LinkedIn URL |
| TChecker API | Enrichment | Pulls publicly available data from the lead's website and LinkedIn |
| Merge Enrichment | Transform | Combines TChecker results with the original form data |
| Lead Score | Transform | Calculates a score based on available lead data |
| Message AI Analysis | AI | Generates a qualification summary of the lead |
| If | Logic | Routes leads by score — hot vs warm |
| Hot Leads | Notification | Posts high-score leads to a Slack channel for immediate follow-up |
| Warm Leads | Action | Creates a CRM record for lower-score leads to enter a nurture sequence |

---

## How It Works

1. **Form submitted** — a lead fills out a form, triggering the workflow
2. **Website + LinkedIn check** — the Web Enrichment node checks whether both fields were provided
   - If **yes** → the data is forwarded to TChecker, which pulls publicly available information from the website and LinkedIn profile. Results are merged with the original submission before scoring
   - If **no** → the raw form data is forwarded directly to the Lead Score node
3. **Lead scoring** — every lead is scored regardless of enrichment level; enriched leads score with more signal, unenriched leads score on form data alone
4. **AI analysis** — the Message AI Analysis node generates a qualification summary of the lead
5. **Routing**
   - **Hot leads** (high score) → posted to Slack for immediate sales action
   - **Warm leads** (lower score) → logged as a CRM record for nurture follow-up

---

## Requirements

- **n8n** self-hosted or cloud instance
- **TChecker API** key and account
- **Slack** workspace with a configured incoming webhook or bot token
- **CRM** (e.g. Airtable, HubSpot) credentials configured in n8n
- A form tool (e.g. Typeform, n8n Form, Tally) pointing to the workflow trigger

---

## Setup

1. Import `Lead Enrichment Flow.json` into your n8n instance
2. Configure credentials:
   - TChecker API — add your API key
   - Slack — connect your workspace and set the target channel in the Hot Leads node
   - CRM — connect your account and map fields in the Warm Leads node
3. Update the Lead Score node logic to reflect your own scoring criteria
4. Activate the workflow and connect your form to the trigger URL

---

## Notes

- The score threshold that separates hot from warm leads is set in the final If node — adjust it to match your sales team's qualification criteria
- Leads without a website or LinkedIn are still fully processed; they simply enter the pipeline with less enrichment data
- The AI analysis prompt in the Message AI Analysis node can be customized to focus on specific qualification criteria relevant to your business
