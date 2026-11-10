# Job Application Assistant 🤖💼

[![n8n](https://img.shields.io/badge/Automation-n8n-FF6D5A?logo=n8n)](https://n8n.io/)
[![Google Gemini](https://img.shields.io/badge/AI-Gemini%20Flash-4285F4?logo=google)](https://ai.google.dev/)
[![Ollama](https://img.shields.io/badge/Local%20AI-Ollama%20llama3.2-black?logo=ollama)](https://ollama.com/)
[![Telegram](https://img.shields.io/badge/Interface-Telegram%20Bot-26A5E4?logo=telegram)](https://telegram.org/)
[![Status](https://img.shields.io/badge/Status-Phases%201%20%26%202%20Active-brightgreen)](#-project-status)

> A human-in-the-loop automation pipeline powered by **n8n**, **Gemini**, **Ollama**, and **Telegram** that speeds up job searching, scoring, and application drafting while keeping complete user control.

---

## 📌 Overview

The **Job Application Assistant** automates the repetitive, error-prone tasks of job hunting—reading job postings, judging candidate fit, screening for scams, and drafting tailored emails—without taking away human authority.

You interact with a private **Telegram Bot** by forwarding job links or raw text. The assistant processes the data, calculates a fit score based on your master resume and rules, and asks for your explicit approval before drafting an application. Once approved, it generates a tailored response, attaches your CV from Google Drive, and sends it directly from your Gmail account.

---

## ✨ Key Features

* **📱 Human-in-the-Loop Telegram Interface:** Approve or skip job drafts with simple inline buttons directly in your private chat.
* **⚡ Two-Stage Approval Flow:** Confirms interest *before* drafting to save AI quota and eliminate wasted calls.
* **🛡️ Scam & Fit Scoring:** Scores jobs (0–100), flags seniority/domain mismatches, and highlights scam risks (e.g., fee requests).
* **🔒 Privacy-First Data Shielding:** Personal Identifiable Information (phone, email, home address) is stripped before sending context to AI models. Contact info is injected locally via n8n prior to sending.
* **📊 Automatic Google Sheets Tracker:** Logs every job, status (`Awaiting decision`, `Pending approval`, `Sent`, `Applied (manual)`, `Skipped`), unique job keys (to prevent duplicate applies), and auto-scheduled follow-up dates (7 days).
* 🔔 **Error Alert Workflow:** Sends instant Telegram notifications if an API fails, Ollama goes offline, or quota limits are reached.

---

## 🏗️ System Architecture

```
[ User via Telegram ] ──► Telegram Bot (Webhook)
                                │
                                ▼
                 ┌─────────────────────────────┐
                 │ Job Assistant Workflow (n8n)│
                 └──────────────┬──────────────┘
                                │
   ┌────────────────────────────┼────────────────────────────┐
   ▼                            ▼                            ▼
Ollama llama3.2           Gemini Flash                Google Sheets
(Local job check)      (Extract, Score, Draft)     (Tracker & Profile)
                                │                            │
                                ▼                            ▼
                          Google Drive                    Gmail
                        (Fetch CV PDF)               (Send Email)
```

---

## 🛠️ Stack & Technologies

| Component | Technology | Role |
| :--- | :--- | :--- |
| **Orchestration** | n8n (self-hosted via npm) | Runs all workflows locally |
| **Tunneling** | ngrok | Provides secure HTTPS webhook endpoint for Telegram |
| **User Interface** | Telegram Bot | Private command and approval channel |
| **Primary AI** | Google Gemini Flash | Extraction, fit scoring, scam detection, drafting |
| **Local AI** | Ollama (`llama3.2`) | Fast local check to verify text is a job posting |
| **Data & Files** | Google Sheets & Drive | Stores application logs, user profile, and CV PDF |
| **Email Delivery** | Gmail API | Sends customized applications with CV attached |

---

## 📊 Database Design (Google Sheets)

The pipeline relies on a single **Google Sheet** with two active tabs:

### 1. `Applications` Tab
| Column | Description |
| :--- | :--- |
| `ID` / `JobKey` | Unique tracking ID and normalized `company-title` key for duplicate detection. |
| `Title` / `Company` / `Location` | Extracted core posting details. |
| `Link` / `Method` / `ToEmail` | Application path (Email vs. Form) and recipient contact. |
| `Subject` / `Body` | Drafted email subject and tailored cover text. |
| `Score` / `Verdict` / `ScamRisk` | Gemini fit score (0–100), decision recommendation, and safety flags. |
| `Status` | Current state: `Awaiting decision`, `Pending approval`, `Sent`, `Applied (manual)`, `Skipped`. |
| `SentAt` / `FollowUpDate` | Timestamp of send date and automated +7 days follow-up marker. |

### 2. `Profile` Tab
Stores key-value pairs used for evaluation and drafting:
* `ResumeText` (PII removed)
* `Rules` (Target roles, excluded titles, minimum acceptable score)
* `FullName`, `Email`, `Phone`, `Signature` (Injected locally by n8n)
* `ResumeFileId` (Google Drive PDF ID)

---

## 🚀 Setup & Installation

### Prerequisites
* **Node.js** (npm) installed on host machine
* **Ollama** installed locally with `llama3.2` model (`ollama run llama3.2`)
* **ngrok** account for webhook tunneling
* **Google Cloud Console** project with Gmail, Sheets, and Drive APIs enabled
* **Telegram Bot** token from [@BotFather](https://t.me/BotFather)

### Quick Start

1. **Clone the repository:**
   ```bash
   git clone https://github.com/your-username/job-application-assistant.git
   cd job-application-assistant
   ```

2. **Start Ollama:**
   ```bash
   ollama run llama3.2
   ```

3. **Expose n8n port via ngrok:**
   ```bash
   ngrok http 5678
   ```

4. **Launch n8n with Webhook configuration:**
   ```powershell
   $env:WEBHOOK_URL="https://<your-ngrok-subdomain>.ngrok-free.app"
   $env:N8N_EDITOR_BASE_URL="http://localhost:5678"
   n8n start
   ```

5. **Import Workflows & Configure Credentials:**
   * Open `http://localhost:5678`.
   * Import workflow JSON files from the `/workflows` directory.
   * Add Google OAuth2, Gemini API Key, Ollama, and Telegram credentials.
   * Set the **Error Alert Workflow** in n8n settings as the default error handler.

---

## 🔒 Security & Safety Controls

* **User Allow-List:** Telegram bot ignores all messages from unauthorized Telegram user IDs.
* **No Unsanctioned Sends:** Emails are **never** sent automatically; sending requires a explicit Telegram button press.
* **Prompt Injection Resilience:** Job descriptions are treated purely as untrusted data; AI only outputs JSON/text drafts.
* **Duplicate Prevention:** Job keys flag previously processed postings to prevent repeated outreach.

---

## 🗺️ Roadmap & Future Work

- [x] Phase 1: Telegram intake, fit scoring, scam detection, Google Sheets logging.
- [x] Phase 2: Two-stage Telegram approval, automated email drafting & Gmail sending with Drive CV attachment.
- [ ] Phase 3: Scheduled **Job Digest** scanning public job APIs (Remotive, RemoteOK, Arbeitnow) and job-alert emails.
- [ ] Phase 4: Form-filling assistance for non-email applications.
- [ ] Phase 5: Automatic follow-up reminders and inbox reply tracking.

---

## 📄 Author & License

**Author:** Fahim Foysal  
**Date:** November 2026  
**License:** [MIT](LICENSE)