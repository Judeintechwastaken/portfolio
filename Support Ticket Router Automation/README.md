# Support Ticket Router

An n8n workflow that takes support tickets from a web form, sorts them into Billing, Technical, Sales or General by keyword, posts each one to the matching Slack channel, sends the customer an auto reply carrying their ticket ID, and logs every ticket to a Google Sheet.

`n8n` · `Slack API` · `Google Sheets` · `Gmail` · `JavaScript`

## Demo

**Video:** [add Loom link here]

**The workflow**

[Workflow canvas] <img width="960" height="540" alt="Workflow Canvas" src="https://github.com/user-attachments/assets/8c1548a2-76ea-4899-a8d0-6dc0158b27f4" />


**A ticket routed to Slack**

[Slack message] <img width="960" height="540" alt="Slack message to billing Dept" src="https://github.com/user-attachments/assets/432aa39a-5cac-40bd-90b7-86112e4c8169" />

**The ticket log**

[Tickets sheet] <img width="960" height="540" alt="Ticket Spreadsheet" src="https://github.com/user-attachments/assets/1e208b91-4e45-485c-949e-dfb95e7ecab1" />


### Sample ticket

```
🎫 New ticket TCK-20260921-8882
Category: Billing
From: Ada Obi (ada.obi@example.com)
Subject: Charged twice for my March invoice
Message: Hello, I was charged twice for invoice #4821 on 3 March, once by card
and once by bank transfer. Please refund the duplicate payment and confirm
once it's done.
```

The data is a sample support inbox for a small business with billing, technical, sales and general enquiries.

## How it works

```
Support Form (n8n Form Trigger)
  → Normalise (Set)
  → Ticket ID (Code)
  → Category (Switch, Rules mode)
       ├─ Billing   → Post to Billing   (Slack) ─┐
       ├─ Technical → Post to Tech      (Slack) ─┤
       ├─ Sales     → Post to Sales     (Slack) ─┼→ Merge (Append, 4 inputs)
       └─ Fallback  → Post to General   (Slack) ─┘
                                                     → Ticket Details (Set)
                                                     → Auto Reply (Gmail)
                                                     → Log Ticket (Google Sheets)
```

| Node | Type | What it does |
|---|---|---|
| Support Form | n8n Form Trigger | Public form with Name, Email, Subject and Message fields. |
| Normalise | Set | Cleans the raw form fields into safe values and builds `search_text` — the subject and message joined and lowercased once, so every downstream match is case-insensitive by construction. |
| Ticket ID | Code | Generates a reference like `TCK-20260921-4817`: a prefix, today's date in the workflow timezone, and four random digits. |
| Category | Switch (Rules) | Tests `search_text` against a keyword list for each category, in order: Billing, then Technical, then Sales. The Fallback Output catches anything that matches none of them. |
| Post to Billing / Tech / Sales / General | Slack | Four identically-shaped messages, one per Category output, each posted to its own channel. |
| Merge | Merge (Append, 4 inputs) | Brings the four branches back into a single line, since only one fires per ticket. |
| Ticket Details | Set | Restores the ticket's fields after the Slack node's own response overwrote them, and works out `category` and `routed_to` from which branch executed. |
| Auto Reply | Gmail | Sends the customer a four-line reply naming their ticket ID and which team it went to. |
| Log Ticket | Google Sheets | Appends a row to the `Tickets` sheet with every field, including category and routing. |

## Design decisions

- **Lowercase once, in Normalise.** `search_text` is built once, immediately after the form submits, rather than inside each Switch rule. That is what keeps "Refund", "REFUND" and "refund" all matching the same rule — doing the lowercasing in three or four separate places is exactly where that kind of inconsistency creeps in.
- **First match wins.** The Switch node's "send to all matching outputs" is off, so a ticket that matches two categories still goes to only one, whichever rule is checked first. Billing is checked before Technical and Sales, because a message like "my payment failed and the screen is broken" is a money problem before it's a technical one, and money problems are the most time-sensitive and hardest for a customer to undo themselves.
- **Nothing is ever silently dropped.** The Switch node's Fallback Output is enabled and wired to General, so a ticket that matches no keyword still gets a Slack post, an auto reply and a sheet row — it just gets filed under General instead of disappearing.
- **One shared body for all four channels.** All four Slack messages use the same layout — ticket ID, category, name and email, subject, and the first 200 characters of the message — so a support channel reads consistently regardless of which team it lands in.
- **Every send is logged.** The Tickets sheet answers "what do people actually contact us about", since Category and Routed To are stored on every row, not just posted to Slack and forgotten.

## Sheet format

**`Tickets` tab** (create the header row first)

| Ticket ID | Received At | Name | Email | Subject | Message | Category | Routed To | Status |
|---|---|---|---|---|---|---|---|---|

## Setup

1. **Import the workflow.** In n8n, open the workflow menu, choose *Import from file*, and select `support-ticket-router.json`.
2. **Prepare the sheet.** Create a Google Sheet named `Tickets` with the header row above.
3. **Create four Slack channels**: `billing`, `tech-support`, `sales` and `general`.
4. **Create a Slack app.** At `api.slack.com/apps`, create a new app from scratch, add the Bot Token Scopes `chat:write`, `chat:write.public` and `channels:read`, install it to your workspace, and invite it to all four channels (`/invite @YourAppName`).
5. **Connect Slack, Google Sheets and Gmail** credentials in their respective nodes.
6. **Set the timezone.** In the workflow settings, choose your timezone, so ticket IDs and timestamps land on the correct day.
7. **Test, then publish.** Submit a test ticket for each category through the form's test URL, check that it lands in the right Slack channel, the right auto reply arrives, and the right row appears in the sheet. Then publish the workflow so the production form URL goes live.

## Testing the fallback branch

Submit a ticket whose subject and message contain none of the keywords below, for example "Where can I find your office hours?" It should route through the Fallback Output to General, and still produce a Slack post, an auto reply and a sheet row.

## Keyword list per category

- **Billing:** invoice, refund, payment, receipt, charged, subscription
- **Technical:** broken, error, not working, crash, login, password, faulty, screen
- **Sales:** price, pricing, quote, buy, order, demo, how much, availability

## Known limitations and next steps

- Matching is keyword-based, not semantic, so it can mismatch on phrasing the keyword list doesn't anticipate (an LLM-based classifier would generalise better, at the cost of latency and predictability).
- "Contains" matching can match inside other words (for example "order" inside "border"). Acceptable here, but worth tightening with word-boundary matching if the keyword list grows.
- Ticket IDs use four random digits, so a collision is possible (about a 4% chance of a same-day repeat at 30 tickets a day). A real system would check the sheet for the ID first or use a counter.
- The auto reply's response time ("aim to reply within one business day") is a promise that has to be kept in step with the team's actual capacity — it isn't enforced by the workflow.
- Possible extensions: a Slack slash command to change a ticket's Status, a weekly count of tickets by category, and a duplicate-ticket check on repeated emails.

## Repository contents

```
support-ticket-router/
├── README.md
├── support-ticket-router.json     # n8n workflow export
└── screenshots/
    ├── 01-workflow-canvas.png
    ├── 04-slack-billing.png
    └── 06-tickets-sheet.png
```

## Author

Built by **Dr. Chibuzo Jude Mbama**  Physician and AI automation engineer. 
[Portfolio](https://judethetaken-portfolio.vercel.app) · [GitHub](https://github.com/Judeintechwastaken) · [LinkedIn](https://www.linkedin.com/in/jude-mbama-md-a87072354/)


