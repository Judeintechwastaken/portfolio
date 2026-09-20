# NGX Daily Price Tracker

An n8n automation that scrapes the Nigerian Stock Exchange (NGX) daily equity price list, cleans and structures the data, and logs it to Google Sheets — the first step toward a historical database intended to eventually power AI-driven technical and fundamental stock analysis.

**Version:** v1.0.0
**Status:** 🟢 Live (runs daily, unattended)

---

## What it does

Every trading day, this workflow automatically:

1. Fetches the live NGX daily price list from APT Securities' public pricelist page
2. Extracts the pricelist date directly from the page (not just "today"), so data stays accurate even if the source updates late
3. Parses every listed stock's Open, Close, High, Low, Change, Number of Trades, Volume, and Value
4. Flags stocks with no intraday price movement (`noRange`) — a useful liquidity signal for later analysis
5. Appends one clean row per stock, per day, to a Google Sheet
6. Sends a Telegram alert if anything fails, so a broken run never goes unnoticed

The result: an always-growing, analysis-ready dataset of NGX daily trading activity, with zero manual work after setup.

---

## Why

This is the first stage of a larger project: building a multi-year historical database of NGX trading data to eventually feed into an AI agent capable of running technical and fundamental analysis on Nigerian equities. Getting clean, consistent daily data flowing automatically is the foundation everything else builds on.

---

## Architecture

```
Schedule Trigger (daily, post-market close)
        ↓
HTTP Request → NGX daily pricelist page
        ↓
Set Node → extract pricelist date from page heading
        ↓
HTML Extract → pull each stock's table row
        ↓
Code Node → parse, clean, and structure each row
        ↓
Google Sheets → append one row per stock
```

A separate **Error Workflow** runs alongside it:

```
Error Trigger → fires on any failure in the main workflow
        ↓
Telegram → sends an alert with the failed node and error message
```

---

## Tech stack

- **n8n** — workflow orchestration
- **HTTP Request node** — data retrieval
- **HTML Extract node** — table parsing
- **Code node (JavaScript)** — data cleaning and transformation
- **Google Sheets node** — data storage
- **Telegram node** — failure alerting

---

## Data structure

Each row logged to the sheet contains:

| Field | Description |
|---|---|
| `pricelistDate` | The actual trading date the data is for (pulled from the source page) |
| `ticker` | Stock ticker symbol |
| `open` | Opening price |
| `close` | Closing price |
| `high` | Day's high (defaults to close if the stock had no intraday range) |
| `low` | Day's low (defaults to close if the stock had no intraday range) |
| `noRange` | `TRUE` if the stock showed no intraday price movement that day |
| `change` | Change from previous close |
| `trades` | Number of trades executed |
| `volume` | Shares traded |
| `value` | Naira value of shares traded |

---

## Notes on data handling

Some illiquid stocks report `0.00` for High/Low when no real intraday range occurred. Rather than storing a misleading `0`, the workflow defaults these to the closing price and flags them with `noRange: true` — preserving the signal (this stock didn't move) without polluting the numeric fields with fake zero-prices.

---

## Roadmap

- [ ] Historical backfill workflow (previous years of NGX data via the site's dated archive)
- [ ] Move from Google Sheets to a proper database as the dataset grows
- [ ] Feed accumulated data into an LLM-based analysis agent (technical + fundamental analysis)

---

## Author

Built by [Jude Mbama](https://github.com/Judeintechwastaken) — medical doctor transitioning into AI automation.

- Portfolio: [judethetaken-portfolio.vercel.app](https://judethetaken-portfolio.vercel.app)
- LinkedIn: [Jude Mbama, MD](https://www.linkedin.com/in/jude-mbama-md-a87072354/)
- X: [@IamJudeDr](https://x.com/IamJudeDr)

