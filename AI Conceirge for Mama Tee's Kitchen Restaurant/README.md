# 🍽️ Mama Tee's Kitchen --- AI Concierge Automation

An n8n automation that receives customer requests from a voice/AI
concierge, extracts the important information, routes the request by
type, stores it in Airtable, and sends a Telegram notification to the
restaurant.

## How the workflow works

``` text
Customer speaks to AI Concierge
            │
            ▼
        Webhook
            │
            ▼
   Extract Customer Data
            │
            ▼
          Switch
     ┌──────┼──────┐
     ▼      ▼      ▼
   Order Reservation Callback
     │      │      │
     ▼      ▼      ▼
  Airtable Airtable Airtable
     │      │      │
     ▼      ▼      ▼
  Telegram Telegram Telegram
```

### 1. Customer request enters the workflow

The workflow starts with an HTTP `POST` Webhook. The incoming message
contains the customer's tool-call arguments, including information such
as:

-   Customer name
-   Phone number
-   Request type
-   Request details

The webhook passes the request to the data extraction step.

### 2. Customer data is extracted and processed

The **Extract Customer Data** Code node normalizes the incoming
information and prepares the fields needed by the restaurant.

It also adds request-specific logic.

#### Orders

For an order, the workflow:

-   Calculates an estimated delivery window of **30--60 minutes**
-   Adds the configured payment information
-   Keeps the customer's order details

#### Reservations

For a reservation, the workflow extracts:

-   Reservation date
-   Reservation time
-   Number of people
-   Required deposit

It also sets the reservation deposit to **N5,000**, which goes toward
the total bill.

#### Callbacks

For callback requests, the workflow:

-   Captures the reason for the callback
-   Assigns a priority based on keywords
-   Classifies the request as Urgent, High, Low, or Normal

The workflow also records the timestamp using the **Africa/Lagos**
timezone.

### 3. The Switch routes the request

The **Switch** node checks `request_type` and sends the request down one
of three branches:

``` text
request_type
    │
    ├── order ───────► Order branch
    │
    ├── reservation ─► Reservation branch
    │
    └── callback ────► Callback branch
```

### 4. Airtable stores the request

Each branch creates a record in the **Mama Tee's Orders & Reservations**
Airtable base.

The stored information includes customer details, request type, details,
timestamps, and the fields relevant to the specific request.

### 5. Telegram sends the restaurant notification

After the Airtable record is created, the workflow sends a formatted
Telegram message.

#### Order notification

``` text
🍽️ NEW ORDER RECEIVED

Customer
Phone
Order Details
Estimated Delivery
Payment
Time Logged
```

#### Reservation notification

``` text
RESERVATION REQUEST

Customer
Phone
Details
Date
Time
Number of People
Deposit Required
Payment
Time Logged
```

#### Callback notification

``` text
📞 NEW CALLBACK REQUEST

Customer
Phone
Reason
Priority
Time Logged
```

This means the restaurant can receive the operational information in
Telegram without manually checking the database.

## Error handling

The workflow also contains an **Error Trigger**.

If an execution fails, the error branch sends a Telegram alert
containing:

-   Workflow name
-   Last node executed
-   Execution time
-   Error message

``` text
Workflow Error
      │
      ▼
 Error Trigger
      │
      ▼
Telegram Alert
```

## Workflow stack

-   **n8n** --- workflow orchestration
-   **Webhook** --- receives structured customer requests
-   **JavaScript** --- data extraction and request-specific logic
-   **Switch** --- routes orders, reservations, and callbacks
-   **Airtable** --- stores customer requests
-   **Telegram** --- sends real-time restaurant notifications
-   **AI / Voice Concierge** --- collects the customer's request before
    sending structured data into the workflow

## Live Demo

🎥 **Loom walkthrough:**  
https://www.loom.com/share/c0797b4e53f642e69abca86c79a09175

The Loom video demonstrates the AI Concierge receiving a customer request and the automation processing the request through the workflow.

### Airtable Backend

The workflow stores orders, reservations, and callback requests in an Airtable base.

The Airtable workspace is access-controlled, so the private invitation URL is intentionally **not included in this public README**. The screenshots above demonstrate the resulting Airtable records.

## Screenshots

### n8n workflow

[n8n workflow] <img width="960" height="540" alt="Screenshot 2026-05-22 143520" src="https://github.com/user-attachments/assets/7866c1bf-3622-4999-afc4-bc36f179a2e3" />


### AI Concierge

[AI Concierge] <img width="960" height="540" alt="Screenshot 2026-05-22 135041" src="https://github.com/user-attachments/assets/dfce03f8-028a-44ff-b2ab-6ead936bbe69" />


### Airtable records

[Airtable records] <img width="960" height="540" alt="Mama Tee&#39;s Kitchen Airtable Database" src="https://github.com/user-attachments/assets/ff19b70d-8f6b-4e62-9b2f-3366b4bf0ab3" />


### Reservation captured in Airtable

[Reservation record] <img width="960" height="540" alt="Mama Tee&#39;s Kitchen Airtable Database 2" src="https://github.com/user-attachments/assets/d0101001-f837-438f-8346-efded254d25b" />


