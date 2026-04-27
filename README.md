# n8n-email-classifier

***

```markdown
# 📧 PrimaSolusi AI Email Automation — n8n Workflow

An enterprise-grade AI email classification and routing system built for consulting agencies. This workflow autonomously reads incoming emails, uses LLMs to categorize them (Clients, Candidates, or Spam), and executes specialized actions for each category—all while logging every step to Google Sheets.

## 📊 Workflow Architecture

```text
Gmail Trigger (New email received)
    ↓
IF Node (Self-loop & bounce prevention)
    ↓
Google Sheets (Log initial incoming email)
    ↓
Markdown Node (Clean HTML syntax to plain text)
    ↓
Basic LLM Chain + Structured Output Parser (Summarize & Classify in one API call)
    ↓
Switch Router (Branching based on category)
    ├── Branch A: CANDIDATE 
    │     ↳ Gmail (Send Auto-Reply) → Sheets (Log Candidate) → Sheets (Update Main Log to SENT)
    │
    ├── Branch B: CLIENT (Corporate)
    │     ↳ AI Agent (Write Draft) → Gmail (Create Draft) → Slack (Alert Sales) → Sheets (Log Client) → Sheets (Update Main Log to DRAFTED)
    │
    └── Branch C: SPAM
          ↳ Gmail (Add 'SPAM-AUTO' Label) → Sheets (Update Main Log to SPAM)
```

## ✨ Engineering Highlights

- **Infinite Loop Prevention**: Includes a dedicated IF node immediately after the trigger to filter out self-sent emails and auto-replies, eliminating the risk of infinite API loops.
- **LLM Optimization**: Combines 'Summarization' and 'Classification' into a single Basic LLM Chain using a Structured Output Parser. This significantly reduces token consumption, speeds up execution, and returns clean, easily mappable JSON data.
- **Automated Drafting & Alerts**: For high-value corporate clients, the AI doesn't just categorize—it proactively drafts a contextual reply in Gmail and pings the Slack channel so the sales team can review and send it immediately.
- **Error Handling**: (Optional but recommended) Compatible with separate error-handling workflows to catch and send Slack notifications if any node crashes.

## 🛠️ Prerequisites

| Service | Purpose |
|---|---|
| **Gmail Account** | Email monitoring, sending auto-replies, and creating drafts |
| **Google Gemini / LLM** | Email summarization, classification, and drafting |
| **Google Sheets** | Database for logging inbox traffic, candidates, and clients |
| **Slack** | Real-time notifications for the sales/operations team |

## 🚀 Quick Setup

### Step 1 — Import the Workflow
1. Open your n8n canvas and click **Import from file**.
2. Select the `PrimaSolusi-Email-Automation.json` file.

### Step 2 — Configure Google Sheets (Crucial)
To ensure the data mapping works flawlessly, create a Google Spreadsheet with **three distinct tabs** and exact column headers:

**Tab 1: Inbox Log**
`Tanggal Masuk` | `Email Pengirim` | `Subjek` | `Kategori` | `Summary` | `Status` | `Message ID`

**Tab 2: Kandidat**
`Tanggal Masuk` | `Email Kandidat` | `Subjek (Posisi)` | `Status Auto-Reply`

**Tab 3: Klien**
`Tanggal Masuk` | `Email Klien` | `Nama Klien` | `Status Auto-Reply`

*Link your Google Sheets credentials and select the corresponding tabs in the respective Sheets nodes.*

### Step 3 — Set Up Credentials & Environment
1. **Gmail:** Connect your Google OAuth2 credentials to both the Trigger and Action nodes.
2. **LLM:** Add your AI API key (e.g., Google Gemini) to the Chat Model nodes.
3. **Slack:** Connect your Slack account and specify the target `#sales-leads` channel in the Slack node.

### Step 4 — Test & Activate
1. Filter your own email address in the **IF Node** to test safely.
2. Click **Execute Workflow** and send three test emails simulating a job application, a corporate inquiry, and a random spam message.
3. Verify that the Google Sheets populate correctly and Slack alerts trigger.
4. Toggle the workflow to **Active**.
