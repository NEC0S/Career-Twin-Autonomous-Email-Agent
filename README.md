


<div align="center">

# 🤖 Agentic AI Career Email Assistant

### An autonomous agent that reads your inbox, thinks, and decides — reply itself or call you in.

![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)
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

> 🎬 **Video walkthrough coming soon!** A short demo showing the agent triaging a live inbox, deciding between an autonomous reply and a human escalation, will be embedded here.

<div align="center">

https://github.com/user-attachments/assets/96e10f17-6e6a-496e-95bb-f2ff4d0f72b0

</div>

---

## ✨ Overview

This project is a fully **agentic AI system** — no orchestration frameworks, no black-box agent libraries. Just an LLM given a clear goal, two tools, and the judgment to choose between them on its own.

It watches your inbox for career-related emails (recruiters, interviews, hiring managers) and for each one, autonomously decides:

- ✅ **"I know this — I'll reply myself."** → drafts and sends a threaded, professional reply grounded strictly in your resume and profile.
- 🙋 **"I don't know this — ask the human."** → sends you a push notification instead of guessing.

No hallucinated answers to recruiters. No missed opportunities. No manual inbox babysitting.

---

## 🧠 How It Works

1. **📥 Inbox Monitoring** — Connects via IMAP and scans recent emails, regardless of read/unread status.
2. **🔍 Keyword Triage** — Regex-filters for career-relevant keywords (`job`, `recruiter`, `interview`, `hiring`, `role`, etc.). Everything else is left untouched.
3. **🤔 Agentic Decision-Making** — The filtered email + your resume/summary are handed to Gemini (via an OpenAI-compatible API). The model must call **exactly one** of two tools (`tool_choice` enforced):
   - 🟢 `send_reply_tool` — writes and sends a first-person, professional reply; optionally attaches your resume.
   - 🔴 `notify_me_tool` — pushes a notification to your phone via Pushover instead of guessing.
4. **🧾 State Tracking** — Every handled email's Message-ID is logged, so restarts never trigger duplicate replies.

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
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| 🐍 Core Language | Python |
| 📬 Email Protocols | IMAP (reading) + SMTP (threaded replies) |
| 🧠 LLM | Google Gemini via OpenAI-compatible API |
| 🔧 Agent Pattern | Manual function/tool calling (JSON Schema, `tool_choice="required"`) |
| 📄 Resume Parsing | `pypdf` |
| 📲 Notifications | Pushover API |
| 💾 State Management | JSON-based idempotent Message-ID tracking |

---

## 📁 Project Structure

```
.
├── career_email_agent.ipynb    # main notebook — the whole agent lives here
├── twin/
│   ├── resume.pdf                # your resume (LinkedIn PDF export works great)
│   └── summary.txt               # plain-text: background, status, skills, goals
├── handled_message_ids.json      # auto-generated — dedup state, don't commit
├── .env                          # credentials — never commit this
└── .gitignore
```

---

## ⚙️ Setup

### 1️⃣ Add your context files

Inside a `twin/` folder next to the notebook:
- `resume.pdf` — your resume or LinkedIn PDF export
- `summary.txt` — a specific, detailed summary of your background, current status, and what you're looking for. **The more specific, the smarter the agent's replies.**

### 2️⃣ Install dependencies

```bash
pip install python-dotenv openai pypdf requests
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

For always-on deployment, adapt the notebook logic into a standalone `.py` script and run it via `cron` or a background process.

---

## 🛡️ Reliability & Safety

- 🚫 **Never guesses** — if the answer isn't explicitly in your resume/summary, it escalates to you instead of fabricating a reply.
- 🔁 **Idempotent** — Message-ID tracking means restarts never cause duplicate replies.
- 🧪 **Dry-run mode** — test the full pipeline with zero real-world side effects.
- 🔒 **Loop prevention** — never replies to itself, no-reply addresses, or mailer-daemons.

---

## 🗺️ Roadmap

- [ ] 🎥 Embed demo video
- [ ] 🖥️ Convert notebook to standalone deployable script
- [ ] 📊 Add logging/analytics dashboard for reply history
- [ ] 🌐 Support multiple inbox providers beyond Gmail

---

## 📜 License

This is a personal project — use, fork, and adapt freely under the MIT License.

<div align="center">

**Built with 🧠 + ☕ as an agentic AI portfolio project.**

</div>
