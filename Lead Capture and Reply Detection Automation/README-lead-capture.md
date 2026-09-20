# Lead Capture & Reply Detection System

An end-to-end lead lifecycle automation: captures leads through a web form, routes and replies based on intent, logs every lead to a tracking sheet, follows up automatically, detects when a lead replies, alerts the team on WhatsApp, and logs its own errors — built in n8n.

## The Problem

Most "lead capture" automations stop at "send a confirmation email." This one covers the full loop: what happens *after* the form is submitted. Did the lead get the right reply for what they actually asked for? Did anyone follow up if they went quiet? Did they eventually respond — and did the team find out in time to act on it? And if any step in that chain breaks, does anyone find out before a lead falls through the cracks?

This workflow answers all four.

## How It Works

1. **Capture** — an embedded n8n form ("Lead Capture Form") collects Name, Phone Number, Email, and which service the lead is interested in (Chat with Customer Agent, Place Order Flow, View/Update Cart, Product Inquiry, or Complaint/Support).
2. **Validation gate** — before anything else runs, an If node checks that a phone number was actually provided.
3. **Field prep** — a Set node stamps the submission time and splits the full name into first/last for personalized messaging later.
4. **Intent-based routing** — a Switch node sends the lead down one of five branches based on which service they selected, and each branch sends a reply written for that specific intent rather than one generic template.
5. **Lead register logging** — four of the five branches log the lead (name, email, service, status, timestamps) to a "Lead Capture Register" Google Sheet.
6. **Automatic follow-up** — after a short wait, the workflow re-checks the lead's status, and if no other action has updated it, sends a follow-up email and marks the row accordingly.
7. **Reply detection** — a separate Gmail trigger polls the inbox for reply threads, extracts the sender and message, matches it back to the original lead by email, and updates their status to "Responded."
8. **Team alerting** — the moment a reply is matched, a WhatsApp notification goes out to the team with the lead's name, service, email, and what they said — so a real reply doesn't sit unread in an inbox.
9. **Error handling** — any node failure anywhere in the workflow is caught by a dedicated Error Trigger, logged to an Error Log sheet, and emailed to the workflow owner as a formatted alert.

## Why the Error-Handling Branch Matters

Most automations are demoed as if they never fail. This one assumes they will. Wiring a dedicated error branch that logs the failure *and* proactively emails the owner means a broken node gets caught the same day, not discovered a week later when someone asks why a lead was never contacted. It's a small addition that signals the difference between a script and a system meant to run unattended.

## Tech Stack

| Layer | Tool |
|---|---|
| Lead intake | n8n Form Trigger |
| Routing | n8n Switch node |
| Email (send + reply trigger) | Gmail API |
| Lead tracking | Google Sheets |
| Team alerting | WhatsApp (community automation node) |
| Error monitoring | n8n Error Trigger + Google Sheets + Gmail |

## Screenshots

[Lead Capture Form] <img width="960" height="540" alt="Lead Capture Submission Form" src="https://github.com/user-attachments/assets/4264bda2-239f-47eb-8b02-932a1bbeb2f6" />

![Lead Register Sheet] <img width="960" height="540" alt="Lead Capture Register" src="https://github.com/user-attachments/assets/3130f0e5-3b8d-45e4-b49b-4b5bc235caa4" />

![Error Log tab — a real logged error entry] <img width="960" height="540" alt="Error Log" src="https://github.com/user-attachments/assets/93f960e4-a172-4e60-91ab-f907b6dba80e" />

[Workflow] <img width="960" height="540" alt="Lead Capture and Reply Detection System Workflow " src="https://github.com/user-attachments/assets/8b1c9169-2c6b-4308-a6d4-5b1b1a2b1cba" />

## Known Limitations & Next Steps

- **One branch skips the register entirely.** Leads who select "Chat with Customer Agent" get a reply but are never logged to the Lead Register — meaning they're invisible to the follow-up and reply-detection system. Fix: route this branch through the same logging step as the other four.
- **No fallback on the Switch node.** Routing depends on an exact string match against the dropdown value (including trailing spaces baked into the option text). If the form's dropdown options are ever edited without updating the Switch conditions to match exactly, those leads fall through with no reply and no log entry.
- **The follow-up wait is only 1 minute.** That's almost certainly a testing value — a real nurture sequence would wait hours or days before following up, not sixty seconds.
- **Follow-up doesn't flip the "needs follow-up" flag.** After sending the one-time follow-up email, the row still reads `FOLLOW UP NEEDED: Yes`. Harmless in the current one-shot design, but would cause duplicate sends if this branch were ever triggered by a recurring poll instead of running once inline.
- **Lookups match on email only.** Two leads sharing an inbox (e.g. a shared team or family email) would collide in the register.
- **Reply detection is a broad Gmail search (`Subject: Re:`).** This could match replies to unrelated emails that happen to start with "Re:", not just replies to this workflow's outreach.

## Setup

1. Import `Lead_Capture_and_Reply_Detection_System_Workflow.json` into n8n.
2. Connect credentials: Gmail OAuth2 (trigger + send), a Google Sheets connection to your own Lead Register and Error Log sheets, and your WhatsApp automation node's credentials.
3. Replace the hardcoded Google Sheet document ID, WhatsApp phone number, and error-alert email address with your own.
4. Publish the form trigger and use its generated URL wherever you want to collect leads.
5. Activate the workflow.

---

Built by **Dr. Chibuzo Jude Mbama**  Physician and AI automation engineer. 
[Portfolio](https://judethetaken-portfolio.vercel.app) · [GitHub](https://github.com/Judeintechwastaken) · [LinkedIn](https://www.linkedin.com/in/jude-mbama-md-a87072354/)

