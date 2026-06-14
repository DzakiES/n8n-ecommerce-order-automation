# 🛒 n8n E-commerce Order Automation

A fully automated order processing pipeline built with **n8n** and **Google Gemini AI**. When a new order comes in, the system classifies it, generates a personalized confirmation email using AI, logs the order to Google Sheets, and notifies the store owner — all with zero manual work.

---

## 📌 What It Does

When a new order is received via webhook, the workflow:

1. **Receives the order** via a webhook trigger (works with any shop platform)
2. **Formats and classifies** the order using a JavaScript code node:
   - 1–100 items → `STANDARD`
   - 101–500 items → `BULK`
   - 501+ items → `VIP`
3. **AI generates a personalized email** using Google Gemini based on the order category:
   - STANDARD → warm and friendly
   - BULK → friendly + 10% discount offer on next order
   - VIP → sophisticated + priority handling mention
4. **Routes the order** through a Switch node based on category
5. **Emails the customer** a confirmation with AI-written message + order summary
6. **Logs the order** to Google Sheets for tracking
7. **Notifies the store owner** with a clean HTML order summary table

---

## 🔧 Tech Stack

| Tool | Purpose |
|---|---|
| [n8n](https://n8n.io) | Workflow automation |
| Google Gemini (AI Agent node) | Personalized email generation |
| JavaScript (Code node) | Order formatting and classification |
| Gmail | Customer confirmation + owner notification |
| Google Sheets | Order logging and tracking |
| Webhook | Order intake from any platform |

---

## 📂 Files

| File | Description |
|---|---|
| `E-commerce Order Automation.json` | Main n8n workflow — import this into your n8n instance |

---

## 🚀 How to Use

### 1. Import the workflow
- Open your n8n instance
- Go to **Workflows** → click **Import**
- Upload `E-commerce Order Automation.json`

### 2. Set up credentials
You'll need to connect:
- **Google Gemini** — get a free API key at [aistudio.google.com](https://aistudio.google.com)
- **Gmail** — connect your Google account (use an App Password if needed)
- **Google Sheets** — connect your Google account

### 3. Set up the Google Sheet
Create a new Google Sheet called `E-commerce Order Log` with these headers in Row 1:

| A | B | C | D | E | F | G | H | I |
|---|---|---|---|---|---|---|---|---|
| Order ID | Customer Name | Email | Product | Quantity | Order Total | Category | Date | Status |

> The **Status** column is automatically set to `New` when an order comes in. Update it manually to mark orders as processed. The daily reminder workflow checks this column every morning at 9AM.

### 4. Activate the webhook
- Open the **Webhook** node
- Copy the **Production URL**
- Use this URL as the endpoint in your shop platform or for testing

### 5. Activate the workflow
- Toggle the workflow to **Active**
- It will now process every incoming order automatically

---

## 🧪 Testing with a Fake Order

Use [Hoppscotch](https://hoppscotch.io) or any API tester to send a POST request to your webhook URL with this sample payload:

```json
{
  "order_id": "ORD-001",
  "customer_name": "John Smith",
  "customer_email": "john@example.com",
  "product": "Wireless Headphones",
  "quantity": 2,
  "price_per_item": 49.99,
  "order_total": 99.98,
  "shipping_address": "123 Main St, New York, USA",
  "order_date": "2026-06-13"
}
```

Change `quantity` to test different categories:
- `50` → STANDARD (1–100 items)
- `500` → BULK (101–500 items)
- `1500` → VIP (501+ items)

---

## 🧠 How the Classification Works

Classification is handled by a simple JavaScript code node — no AI needed for this step, keeping the workflow fast and reliable:

```javascript
if (quantity <= 100) category = "STANDARD";
else if (quantity <= 500) category = "BULK";
else category = "VIP";
```

AI is used where it genuinely adds value — **writing personalized emails** — not for tasks a simple condition handles perfectly.

---

## ⏰ Daily Order Reminder Workflow

A second workflow runs every morning at **9:00 AM**. It checks the Order Log sheet for any orders still marked as `New` and sends the store owner a reminder email listing all unprocessed orders.

**Flow:**
```
Schedule Trigger (9AM) → Google Sheets (get all rows) → Code node (filter "New") → If (any pending?) → Gmail (reminder)
```

| Step | Node | Purpose |
|---|---|---|
| 1 | Schedule Trigger | Fires every day at 9AM |
| 2 | Google Sheets | Fetches all rows from Order Log |
| 3 | Code node | Filters rows where Status = "New" |
| 4 | If node | Checks if any pending orders exist |
| 5 | Gmail | Sends HTML table of unprocessed orders |

If no orders have `New` status, the workflow stops silently — no unnecessary emails.

---

## 📧 Email Examples

**Customer confirmation email** includes:
- AI-written personalized message based on order category
- Full order summary (product, quantity, total, shipping address)

**Owner notification email** includes:
- Clean HTML table with all order details
- Subject line includes order category and ID for quick scanning

**Daily reminder email** includes:
- List of all orders still marked as `New`
- Clean HTML table with order ID, customer, product, quantity, total, and date
- Only sent if there are actually unprocessed orders

---

## 📄 License

MIT License — see [LICENSE](./LICENSE) for details.

---

Built by [Dzaki Endraghani Sunarko](https://github.com/DzakiES)
