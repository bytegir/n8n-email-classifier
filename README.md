# AI Email Triage Automation (n8n)

An n8n workflow that reads incoming Gmail messages, uses an LLM to classify them as **Client**, **Candidate**, or **Spam**, and runs a different action for each category. Every step is logged to Google Sheets, so nothing happens without a record.

> **Demo project.** The scenario is a fictional consulting agency ("PrimaSolusi") that receives a mix of client inquiries, job applications, and spam in one inbox. The workflow is designed to be adapted to a real inbox.

## The Problem

Shared inboxes mix very different kinds of messages. Someone has to read each one, decide what it is, reply or forward it, and keep track of what was handled. This workflow does the sorting, the first response, and the tracking automatically, and leaves the high-value replies for a human to review.

<img width="830" height="308" alt="Screenshot 2026-10-21 030414" src="https://github.com/user-attachments/assets/5f725146-315e-4fb9-960b-16f1f3c1e9b1" />

## What It Does

| Category | Action |
|---|---|
| **Candidate** | Sends an automatic reply, logs the candidate, and marks the inbox entry as `SENT` |
| **Client** | An AI agent writes a contextual reply as a **Gmail draft** (not sent), alerts the sales team on Slack, logs the client, and marks the entry as `DRAFTED` |
| **Spam** | Adds a `SPAM-AUTO` label in Gmail and marks the entry as `SPAM` |

Client replies are saved as drafts on purpose: a person reviews and sends them, so the AI never contacts an important prospect unsupervised.

## Workflow Architecture

```text
Gmail Trigger (new email received)
    ↓
IF Node (self-loop and bounce prevention)
    ↓
Google Sheets (log the incoming email)
    ↓
Markdown Node (clean HTML into plain text)
    ↓
Basic LLM Chain + Structured Output Parser (summarize and classify in one call)
    ↓
Switch Router (branch by category)
    ├── CANDIDATE
    │     ↳ Gmail (auto-reply) → Sheets (log candidate) → Sheets (update main log to SENT)
    │
    ├── CLIENT
    │     ↳ AI Agent (write draft) → Gmail (create draft) → Slack (alert sales)
    │       → Sheets (log client) → Sheets (update main log to DRAFTED)
    │
    └── SPAM
          ↳ Gmail (add 'SPAM-AUTO' label) → Sheets (update main log to SPAM)
```

## Engineering Highlights

- **Infinite loop prevention:** an IF node right after the trigger filters out self-sent emails and auto-replies, so the workflow can never trigger itself.
- **One LLM call for two jobs:** summarization and classification run in a single Basic LLM Chain with a Structured Output Parser. This lowers token usage, speeds up execution, and returns clean JSON that maps directly to later nodes.
- **Human in the loop for important emails:** client replies are drafted by an AI agent but sent by a person, with a Slack alert so the team sees them immediately.
- **Full audit trail:** each email is logged on arrival and its status is updated at the end of its branch, so a stuck or failed run is easy to spot in the sheet.
- **Error handling:** compatible with a separate n8n error workflow that sends a Slack notification if any node fails (optional but recommended).

## Requirements

| Service | Purpose |
|---|---|
| **Gmail account** | Monitor the inbox, send auto-replies, create drafts, apply labels |
| **LLM (e.g. Google Gemini)** | Summarization, classification, and draft writing |
| **Google Sheets** | Log of inbox traffic, candidates, and clients |
| **Slack** | Real-time alerts for the sales team |

## Setup

### 1. Import the workflow

1. Open your n8n canvas and choose **Import from file**.
2. Select `email-classifier.json` from this repository.

### 2. Prepare Google Sheets

Create one spreadsheet with **three tabs**. The column headers must match exactly, because the Sheets nodes map data by header name.

**Tab 1: Inbox Log**
`Tanggal Masuk` | `Email Pengirim` | `Subjek` | `Kategori` | `Summary` | `Status` | `Message ID`

**Tab 2: Kandidat**
`Tanggal Masuk` | `Email Kandidat` | `Subjek (Posisi)` | `Status Auto-Reply`

**Tab 3: Klien**
`Tanggal Masuk` | `Email Klien` | `Nama Klien` | `Status Auto-Reply`

The headers are in Indonesian because the demo was built for an Indonesian agency. In English: *Date Received*, *Sender Email*, *Subject*, *Category*, *Summary*, *Status*, *Message ID*, *Candidate Email*, *Subject (Position)*, *Auto-Reply Status*, *Client Email*, *Client Name*. If you rename them, update the column mapping in each Sheets node as well.

### 3. Connect credentials

1. **Gmail:** connect Google OAuth2 credentials to the trigger and to every Gmail action node.
2. **LLM:** add your API key (e.g. Google Gemini) to the Chat Model nodes.
3. **Slack:** connect your account and choose the target channel (the demo uses `#sales-leads`).
4. **Google Sheets:** connect your credentials and select the matching tab in each Sheets node.

### 4. Test, then activate

1. In the IF node, restrict the workflow to your own email address so you can test safely.
2. Click **Execute Workflow** and send three test emails: a job application, a corporate inquiry, and a spam-like message.
3. Check that the Sheets are filled in correctly, the client draft appears in Gmail, and the Slack alert arrives.
4. Switch the workflow to **Active**.

## Adapting It to Your Business

- **Categories:** change the categories in the prompt and the Structured Output Parser, then add or remove branches in the Switch node to match.
- **Replies:** edit the auto-reply text and the AI agent's drafting instructions to match your tone and services.
- **Alert channel and sheet layout:** both can be changed in the Slack and Sheets nodes.

## Notes

- Built and tested as a portfolio demo with test emails, not on a live business inbox.
- Candidate auto-replies are sent automatically. If that is too aggressive for your use case, change that branch to create a draft instead, like the client branch.

## Tech Stack

n8n · Gmail · Google Gemini (any LLM supported by n8n) · Google Sheets · Slack
