# 🏥 Clinical Research Training Portal

An AI-powered onboarding and training compliance tool for clinical research staff at GWU/MFA, built on **React + Vite** and backed by an **n8n** automation workflow. Grounded in **SOPs**.

---

## Overview

The portal has two primary interfaces:

**Staff Portal** — A guided intake form that collects role, hire date, system access needs, and prior training history. On completion, it calls an n8n webhook which invokes a Claude-powered agent to generate a personalized, role-specific training plan in real time. Staff can then ask follow-up questions and report training completions in a chat interface.

**Manager Dashboard** — A live compliance view for PIs and regulatory staff. Pulls all staff records from n8n/Google Sheets, surfaces overdue 2-week deadlines, flags third-party contractors (SOP 21), and displays session status per team member.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 18, Vite |
| Styling | Inline styles + Tailwind utility classes |
| AI Agent | Claude (Anthropic) via n8n workflow |
| Automation | n8n (self-hosted or cloud) |
| Data store | Google Sheets (via n8n) |
| Fonts | Playfair Display, DM Mono (Google Fonts) |

---

## Project Structure

```
/
├── src/
│   ├── CRAPortal.jsx       # Main application (all views in one file)
│   └── main.jsx            # React entry point
├── index.html
├── vite.config.js
└── package.json
```

---

## Getting Started

### Prerequisites

- Node.js 18+
- An n8n instance (cloud or self-hosted) with the CRA workflow imported
- A Google Sheet configured as the data store (see n8n workflow setup)

### Installation

```bash
git clone <repo-url>
cd <repo-name>
npm install
npm run dev
```

### Environment / Webhook Configuration

The app communicates with two n8n webhooks. Update the constants at the top of `src/CRAPortal.jsx`:

```js
// Staff intake + AI plan generation
const N8N_CHAT_WEBHOOK = "https://<your-n8n-instance>/webhook/cra-intake";

// Manager dashboard data fetch
const N8N_DASHBOARD_WEBHOOK = "https://<your-n8n-instance>/webhook/<dashboard-webhook-id>";
```

No `.env` file is required — these are set directly in the source for now.

---

## n8n Workflow

The repo includes the workflow export file:

```
CRA_Training_Agent___Hybrid_v2.json
```

Import this into your n8n instance. The workflow handles:

- Receiving intake payloads and generating a personalized training plan via Claude
- Persisting staff records to Google Sheets
- Serving dashboard data to the Manager view

### Workflow Nodes (high level)

1. **Webhook** — receives `POST` from the React app
2. **Claude AI Agent** — generates the training plan using SOP 15 context
3. **Google Sheets** — reads/writes staff training records
4. **Respond to Webhook** — returns `{ training_plan, session_id, ... }` to the frontend

---

## n8n Response Format

The frontend handles two n8n response shapes:

```json
// Training plan (new hire intake)
{ "training_plan": "# Training Plan — Jane Smith\n..." }

// Q&A follow-up
{ "answer": "GCP certification is valid for 3 years..." }
```

Arrays are also unwrapped automatically: `[{ ... }]` → `{ ... }`.

---

## Key Features

- **Role-aware training plans** — CRC, Research Nurse, Data Coordinator, RA, PI, and Other each receive a tailored plan
- **2-week deadline tracking** — calculated from hire date, surfaced with urgency indicators in the dashboard
- **Third-party contractor flagging** — routes to SOP 21 compliance path automatically
- **Prior CITI recognition** — intake captures previous institutional training to avoid redundant coursework
- **Session persistence** — session ID stored in `sessionStorage` so chat context is maintained within a browser session
- **Live compliance dashboard** — PI view with overdue alerts and per-member status badges
- **Demo/mock mode** — if the n8n webhook is unreachable, the dashboard falls back to mock records so the UI is always testable

---

## SOP Compliance Notes

- This tool is designed to support (not replace) PI oversight responsibilities under **the sameple SOP**
- General training certificates must be retained in a central location with a Note to File in the regulatory binder
- Protocol-specific training must be filed in the respective protocol's regulatory binder
- **GCP certification must never be allowed to expire**
- External/vendor contractors are flagged for **SOP 21** handling

---

## Development Notes

- All views (`LandingPage`, `StaffPortal`, `ManagerDashboard`) live in `src/CRAPortal.jsx` as a single-file app
- `sessionStorage` is used for session ID and staff name — these reset on tab close
- `localStorage` is intentionally not used
- The `FormatMessage` component renders markdown output from the n8n agent (headings, bullets, horizontal rules, completion markers)

---

## Deployment

Build for production:

```bash
npm run build
```

Output goes to `/dist`. Deploy to any static host (Netlify, Vercel, S3, etc.). Make sure your n8n webhooks allow CORS from your deployment domain.

---

## Related Files

| File | Description |
|---|---|
| `CRA_Training_Agent___Hybrid_v2.json` | n8n workflow export |
| `My_workflow.json` | Alternative/backup workflow export |
| `sop_onboarding_and_training_clinical_research.pdf` | Source SOP document used to ground the AI agent |
| `Training_Tracker.xlsx` | Manual training tracker (pre-portal reference) |
| `Clinical_Research_Training_Agent.docx` | Agent design specification |
| `ClinicalResearch_training_assignment_BA_PM.docx` | Business analysis / PM requirements doc |

---

## License

Copyright © Mrunal Surve. All rights reserved.
This project and its contents are proprietary. No part of this codebase, documentation, or associated files may be copied, modified, distributed, sublicensed, or reused in any form — in whole or in part — without prior written permission from the owner.
To request permission for reuse, please contact the at pmmrunal@gmail.com.
