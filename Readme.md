# 🤖 AI HR Screening Agent

> An intelligent, fully automated HR screening pipeline built on **n8n** — powered by **OpenAI GPT-4**, **Gmail**, **Google Sheets**, and **Google Calendar**. Screens CVs, routes candidates, handles follow-ups, and ranks shortlisted applicants — all without human intervention.

![n8n](https://img.shields.io/badge/Built%20with-n8n-EA4B71?logo=n8n&logoColor=white)
![OpenAI](https://img.shields.io/badge/AI-OpenAI%20GPT--4-412991?logo=openai&logoColor=white)
![Google Sheets](https://img.shields.io/badge/Database-Google%20Sheets-34A853?logo=googlesheets&logoColor=white)
![Gmail](https://img.shields.io/badge/Email-Gmail-EA4335?logo=gmail&logoColor=white)
![License](https://img.shields.io/badge/License-All%20Rights%20Reserved-red)

---

## 📌 Overview

Developed as a **4th Semester Artificial Intelligence Lab project**, this system automates the entire candidate screening pipeline — from CV submission to interview booking — using a multi-workflow n8n agent system backed by GPT-4. Traditional HR screening is slow, inconsistent, and expensive; this project eliminates those bottlenecks end-to-end.

The system handles:
- Receiving and parsing CVs via webhook
- Validating completeness and routing by job role
- Emailing candidates for missing information
- Shortlisting, rejecting, or flagging for manual review
- Listening for candidate replies and re-evaluating updated CVs
- Ranking all shortlisted candidates every 6 hours via a scheduled engine

---

## 🏗️ System Architecture

The pipeline is split into **3 interconnected workflows**:

```
[CV Upload Webhook]
       │
       ▼
┌─────────────────────────────────┐
│  WORKFLOW 1 — CV Intake &       │
│  Role Router                    │
│                                 │
│  • Extracts & validates CV text │
│  • GPT-4 completeness check     │
│  • Decision: Shortlist /        │
│    Reject / Review / Follow-Up  │
│  • Books interview via Calendar │
│  • Sends Gmail notifications    │
│  • Logs all results to Sheets   │
└────────────┬────────────────────┘
             │ triggers
             ▼
┌─────────────────────────────────┐
│  WORKFLOW 3 — AI Ranking Engine │
│                                 │
│  • Runs every 6 hours           │
│  • Fetches all shortlisted CVs  │
│  • GPT-4 comparative ranking    │
│  • Writes fresh rankings to     │
│    Google Sheets                │
└─────────────────────────────────┘

[Gmail Reply Trigger] ──► WORKFLOW 2 — Follow-Up Listener
                           • Watches for candidate email replies
                           • Re-evaluates with updated CV
                           • Updates Sheets: Pending → Resolved
                           • Re-routes through decision pipeline
```

---

## ⚙️ Workflow Breakdown

### Workflow 1 — CV Intake & Role Router
**Trigger:** Webhook (CV file upload)

| Node | Purpose |
|------|---------|
| Webhook — CV Upload | Receives incoming CV submissions |
| Validate Input & Extract Metadata | Sanitises input, extracts file metadata |
| Extract CV Text | Parses PDF/DOCX into plain text |
| CV Completeness Analyzer | Checks for required fields (education, experience, skills) |
| Message a model (GPT-4) | AI screening against job role requirements |
| Decision Router | Routes to: Shortlist / Reject / Review / Follow-Up |
| Calendar: Book Interview | Auto-books slot for shortlisted candidates |
| Email: Interview Invitation | Sends personalised invite via Gmail |
| Email: Polite Rejection | Sends rejection email |
| Email: Request Missing Info | Requests incomplete data from candidate |
| Sheets: Log * | Logs all outcomes to respective Google Sheets tabs |
| Trigger Ranking Engine | Calls Workflow 3 after shortlisting |

---

### Workflow 2 — Follow-Up Listener & Re-Evaluator
**Trigger:** Gmail Watch (incoming candidate reply)

| Node | Purpose |
|------|---------|
| Gmail: Watch Candidate Replies | Listens for replies to screening emails |
| Extract Reply & Session Context | Parses reply content and session ID |
| Sheets: Find Pending Application | Looks up original candidate record |
| Merge Reply with Original Context | Combines original CV + new reply |
| Has Updated CV? | Checks if candidate attached a new CV |
| Extract Updated CV Text | Parses new CV if attached |
| Message a model (GPT-4) | Re-evaluates candidate with updated info |
| Parse Re-Evaluation Result | Structures AI output |
| Sheets: Update Pending → Resolved | Updates application status |
| Send to Decision Pipeline | Re-routes through Workflow 1 logic |

---

### Workflow 3 — AI Candidate Ranking Engine
**Trigger:** Schedule (every 6 hours) + Sub-workflow call

| Node | Purpose |
|------|---------|
| Schedule: Run Every 6 Hours | Periodic autonomous trigger |
| Sheets: Fetch All Shortlisted | Pulls all shortlisted candidates |
| Group & Prepare Candidates | Batches candidates per job role |
| Message a model (GPT-4) | Comparative ranking with scoring rationale |
| Parse & Flatten Rankings | Structures ranking output |
| Sheets: Clear Old Rankings | Removes stale rankings |
| Sheets: Write Fresh Rankings | Writes updated ranked list |

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| Workflow Automation | n8n (self-hosted or cloud) |
| AI / LLM | OpenAI GPT-4 |
| Email | Gmail (via Google OAuth) |
| Database | Google Sheets |
| Scheduling | Google Calendar + n8n Scheduler |
| CV Parsing | n8n Extract from File node |
| Trigger | n8n Webhook |

---

## 🚀 Getting Started

### Prerequisites

- [n8n](https://n8n.io) instance (self-hosted or n8n Cloud)
- OpenAI API key (GPT-4 access)
- Google account with Gmail, Sheets, and Calendar APIs enabled
- Google OAuth credentials configured in n8n

### Step 1 — Clone this repository

```bash
git clone https://github.com/OwaisTanoli71/ai-hr-screening-agent.git
```

### Step 2 — Import workflows into n8n

1. Open your n8n instance
2. Go to **Workflows** → **Import from file**
3. Import in this order:
   - `WORKFLOW 3 — AI Candidate Ranking Engine.json` *(import first — others call it)*
   - `WORKFLOW 2 — Multi-Step Agent_ Follow-Up Listener & Re-Evaluator.json`
   - `WORKFLOW 1 — CV Intake & Role Router.json`

### Step 3 — Configure credentials

In n8n, set up the following credentials:

| Credential | Used In |
|-----------|---------|
| OpenAI API Key | All 3 workflows |
| Google OAuth (Gmail) | Workflow 1, 2 |
| Google OAuth (Sheets) | Workflow 1, 2, 3 |
| Google OAuth (Calendar) | Workflow 1 |

### Step 4 — Set up Google Sheets

Create a Google Sheet with the following tabs:

| Tab Name | Purpose |
|----------|---------|
| `Shortlisted` | Candidates approved for interview |
| `Rejected` | Declined applications |
| `For Review` | Flagged for manual HR review |
| `Pending Follow-Up` | Awaiting candidate response |
| `Rankings` | AI-generated ranked candidate list |

### Step 5 — Activate workflows

1. Activate **Workflow 3** first
2. Activate **Workflow 2**
3. Activate **Workflow 1** last
4. Copy the webhook URL from Workflow 1 — this is your CV submission endpoint

---

## 📁 Repository Structure

```
ai-hr-screening-agent/
├── WORKFLOW 1 — CV Intake & Role Router.json
├── WORKFLOW 2 — Multi-Step Agent_ Follow-Up Listener & Re-Evaluator.json
├── WORKFLOW 3 — AI Candidate Ranking Engine.json
└── README.md
```

---

## 🔒 Security Notes

- Never commit your `.env` file or hardcoded API keys
- All credentials are stored inside n8n's encrypted credential store
- Webhook URLs should be kept private — treat them as secrets
- Google OAuth tokens are managed by n8n and never stored in this repo

---

## 👤 Author

**Muhammad Owais Arshad**
- GitHub: [@OwaisTanoli71](https://github.com/OwaisTanoli71)

---

## 📄 License

© 2026 Muhammad Owais Arshad. All rights reserved.

This project was independently conceived and developed as an academic semester project. No part of this project — including workflows, logic, or documentation — may be copied, reused, or redistributed without explicit written permission from the author.

---

## ⭐ Acknowledgements

- [n8n](https://n8n.io) — workflow automation platform
- [OpenAI](https://openai.com) — GPT-4 language model
