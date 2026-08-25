<div align="center">

# 📬 Inboxly

### Your inbox, triaged while you sleep.

![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/Backend-FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)
![Clerk](https://img.shields.io/badge/Auth-Clerk-6C47FF?style=for-the-badge)
![Agentic AI](https://img.shields.io/badge/Agentic-AI-8A2BE2?style=for-the-badge)
![Gemini](https://img.shields.io/badge/LLM-Gemini-4285F4?style=for-the-badge&logo=googlegemini&logoColor=white)
![IMAP](https://img.shields.io/badge/Protocol-IMAP%2FSMTP-orange?style=for-the-badge)
![Status](https://img.shields.io/badge/status-active-brightgreen?style=for-the-badge)
![License](https://img.shields.io/badge/license-MIT-lightgrey?style=for-the-badge)

</div>

---

## 📖 Table of Contents

- [🎥 Demo](#-demo)
- [✨ Overview](#-overview)
- [🌐 Product](#-product)
- [🧠 How It Works](#-how-it-works)
- [🏗️ Architecture](#️-architecture)
- [🛠️ Tech Stack](#️-tech-stack)
- [📁 Project Structure](#-project-structure)
- [⚙️ Setup](#️-setup)
- [🔐 Configuration](#-configuration)
- [🚀 Usage](#-usage)
- [🛡️ Reliability & Safety](#️-reliability--safety)
- [🗺️ Roadmap](#️-roadmap)
- [📜 License](#-license)

---

## 🎥 Demo

> 🎬 A short demo showing Inboxly triaging a live inbox, deciding between an autonomous reply and a human escalation, will be embedded here.

<div align="center">

https://github.com/user-attachments/assets/96e10f17-6e6a-496e-95bb-f2ff4d0f72b0

</div>

---

## ✨ Overview

**Inboxly** is a fully **agentic AI system** — no orchestration frameworks, no black-box agent libraries. Just an LLM given a clear goal, two tools, and the judgment to choose between them on its own.

It watches your inbox for career-related emails (recruiters, interviews, hiring managers) and for each one, autonomously decides:

- ✅ **"I know this — I'll reply myself."** → drafts and sends a threaded, professional reply grounded strictly in your resume and profile.
- 🙋 **"I don't know this — ask the human."** → sends you a push notification instead of guessing.

No hallucinated answers to recruiters. No missed opportunities. No manual inbox babysitting.

What started as a single notebook (`career_email_agent.ipynb`) is now packaged as **Inboxly**: a hosted product with a landing page, authenticated dashboard, and a FastAPI backend behind it.

---

## 🌐 Product

Inboxly is now a three-step product, not just a script:

| Step | What happens |
|---|---|
| **01 — Tell it who you are** | Upload your resume and a short summary in your own words. That's the only context the agent is allowed to use. |
| **02 — Connect your inbox** | IMAP/SMTP credentials, encrypted at rest. It only scans career-related mail. |
| **03 — It triages, you review** | Confident replies go out automatically. Anything uncertain lands in your dashboard and triggers a push notification. |

The dashboard shows a **live triage feed** for every email the agent has looked at, tagged with the action it took:

- `SEND_REPLY` — the agent was confident and replied on your behalf.
- `NOTIFY_ME` — the agent wasn't sure and pushed the decision to you.
- `SCANNED` — the email was reviewed and didn't need action (e.g. a routine application acknowledgment).

Users land on the marketing page, sign in (Clerk-authenticated), and land on their dashboard to review triage history and manage onboarding (resume/summary upload, inbox connection).
<img width="1672" height="894" alt="image" src="https://github.com/user-attachments/assets/115b81d2-0bf2-4f54-988d-3d7532184004" />

---

## 🧠 How It Works

1. **📥 Inbox Monitoring** — Connects via IMAP and scans recent emails, regardless of read/unread status.
2. **🔍 Keyword Triage** — Regex-filters for career-relevant keywords (`job`, `recruiter`, `interview`, `hiring`, `role`, etc.). Everything else is left untouched.
3. **🤔 Agentic Decision-Making** — The filtered email + your resume/summary are handed to Gemini (via an OpenAI-compatible API). The model must call **exactly one** of two tools (`tool_choice` enforced):
   - 🟢 `send_reply_tool` — writes and sends a first-person, professional reply; optionally attaches your resume.
   - 🔴 `notify_me_tool` — pushes a notification to your phone via Pushover instead of guessing.
4. **🧾 State Tracking** — Every handled email's Message-ID is logged, so restarts never trigger duplicate replies.
5. **📊 Surfacing to the user** — Every decision (reply sent, notification raised, or scan-only) is written back through the FastAPI backend so it shows up in the dashboard's live triage feed.

---

## 🏗️ Architecture

```
 ┌────────────┐     ┌───────────────┐     ┌──────────────────┐
 │   Inbox     │────▶│ Keyword Triage │────▶│  Agentic LLM Core │
 │ (IMAP poll) │     │ (regex filter) │     │  (Gemini + tools) │
 └────────────┘     └───────────────┘     └────────┬──────────┘
                                                     │
                                     ┌───────────────┴───────────────┐
                                     ▼                                ▼
                          ┌────────────────────┐          ┌────────────────────┐
                          │  send_reply_tool     │          │  notify_me_tool     │
                          │  → SMTP threaded     │          │  → Pushover alert   │
                          │    reply + resume     │          │    to your phone    │
                          └────────────────────┘          └────────────────────┘
                                     │                                │
                                     └───────────────┬────────────────┘
                                                      ▼
                                         ┌────────────────────────┐
                                         │   FastAPI Backend        │
                                         │ (auth, onboarding,       │
                                         │  mailbox, triage log)    │
                                         └────────────┬─────────────┘
                                                       ▼
                                         ┌────────────────────────┐
                                         │  Dashboard (Web UI)      │
                                         │  Live triage feed +      │
                                         │  onboarding flow          │
                                         └────────────────────────┘
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| 🐍 Agent Core | Python |
| 🌐 Backend | FastAPI (routers: onboarding, mailbox) |
| 🔑 Auth | Clerk (`clerk_backend_api`) |
| 📬 Email Protocols | IMAP (reading) + SMTP (threaded replies) |
| 🧠 LLM | Google Gemini via OpenAI-compatible API |
| 🔧 Agent Pattern | Manual function/tool calling (JSON Schema, `tool_choice="required"`), OpenAI SDK |
| 📄 Resume Parsing | `pypdf` |
| 📲 Notifications | Pushover API |
| 💾 State Management | JSON-based idempotent Message-ID tracking (backend-persisted for the hosted product) |
| 🖥️ Frontend | Landing page + authenticated dashboard |

---

## 📁 Project Structure

```
.
├── agent/
│   ├── career_email_agent.ipynb   # original agent core / prototyping notebook
│   ├── twin/
│   │   ├── resume.pdf               # your resume (LinkedIn PDF export works great)
│   │   └── summary.txt              # plain-text: background, status, skills, goals
│   └── handled_message_ids.json     # auto-generated — dedup state, don't commit
├── backend/
│   ├── routers/
│   │   ├── onboarding.py            # resume/summary upload, inbox connection
│   │   └── mailbox.py               # triage results, live feed data
│   └── main.py                      # FastAPI app entrypoint
├── frontend/
│   ├── landing/                     # marketing/landing page ("Your inbox, triaged while you sleep.")
│   └── dashboard/                   # authenticated dashboard + live triage feed
├── .env                             # credentials — never commit this
└── .gitignore
```

---

## ⚙️ Setup

### 1️⃣ Add your context files

Inside `agent/twin/` next to the notebook:
- `resume.pdf` — your resume or LinkedIn PDF export
- `summary.txt` — a specific, detailed summary of your background, current status, and what you're looking for. **The more specific, the smarter the agent's replies.**

(In the hosted product, this step is done through the **"Tell it who you are"** onboarding screen instead of dropping files locally.)

### 2️⃣ Install dependencies

```bash
pip install python-dotenv openai pypdf requests fastapi uvicorn clerk-backend-api
```

---

## 🔐 Configuration

Create a `.env` file in the project root:

```env
EMAIL_ADDRESS=your_email@gmail.com
EMAIL_APP_PASSWORD=your_app_password
EMAIL_SMTP_SERVER=smtp.gmail.com
EMAIL_IMAP_SERVER=imap.gmail.com
GOOGLE_API_KEY=your_gemini_api_key
PUSHOVER_USER=your_pushover_user_key
PUSHOVER_TOKEN=your_pushover_app_token
CLERK_SECRET_KEY=your_clerk_secret_key
```

> ⚠️ **Never commit `.env` to GitHub.** Make sure it's listed in `.gitignore`.

---

## 🚀 Usage

**Test safely first:**
```python
DRY_RUN = True   # nothing is actually sent — just simulated
```

**Run once:**
```python
process_inbox()
```

**Run continuously** (checks every 120 seconds by default):
```python
while True:
    process_inbox()
    time.sleep(POLL_INTERVAL_SECONDS)
```

Once you trust the output, flip to:
```python
DRY_RUN = False   # 🔴 live mode — real emails/pushes will be sent
```

**Run the hosted backend:**
```bash
uvicorn backend.main:app --reload
```

Then open the dashboard to connect an inbox and watch the live triage feed populate in real time.

---

## 🛡️ Reliability & Safety

- 🚫 **Never guesses** — if the answer isn't explicitly in your resume/summary, it escalates to you instead of fabricating a reply.
- 🔁 **Idempotent** — Message-ID tracking means restarts never cause duplicate replies.
- 🧪 **Dry-run mode** — test the full pipeline with zero real-world side effects.
- 🔒 **Loop prevention** — never replies to itself, no-reply addresses, or mailer-daemons.
- 🔑 **Authenticated access** — Clerk-backed sign-in gates the dashboard and onboarding flow.

---

## 🗺️ Roadmap

- [x] 🌐 Rebrand and ship a landing page + authenticated dashboard (Inboxly)
- [x] 🔑 Add Clerk-based auth and a FastAPI backend
- [ ] 🖥️ Move always-on polling from notebook logic into a deployable background service
- [ ] 📊 Expand the dashboard with analytics on reply history and response rates
- [ ] 🌐 Support multiple inbox providers beyond Gmail

---

**Built with 🧠 + ☕ as an agentic AI project in my portfolio.**

</div>
