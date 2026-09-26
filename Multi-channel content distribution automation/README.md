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
| ![Zap editor, steps 1–4] <img width="960" height="540" alt="Page 1 Zap" src="https://github.com/user-attachments/assets/0831ac17-fbfe-4a76-8d90-1743b24da7fa" /> | ![Zap editor, steps 5–8](screenshots/01b-zap-editor-full-steps5-8.png) |

**Each step's configuration:**

| Step | Screenshot |
|------|------------|
| RSS trigger | ![RSS trigger](screenshots/02a-rss-trigger.png) |
| Filter conditions | ![Filter](screenshots/02b-filter.png) |
| Strip HTML | ![Formatter — remove HTML](screenshots/02c-formatter-remove-html.png) |
| Trim to 200 chars | ![Formatter — truncate](screenshots/02d-formatter-truncate.png) |
| Facebook Pages | ![Facebook step](screenshots/02e-facebook-step.png) |
| LinkedIn | ![LinkedIn step](screenshots/02f-linkedin-step.png) |
| WhatsApp (GreenAPI webhook) | ![WhatsApp step](screenshots/02g-whatsapp-step.png) |
| Slack | ![Slack step](screenshots/02h-slack-step.png) |

**The post, live on every destination:**

| Facebook | LinkedIn |
|---|---|
| ![Facebook post live](screenshots/03-facebook-post-live.png) | ![LinkedIn post live](screenshots/04-linkedin-post-live.png) |

| WhatsApp | Slack |
|---|---|
| ![WhatsApp message](screenshots/05-whatsapp-message-phone.jpeg) | ![Slack alert](screenshots/06-slack-alert.png) |

**A successful run in Zap History:**

![Zap History run](screenshots/07-zap-history-run.png)

### Brand assets

Reusable logo, Facebook cover photo and LinkedIn banner used across the Page and Company Page, in [`brand-assets/`](brand-assets/).

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
