# Ledger Assistant — AI Expense Tracker

A full-stack expense tracker with a dashboard, charts, and a chatbot that
answers plain-English questions about your spending.

- **Frontend:** HTML, CSS, JavaScript (Chart.js for the graphs) — no build step
- **Backend:** Python + FastAPI + SQLite (single file database, no setup needed)
- **Accounts:** username/password login, each account has its own private ledger
- **AI Assistant:** a rule-based natural-language engine that reads real
  numbers from your ledger and answers questions like *"How much did I
  spend on food this week?"* or *"Which category do I spend the most?"* —
  works fully offline, no API key required.
- **Charts:** Chart.js is bundled locally in `frontend/vendor/` — no CDN
  or internet connection needed at runtime.

## Project structure

```
expense-tracker/
├── backend/
│   ├── main.py          FastAPI app + all API routes (also serves the frontend)
│   ├── database.py      SQLite schema (users, sessions, transactions)
│   ├── auth.py           Password hashing + token-based sessions
│   ├── chatbot.py        Natural-language question answering engine
│   └── requirements.txt
└── frontend/
    ├── login.html        Sign in / create account page
    ├── index.html        Dashboard layout
    ├── styles.css        Design (navy sidebar / cream dashboard, matches the mock)
    ├── script.js         Charts, table, modal, and chat wiring
    └── vendor/
        └── chart.umd.js  Chart.js, bundled locally
```

## Run it

```bash
cd backend
pip install -r requirements.txt
uvicorn main:app --reload
```

Then open **http://127.0.0.1:8000** in your browser. That's it — one
command runs both the API and the dashboard, because FastAPI serves the
`frontend/` folder directly.

You'll land on a **login page** first. Click "Create account", pick any
username and a password of 6+ characters, and you'll be taken straight to
an empty dashboard, ready for your own transactions — nothing is
pre-filled. Each account's ledger is private to that account. Delete
`backend/ledger.db` any time to wipe all accounts and start fresh.

> **Upgrading from an earlier version of this project?** The database
> schema changed to support accounts. Delete the old `backend/ledger.db`
> before starting the server, or you'll get schema errors.

## Smart entry (rules + receipt OCR)

- **Auto-category:** typing a description in *Add transaction* suggests a
  category from `backend/categorizer.py` (English / Tanglish / Tamil
  keywords). Amounts above ₹1,000 or unmatched text show a review warning.
  Add your own shops/keywords in `RULES`.
- **Scan receipt:** the 📷 button uploads a photo to `/api/receipt/scan`,
  which fills shop, amount, date and category. Nothing is saved until you
  check the form and press Save. Needs `pip install pytesseract pillow`
  plus the Tesseract program (add the `tam` language data for Tamil).

## What you can do in the app

- **Dashboard** — total income, total expenses, balance, and today's
  spend, each with a month-over-month change indicator.
- **Daily Expenses chart** — a day-by-day income vs. expenses bar chart
  covering the current month, from day 1 through today.
- **Category Wise Expenses** — a donut chart + legend broken down by
  category, with percentages and amounts.
- **Ledger table** — recent transactions with a running balance; click
  "View All" (or "My Ledger" in the sidebar) to see everything.
- **Add Expense** — a modal form (from the sidebar or the quick-action
  tile) to record a new debit or credit; the whole dashboard refreshes
  instantly.
- **Export / Import** — downloads your full ledger as a CSV.
- **Logout** — from the bottom of the sidebar; ends your session and
  returns to the login page.
- **AI Assistant** — a chat panel on the right. Try:
  - "How much did I spend on food this week?"
  - "Which category do I spend the most?"
  - "What's my balance?"
  - "Compare my transport spending to last week"
  - "Any tips to save money?"

## How the chatbot understands questions

`backend/chatbot.py` does three things for every message you send:

1. **Extract entities** — looks for a category (food, transport, shopping,
   bills, entertainment, income, others) and a time period (today,
   yesterday, this week, last week, this month, last month, or all time).
2. **Detect intent** — decides if you're asking for a total, the top
   category, your balance, a comparison, or a savings tip.
3. **Answer** — runs the matching SQL query against your real
   transactions and returns a formatted, natural-sounding reply.

Because it's rule-based, it needs no external AI API key and works
completely offline — but the code is structured so you could swap
`chatbot.answer()` for a call to a real LLM later if you want it to
handle more open-ended questions.

## Customizing

- **Change the demo data:** edit `day_plan` in `backend/database.py`,
  delete `ledger.db`, and restart the server.
- **Add a category:** add it to `CATEGORY_COLORS` in `database.py` and to
  `CATEGORY_ALIASES` in `chatbot.py`.
- **Change the name in "Welcome, Santhosh":** edit the `#welcome-heading`
  text in `frontend/index.html`.
