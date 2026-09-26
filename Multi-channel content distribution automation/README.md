# Multi-Channel Content Distribution Automation

A Zapier automation that watches a blog's RSS feed and, the moment a new post goes live, automatically shares it to **Facebook, LinkedIn, WhatsApp and Slack** — no manual copy-pasting required.

## What it does

Most blogs have the same problem: writing the content is the expensive part, and it's already paid for, but *sharing* it depends on someone remembering to do it, and they don't always. This automation removes that dependency entirely.

The moment a post is published:
1. It's picked up from the blog's RSS feed
2. Cleaned up (HTML stripped, summary trimmed to a punchy 200-character caption)
3. Posted to a **Facebook Page**
4. Posted as a **LinkedIn Company Update**
5. Sent to a **WhatsApp group**
6. Announced in a **Slack channel**, so the team knows it's already out and doesn't duplicate the work

## How it works

Built entirely in **Zapier**, as an 8-step Zap:

| # | Step | What it does |
|---|------|---------------|
| 1 | **RSS by Zapier** — New Item in Feed | Watches the blog's RSS feed for new posts |
| 2 | **Filter by Zapier** | Only lets real posts through (title exists + link matches the blog's own domain) — filters out stray/malformed feed entries |
| 3 | **Formatter by Zapier** — Remove HTML Tags | Strips the `<p>`, `<img>` etc. tags RSS content arrives wrapped in |
| 4 | **Formatter by Zapier** — Truncate | Trims the cleaned text to 200 characters (+ ellipsis) so captions stay a hook, not a rewrite of the article |
| 5 | **Facebook Pages** — Create Page Post | Publishes to the Facebook Page |
| 6 | **LinkedIn** — Create Company Update | Publishes to the LinkedIn Company Page |
| 7 | **Webhooks by Zapier** — POST | Sends a WhatsApp message via [GreenAPI](https://green-api.com) to a target group |
| 8 | **Slack** — Send Channel Message | Posts an internal alert in a `#content` channel so the team knows it's live everywhere |

### Why GreenAPI for WhatsApp?

The official WhatsApp Business Cloud API requires business verification and approved message templates for outbound messages, which makes it slow to stand up for a project like this. GreenAPI authorizes a personal WhatsApp number by QR code and exposes a simple REST endpoint, so a POST request from Zapier is all that's needed to send a message. It's a fast, low-friction option — not a substitute for the official API in a production client rollout at scale.

## Caption strategy

Each destination gets a caption built the same way, adapted to the platform:

> **Hook** (title, or a reframed problem for LinkedIn) → **one-line summary** (trimmed to 200 characters) → **call to action + link**

The 200-character limit keeps the caption a teaser rather than a full restatement — the goal is the click-through, not reading the whole point without visiting the site.

## Demo

### Walkthrough video

A 5 minute walkthrough of the Zap and all four destinations receiving the same live post.

📹 **[Watch the walkthrough video] https://www.loom.com/share/76a7d988979c462c9fe5cef369904441**

### Screenshots

**The full Zap, end to end:**

| Steps 1–4 | Steps 5–8 |
|---|---|
| ![Zap editor, steps 1–4] <img width="960" height="540" alt="Page 1 Zap" src="https://github.com/user-attachments/assets/0831ac17-fbfe-4a76-8d90-1743b24da7fa" /> | ![Zap editor, steps 5–8] <img width="960" height="540" alt="Page 2 Zap" src="https://github.com/user-attachments/assets/1e6e2d64-4150-49c4-a508-9206b6649880" /> |

**Each step's configuration:**

| Step | Screenshot |
|------|------------|
| RSS trigger | [RSS trigger] <img width="960" height="540" alt="RSS Trigger" src="https://github.com/user-attachments/assets/3948fe43-ae4b-4a8b-9b97-8c34074545a6" /> |
| Filter conditions | [Filter] <img width="960" height="540" alt="Filter" src="https://github.com/user-attachments/assets/715dbe4b-4f2d-43d2-8f34-46225c746544" /> |
| Strip HTML | [Formatter — remove HTML] <img width="960" height="540" alt="Formatter Remove HTML" src="https://github.com/user-attachments/assets/82600cd2-b75a-459f-9c91-0e3aebd3e12e" /> |
| Trim to 200 chars | [Formatter — truncate] <img width="960" height="540" alt="Formatter Truncate" src="https://github.com/user-attachments/assets/104a3e39-e4a2-4c51-8c2f-173e4e618a9e" /> |
| Facebook Pages | ![Facebook step] <img width="960" height="540" alt="Facebook step" src="https://github.com/user-attachments/assets/5f28811b-a29e-4ac3-90e7-2e6d3229cb27" /> |
| LinkedIn | [LinkedIn step] <img width="960" height="540" alt="Linkedin Setup" src="https://github.com/user-attachments/assets/c5366248-5d31-4295-b3c6-6b70b8add7ed" /> |
| WhatsApp (GreenAPI webhook) | [WhatsApp step] <img width="960" height="540" alt="WhatsApp Step" src="https://github.com/user-attachments/assets/79322e0a-fffd-499c-a7f9-af3becc3421b" /> |
| Slack | [Slack step] <img width="960" height="540" alt="Slack Step" src="https://github.com/user-attachments/assets/79a87b4a-0db0-45e7-8d7d-cce08fe5d41d" /> |

**The post, live on every destination:**

| Facebook | LinkedIn |
|---|---|
| [Facebook post live] <img width="960" height="540" alt="Facebook post Live" src="https://github.com/user-attachments/assets/eee96e12-590d-4bb1-8d2b-e4c6738be0ea" /> | [LinkedIn post live] <img width="960" height="540" alt="Linkedin Post Live" src="https://github.com/user-attachments/assets/e166c179-be5e-441e-b5ef-78c126a126a1" /> |

| WhatsApp | Slack |
|---|---|
| [WhatsApp message] <img width="540" height="1218" alt="WhatsApp Phone message" src="https://github.com/user-attachments/assets/9b53688f-faf6-41ce-a41d-f111ab9b3451" /> | [Slack alert] <img width="960" height="540" alt="Slack Post live" src="https://github.com/user-attachments/assets/6f0725cc-5556-4557-a09c-7830d64439a3" /> |

**A successful run in Zap History:**

[Zap History run] <img width="960" height="540" alt="Zapier History" src="https://github.com/user-attachments/assets/eca9e199-d9ba-4310-b09d-f99294ba92bb" />

### Brand assets

Reusable logo, Facebook cover photo and LinkedIn banner used across the Page and Company Page, in [`brand-assets/`] <img width="1128" height="191" alt="03-linkedin-banner" src="https://github.com/user-attachments/assets/3f5e0dce-fd75-406d-a3b0-44223ab377f5" />
<img width="1640" height="624" alt="02-facebook-cover-photo" src="https://github.com/user-attachments/assets/c11ecfb3-189b-4de2-8066-ab0e24f85edb" />
<img width="1080" height="1080" alt="01-facebook-profile-logo" src="https://github.com/user-attachments/assets/6486be94-db60-4224-917e-ee9ddb5b573c" />
.

## Setup

1. **Blog with a public RSS feed** (WordPress, Medium, Substack, etc. all work)
2. **Facebook Page** you have full admin access to (not a Business-Portfolio-owned Page — see note below)
3. **LinkedIn Company Page**
4. **GreenAPI account**, instance authorized via WhatsApp QR scan
5. **Slack workspace** with a channel for alerts (e.g. `#content`)
6. Build the 8 steps above in Zapier, mapping fields as described, then publish the Zap

### Note on Facebook Pages

If a Page is owned by a Meta Business Portfolio, Zapier may fail to retrieve a Page access token even when the connecting account has full access. Either grant the connecting profile **Full Control** on the Page inside the portfolio (Business Settings → Accounts → Pages), or use a standalone Page created directly from a personal profile with no portfolio layer.

## Task usage

Each action step (Facebook, LinkedIn, WhatsApp, Slack) counts as one billed Zapier task per post. Filter and Formatter steps generally don't count against the task quota. Adding more destinations one-by-one scales linearly in cost — past 3–4 channels, routing the fan-out through a dedicated multi-channel publishing tool (e.g. Buffer, Publer, or a small n8n/Make workflow triggered by a single webhook) is usually more efficient than adding native Zapier action steps per network.

## Disclaimer

The demo article used to trigger this automation is educational content about investing basics and does not constitute financial advice.
