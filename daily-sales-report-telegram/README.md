# Daily Sales Report Bot: Google Sheets to Telegram

An n8n workflow that reads a business's sales from Google Sheets every morning at 7:00 (Lagos time), summarises yesterday, compares it with the day before, breaks it down by branch, and sends the report to Telegram. Every send is logged to a second sheet, and a day with no sales still gets a message instead of silence.

`n8n` · `Google Sheets` · `Telegram Bot API` · `JavaScript`

## Demo

**Videos**

- [Automated Daily Sales Summary for Restaurants](https://www.loom.com/share/17cc03f33dd6474bb78f58f319f2c725)
- [Automated Daily Sales Reports With Telegram Alerts](https://www.loom.com/share/5fde8432512048f7a93fcb8455552efc)

**The workflow**

[Workflow canvas] <img width="960" height="540" alt="Workflow Canvas" src="https://github.com/user-attachments/assets/721443aa-5d94-4772-a53c-5888c49b4f9d" />

**The report as it arrives on a phone**

[Telegram report] <img width="540" height="1218" alt="Telegram Notification for Sales summary and No Sales" src="https://github.com/user-attachments/assets/ff4ca403-6fbc-4a19-a91c-63f0fe7dda93" />

**The report log**

[Report log sheet]<img width="960" height="540" alt="No Sales Record on Sales Sheet" src="https://github.com/user-attachments/assets/6a058df6-45c1-4115-b651-5850336b3cb0" />


### Sample report

```
📊 Daily Sales Report - Saturday 19 September 2026

Revenue: ₦244,200 (▲ +162.9%, day before: ₦92,900)
Sales: 19 (▲ +171.4%, day before: 7)
Average sale: ₦12,853
Best seller: Chicken Burger (52 units)

By branch:
• Ikeja: ₦93,700 (7 sales)
• Victoria Island: ₦79,600 (6 sales)
• Lekki: ₦70,900 (6 sales)
```

The data is a sample sales sheet for a three-branch restaurant business.

## How it works

```
Schedule Trigger (07:00, Africa/Lagos)
  → Get All Sales (Google Sheets)
  → Yesterday Only (Code)
  → Day Before (Code)
  → Any Sales? (IF)
       ├─ true  → Build Message (Set) ─────────┐
       └─ false → No Sales Message (Set) ──────┴→ Send Report (Telegram) → Log the Report (Google Sheets)
```

| Node | Type | What it does |
|---|---|---|
| Schedule Trigger | Schedule | Fires once a day at 7:00 in the workflow timezone (Africa/Lagos). |
| Get All Sales | Google Sheets | Reads every row from the `Sales` tab. |
| Yesterday Only | Code | Keeps yesterday's rows, cleans the numbers, and returns one summary item: sales count, revenue, best seller by units, average sale, and a per-branch breakdown. |
| Day Before | Code | Does the same sums for the day before and adds the percentage change and direction (up, down, flat, new). |
| Any Sales? | IF | Routes on `numberOfSales > 0`. |
| Build Message | Set | Writes the full report text with naira formatting and thousands separators. |
| No Sales Message | Set | Writes a short "no sales were recorded" message. |
| Send Report | Telegram | Sends whichever message the run produced. |
| Log the Report | Google Sheets | Appends a row to the `Report` tab with the date, sales, revenue, exact text sent, and a timestamp. |

## Design decisions

- **One item out of each Code node.** A summary should run every node after it exactly once, so both Code nodes return a single item, even on a zero-sales day. Returning nothing would stop the workflow.
- **Timezone.** The workflow timezone is set explicitly to Africa/Lagos, and both Code nodes compute "yesterday" and "the day before" as `YYYY-MM-DD` strings in Lagos time, so they never depend on the server clock.
- **Cleaning numbers.** Sheet values can arrive as text such as `₦12,500`, and adding text joins strings instead of summing. A `cleanNumber` helper strips `₦`, commas and spaces and converts with `Number()`. Invalid values become 0. The code uses the `Total` column when present and falls back to `Quantity × Unit Price`.
- **No divide-by-zero.** If the day before had no sales, the percentage change is `null` rather than `Infinity`, and the message says "no sales the day before".
- **Best seller means most units.** It is ranked by units sold, not by revenue or price.
- **Every send is logged.** The log row is written after the message is sent, on both branches, so the sheet doubles as proof that the workflow ran each day.

## Sheet format

**`Sales` tab**

| Date | Time | Branch | Item | Quantity | Unit Price | Total | Cashier |
|---|---|---|---|---|---|---|---|
| 2026-09-19 | 09:05 | Ikeja | Chicken Burger | 4 | 3500 | 14000 | Amaka |

`Date` must be text in `YYYY-MM-DD` format. `Total` can be a formula such as `=E2*F2`.

**`Report` tab** (create the header row first)

| Dates | Sales | Revenue | Message | Sent At |
|---|---|---|---|---|

A sample `Sales` sheet is in `sample-data/`.

## Setup

1. **Import the workflow.** In n8n, open the workflow menu, choose *Import from file*, and select `daily-sales-report.json`.
2. **Prepare the sheet.** Create a Google Sheet with the `Sales` and `Report` tabs above, or upload the sample data.
3. **Connect Google Sheets.** Add a Google Sheets credential, then select your spreadsheet in both *Get All Sales* and *Log the Report*.
4. **Create a Telegram bot.** Message `@BotFather`, send `/newbot`, and copy the token into an n8n Telegram credential. Open your bot and press **Start**, otherwise it cannot message you.
5. **Set your chat ID.** Get your numeric chat ID (for example from `@userinfobot`) and put it in *Send Report*.
6. **Set the timezone.** In the workflow settings, choose your timezone. Also change the `TZ` value at the top of both Code nodes if you are not in Lagos.
7. **Test, then publish.** Click *Execute workflow* and check Telegram and the `Report` tab. When the message looks right, publish the workflow.

## Testing the no-sales branch

Rather than waiting for a real zero-sales day, pin this output on the *Day Before* node and run the workflow. Unpin it afterwards.

```json
[
  {
    "date": "2026-09-19",
    "numberOfSales": 0,
    "totalRevenue": 0,
    "bestSellingItem": null,
    "unitsOfBestSellingItem": 0,
    "averageSaleValue": 0,
    "branchBreakdown": {},
    "previous": { "date": "2026-09-18", "sales": 7, "revenue": 92900 },
    "revenueChangePct": -100,
    "salesChangePct": -100,
    "revenueDirection": "down"
  }
]
```

The item should leave *Any Sales?* through the **False** branch, send the short message to Telegram, and append a row with 0 sales and 0 revenue.

## Known limitations and next steps

- Dates in the `Sales` tab must be `YYYY-MM-DD` text, because the filters match that format.
- The report goes to a single chat. A list of recipients would need a loop before the send node.
- A failed run is not reported anywhere yet. An error workflow that alerts on failure would fix this.
- The whole `Sales` tab is read on every run. That is fine for a small sheet, but a larger one would need a filter on the Sheets node.
- The currency symbol is hard-coded to ₦.
- Possible extensions: a WhatsApp version, a weekly summary, and a low-stock or slow-seller alert.

## Repository contents

```
daily-sales-report/
├── README.md
├── daily-sales-report.json        # n8n workflow export
├── sample-data/
│   └── Sales.xlsx                 # sample sales sheet
└── screenshots/
    ├── 01-workflow-canvas.png
    ├── 05-telegram-report.png
    └── 06-reports-log.png
```

## Author

Built by Mbama Chibuzo Jude. [Portfolio](https://judethetaken-portfolio.vercel.app) · [GitHub](https://github.com/Judeintechwastaken) · [LinkedIn](https://www.linkedin.com/in/jude-mbama-md-a87072354/)

