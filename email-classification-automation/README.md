# Email Classification & Response Automation

An AI-driven email triage system that classifies inbound business email by department and priority, drafts a context-aware reply, routes it to the right team for human approval, and logs every decision for auditability — built in n8n with Google Gemini as the reasoning engine.

## The Problem

Inbound business email is a bottleneck: someone has to read every message, figure out which team owns it, decide how urgent it is, and write a response — all before the real work of actually handling the request even starts. Delay on the wrong message (a legal threat, an overdue invoice, a compliance complaint) is where that bottleneck gets expensive.

This workflow removes the triage step, not the human decision. It reads every inbound email, classifies it, and drafts the reply — but nothing reaches a customer until a person on the relevant team approves it.

## How It Works

```mermaid
flowchart TD
    A[Gmail Trigger — polls inbox every minute] --> B[AI Agent — Gemini 2.5 Flash]
    B --> C[Parse & clean JSON output]
    C --> D[Map fields: Department / Priority / Reason / Reply]
    D --> E{Switch on Department}
    E -->|Sales| F1[Slack: approval request]
    E -->|Customer Service| F2[Slack: approval request]
    E -->|Human Resources| F3[Slack: approval request]
    E -->|Finance| F4[Slack: approval request]
    E -->|Operations| F5[Slack: approval request]
    F1 & F2 & F3 & F4 & F5 --> G[Wait for human response]
    G --> H{Approved?}
    H -->|Yes| I[Send reply via Gmail]
    I --> J[Log to Google Sheets]
    H -->|No| K[Discarded — no reply, no log]
```

1. **Trigger** — a Gmail polling trigger checks the inbox every minute for new mail.
2. **Classification & drafting** — the email subject and body are passed to an AI Agent node running Google Gemini 2.5 Flash. A single prompt does two jobs at once: classify the email (department + priority) and draft a complete three-paragraph reply, including an SLA commitment scaled to priority (High → 2 hours, Medium → 24 hours, Low → 3 business days).
3. **Defensive parsing** — LLMs don't always return clean JSON. A Code node strips any stray markdown fences before parsing, so a slightly-off model response doesn't break the workflow.
4. **Routing** — a Switch node sends the email down one of five department branches (Sales, Customer Service, Human Resources, Finance, Operations) based on the AI's classification.
5. **Human-in-the-loop approval** — each branch posts an interactive Slack message to that department's channel, showing the sender, subject, priority, the AI's stated reasoning, and the full proposed reply. A human replies **Approve** or **Reject** directly in Slack.
6. **Conditional send** — only on approval does the workflow send the drafted reply back to the original sender via Gmail.
7. **Audit log** — every approved classification (timestamp, sender, subject, department, priority, reasoning) is appended to a Google Sheet for reporting and quality review.

## Why the Approval Gate Matters

The easy version of this workflow lets the AI send replies straight away. This one doesn't, on purpose. An AI model can misjudge tone, miss context, or occasionally hallucinate a detail — and an email that goes out under "The Support Team" signature can't be unsent. Routing every draft through a human checkpoint before it touches a real inbox keeps the speed benefit of automation without inheriting its failure mode. It's the same instinct that shapes triage in a clinical setting: escalate uncertainty to a person, don't let the system make the final call alone.

## Tech Stack

| Layer | Tool |
|---|---|
| Orchestration | n8n |
| Email trigger & send | Gmail API |
| Classification & drafting | Google Gemini 2.5 Flash (via LangChain node) |
| Human approval | Slack (interactive send-and-wait) |
| Parsing / data shaping | JavaScript (Code node) |
| Audit logging | Google Sheets |

## Screenshots

**Inbound emails (triggers)**

![Finance — overdue invoice](./screenshots/01-inbound-email-finance.png)
![HR — formal complaint](./screenshots/02-inbound-email-hr.png)
![Customer Service — delayed order](./screenshots/03-inbound-email-customer-service.png)
![Operations — supply restock request](./screenshots/04-inbound-email-operations.png)

**The workflow in n8n**

![Full workflow canvas](./screenshots/06-workflow-canvas-overview.png)
![Node execution / data debug view](./screenshots/05-workflow-node-debug-view.png)

**Slack human-in-the-loop approvals**

![Approval request — Human Resources](./screenshots/07-slack-approval-human-resources.png)
![Approval request — Finance (Critical priority)](./screenshots/08-slack-approval-finance.png)
![Approval request — Customer Service](./screenshots/09-slack-approval-customer-service.png)
![Approval request — Operations](./screenshots/10-slack-approval-operations.png)

**Replies sent after approval**

![Sent reply — Human Resources case](./screenshots/11-sent-reply-human-resources.jpg)
![Sent reply — Finance case](./screenshots/12-sent-reply-finance.jpg)
![Sent reply — Operations case](./screenshots/13-sent-reply-operations.jpg)

**Audit log**

![Every classification logged to Google Sheets](./screenshots/14-audit-log-google-sheets.png)

## Known Limitations & Next Steps

Being upfront about these is part of the engineering — a workflow presented as flawless is less convincing than one with a clear-eyed list of what's next:

- **"Other" classification has no destination.** The Switch node only routes five departments; an email the AI tags as "Other" currently falls through with no Slack notification, no reply, and no log entry. Fix: add a default/fallback output routing to a general triage channel.
- **Rejected emails aren't logged.** The Google Sheets append only happens after an approved send, so a rejected classification leaves no audit trail. Fix: log every classification outcome, approved or not, with a status column.
- **No handling for malformed model output.** If Gemini returns anything the Code node can't parse as JSON, the run fails outright rather than degrading gracefully. Fix: wrap the parse in a try/catch with a fallback "needs manual review" path.
- **No approval timeout.** If nobody responds in Slack, the workflow waits indefinitely. Fix: add a timeout with automatic escalation.
- **No de-duplication check.** Repeated trigger polls could theoretically reprocess the same message. Fix: track processed message IDs.

## Setup

1. Import `Email_Classification_Workflow_JUDE.json` into n8n.
2. Connect credentials: Gmail OAuth2 (trigger + send), Google Gemini API key, Slack OAuth2 (with channel IDs for each department), and a Google Sheets connection to your own tracking sheet.
3. Update the Switch node's department values and the Slack channel IDs to match your own team structure.
4. Activate the workflow — new inbox mail will start flowing through within a minute.

---

Built by **Dr. Chibuzo Jude Mbama** — physician and AI automation engineer, applying the same risk-aware, human-checked design instincts from clinical practice to business process automation.
