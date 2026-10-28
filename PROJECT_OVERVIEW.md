# AI CFO Dashboard: Complete Project Guide (Start to Finish, 360° View)

Repository: https://github.com/atharvanasare16-web/AI-CFO-DASHBOARD (branch `main`, 1 commit: "Initial commit")
Team credit in code: **Smart Solutions Team**
Size: about 13,000 lines (backend about 2,500 Python, frontend about 10,500 HTML/JS/CSS)

---

## 1. The idea in one paragraph

Small and medium Indian businesses often cannot afford a full-time CFO. This project is a web app that plays that role. The owner enters money data (sales, expenses, customers, vendors, invoices, bills). The app shows the business's financial health, calculates GST, checks compliance, flags suspicious transactions with machine learning, and answers questions in plain English using an AI model that reads the business's own numbers.

**Analogy:** Tally or QuickBooks (record keeping), plus a tax assistant (GST and compliance), plus a security guard (fraud detection), plus a financial advisor (AI chat), in one dashboard.

---

## 2. Who it is for

| User | What they get |
|---|---|
| Business owner / founder | One screen showing revenue, expenses, cash flow, profit margin and health |
| Accountant | Invoices, bills, GST summary, transaction records, CSV import |
| Compliance / finance manager | Compliance score, regulatory news, fraud alerts |
| Student / evaluator | A full-stack project combining web, data, ML and LLM |

---

## 3. Architecture

```
┌──────────────────────────────┐
│ BROWSER (frontend/)          │
│ 11 HTML pages + JS + CSS     │
│ Bootstrap, Chart.js          │
└──────────────┬───────────────┘
               │ fetch() over HTTPS, JSON
               │ URL set in frontend/config.js
               ▼
┌──────────────────────────────────────────────┐
│ BACKEND (backend/): FastAPI on Render        │
│ app.py: dashboard, GST, compliance, fraud,   │
│         analytics, reports, AI assistant     │
│ routes/: transactions, customers, vendors,   │
│          invoices, bills (CRUD)              │
│ ml/: fraud detection (Isolation Forest)      │
│ services/: regulatory news fetcher           │
└───────┬───────────────┬───────────────┬──────┘
        ▼               ▼               ▼
  SQL database       Groq LLM        NewsAPI
  (SQLite default,   (llama-3.3-     (GST / RBI / SEBI
  or DATABASE_URL)    70b-versatile)  / CBIC news)
```

**Request flow, e.g. "Add an invoice":**
1. The user fills the form on `invoices.html`. `invoice.js` sends `POST /invoices`.
2. `routes/invoices.py` checks the customer exists, computes `gst_amount = subtotal × rate / 100` and `total = subtotal + gst`.
3. SQLAlchemy saves it to the database and returns the saved invoice as JSON.
4. The page refreshes the table and totals.

---

## 4. Folder-by-folder tour

### Root
- `.gitignore`: files git should skip.

### `backend/`
| File | Purpose |
|---|---|
| `app.py` (about 1,060 lines) | Main FastAPI app: CORS, router registration, dashboard, insights, GST summary, expense breakdown, compliance check, AI assistant, analytics, fraud endpoint, reports, regulatory news, scheduler startup |
| `models.py` | Database connection (`DATABASE_URL`, default `sqlite:///./ai_cfo.db`), `get_db` session helper, `TransactionModel`, `RegulatoryNewsModel` |
| `models_accounting.py` | `CustomerModel`, `VendorModel`, `InvoiceModel`, `BillModel` |
| `routes/transactions.py` | CRUD, multi-delete, CSV upload, stats, categories, plus `/seed` and `/reset` |
| `routes/customers.py`, `vendors.py` | CRUD for contacts, including GSTIN |
| `routes/invoices.py`, `bills.py` | CRUD with automatic GST math and status tracking |
| `ml/fraud_detection.py` | `FraudDetector` class wrapping scikit-learn `IsolationForest` |
| `ml/fraud_model.pkl` | Saved trained model |
| `services/news_service.py` | Calls NewsAPI for Indian regulatory news |
| `requirements.txt` | Dependencies (FastAPI, uvicorn, SQLAlchemy, pandas, scikit-learn, groq, apscheduler, and more) |
| `README.md` | Feature list |

### `frontend/`
| Page | Purpose |
|---|---|
| `index.html` | Main dashboard: KPI cards, revenue chart, cash flow chart, expense pie, AI insights |
| `analytics.html` | Trends and deeper analysis |
| `transactions.html` | Table, add, edit, delete, bulk delete, CSV upload |
| `customers.html`, `vendors.html` | Contact management |
| `invoices.html`, `bills.html` | Money in and money out documents |
| `gst.html` | GST summary (collected vs paid vs payable) |
| `compliance.html` | Compliance score, alerts, regulatory news |
| `reports.html` | Summary reports |
| `assistant.html` | Chat with the AI CFO |
| `script.js` (about 3,200 lines) | Shared logic: charts, dashboard loading, transactions, chat, CSV upload |
| `bills.js`, `invoice.js`, `customers.js`, `vendors.js` | Per-page logic |
| `config.js` | API URL and app/team names |
| `style.css` | Shared styling |

---

## 5. Data model (what is stored)

```
Customer ──< Invoice          Vendor ──< Bill
   │                              │
   └──────< Transaction >─────────┘
            (income needs a customer; expense needs a vendor)

RegulatoryNews (title, category, source, summary, impact, url, date)
```

- **Transaction:** amount, category, description, type (`income` or `expense`), date, customer_id or vendor_id.
- **Categories (fixed list):** Revenue, Operations, Investment, Marketing, Sales, HR, Technology, Finance.
- **Invoice:** number (unique), customer, dates, subtotal, GST rate, GST amount, total, status (default "Draft").
- **Bill:** same shape, linked to a vendor.
- Expenses are stored as negative amounts in the seed data. The stats code uses `abs()`.

---

## 6. Feature walk-through (the full user journey)

**Step 1: Setup.** Add customers and vendors with their GSTIN.
**Step 2: Record activity.** Add transactions manually or import a CSV (`amount, category, description, type` are required).
**Step 3: Bill and get billed.** Create invoices for customers and bills from vendors. GST is calculated automatically.
**Step 4: Watch the dashboard.** `/dashboard` returns total revenue, cash flow, expenses and profit margin. `/dashboard-insights` adds written insights. `/expense-breakdown` feeds the pie chart.
**Step 5: GST.** `/gst/summary` computes output GST (from invoices) minus input GST (from bills) to get GST payable.
**Step 6: Compliance.** `/compliance/check` starts from a score of 100 and subtracts for issues (for example no invoices, fraud findings). It returns alerts. News is refreshed on startup and every 12 hours by APScheduler. `/regulatory-news` returns the latest 10 items.
**Step 7: Fraud detection.** `/fraud/detect` needs at least 5 transactions. It trains an Isolation Forest on (amount, category code) with 10% assumed contamination, flags outliers and returns a score plus suspicious transactions.
**Step 8: Analytics and reports.** `/analytics`, `/analytics/insights`, `/reports/summary`.
**Step 9: Ask the AI CFO.** `POST /ai/assistant` gathers dashboard, transactions, customers, vendors, invoices, bills, GST, fraud, compliance and latest news into one context and sends it with the question to Groq (`llama-3.3-70b-versatile`, temperature 0.7, max 700 tokens). The prompt tells the model to answer like an experienced CFO and adapt its headings to the question. If the LLM call fails, hard-coded fallback answers are returned.

---

## 7. Technology stack and why

| Layer | Choice | Why it fits |
|---|---|---|
| API | FastAPI + uvicorn | Fast, automatic docs at `/docs`, typed request bodies |
| Database | SQLAlchemy + SQLite (Postgres-ready via `DATABASE_URL`) | Zero-setup locally, upgradeable |
| Data | pandas, numpy | CSV parsing, DataFrames for the ML model |
| ML | scikit-learn IsolationForest, joblib | Unsupervised anomaly detection, with no labelled fraud data needed |
| LLM | Groq API | Fast, low-cost inference |
| Scheduler | APScheduler | Periodic news refresh |
| Frontend | HTML/CSS/JS, Bootstrap 5, Chart.js, Font Awesome | No build step, easy to host anywhere |
| Hosting | Render (backend); frontend is static, so it can go on any static host | |

`requirements.txt` also lists `python-jose`, `passlib[bcrypt]`, `alembic` and `psycopg2-binary`. These suggest planned login, migrations and Postgres, but they are not used in the code yet.

---

## 8. How to run it locally

```bash
# Backend
cd backend
python -m venv venv
venv\Scripts\activate            # Windows
pip install -r requirements.txt

# Create backend/.env
GROQ_API_KEY=your_groq_key
NEWS_API_KEY=your_newsapi_key
DATABASE_URL=sqlite:///./ai_cfo.db      # optional

python app.py                    # serves on http://localhost:8000 (docs at /docs)
```

```text
# Frontend
Edit frontend/config.js: set API_BASE_URL to "http://localhost:8000"
Open frontend/index.html (or use a static server, e.g. `python -m http.server 5500`)
```

Optional demo data: open `GET /seed` once. It inserts 3 sample transactions that reference customer 1 and vendor 1, so create those first.
The ML model is saved to a relative path (`ml/fraud_model.pkl`), so run the backend from inside `backend/`.

---

## 9. Strengths

- Broad coverage: accounting, tax, compliance, ML and LLM in one product.
- India-specific (GST, GSTIN, CBIC, RBI, SEBI, ICAI, MCA), a real niche.
- Clean separation of backend routes, models and services, with a separate frontend.
- Deployed and working, not only local.
- The AI assistant uses the user's real data, not generic advice.
- Validation rules protect data quality (income needs a customer, expense needs a vendor, CSV category and type checks).

## 10. Risks and weaknesses

| # | Issue | Impact | Fix |
|---|---|---|---|
| 1 | **No authentication.** An OAuth import exists but is unused | Anyone with the URL can read, change or delete everything | JWT login (libraries already in requirements) |
| 2 | **`GET /reset` deletes all transactions and `GET /seed` inserts data, with no protection** | One link click or crawler visit can wipe data | Remove, or make them admin-only POST |
| 3 | **CORS is `*`** with credentials allowed | Any website can call the API | Restrict to the frontend's domain |
| 4 | **Single shared dataset** (no per-user or per-company separation) | Not usable as multi-tenant SaaS | Add a user or company ID to every table |
| 5 | **SQLite on Render's free tier** | Data can be lost when the instance restarts or redeploys | Hosted Postgres via `DATABASE_URL` |
| 6 | **CSV import ignores customer and vendor IDs** (code is commented out) | Imported rows break the app's own linking rule | Add optional ID columns and validation |
| 7 | **Fraud model is basic**: two features, hard-coded 90/10 "probability", trained on the user's own (possibly fraudulent) data | Weak signal, misleading percentages | Add features (date, vendor, frequency), use real anomaly scores |
| 8 | **Binary `.pkl` in git** | Version-fragile; unsafe if the file is ever untrusted | Generate at runtime, don't commit |
| 9 | **All financial data sent to a third-party LLM on every chat** | Privacy and cost exposure | Send only summaries, add consent notice |
| 10 | **LLM fallback returns canned numbers** (e.g. "₹2.13 Cr") that are not from the user's data | Can show false facts | Show an honest error instead |
| 11 | **Very large files** (`app.py` 1,060 lines, `script.js` 3,200 lines) and repeated CRUD code | Hard to maintain | Split into modules, shared helpers |
| 12 | **No tests, no CI, no root README** | Regressions go unnoticed | pytest for routes and GST math, GitHub Actions |
| 13 | `models` use a deprecated `declarative_base` import and `@app.on_event`. News refresh runs on every startup | Warnings, slow cold starts | Update to current FastAPI and SQLAlchemy patterns |

---

## 11. Roadmap (suggested order)

1. **Security first:** login, remove or protect `/reset` and `/seed`, restrict CORS.
2. **Reliability:** Postgres, Alembic migrations, error handling instead of canned fallbacks.
3. **Quality:** unit tests (GST calculation, compliance score), CI, root README with screenshots.
4. **Product:** multi-company support, PDF invoice export, email or WhatsApp reminders for due invoices, GST return (GSTR-1 and 3B) export.
5. **AI upgrade:** better fraud features, cash-flow forecasting, "explain this number" answers with sources, streaming chat.
6. **Maintainability:** split `app.py` and `script.js`, share a single CRUD helper.

---

## 12. Glossary

- **GST:** Goods and Services Tax, India's indirect tax.
- **GSTIN:** 15-character GST registration number of a business.
- **Output GST:** GST you collect on sales (invoices).
- **Input GST:** GST you pay on purchases (bills), which can be offset.
- **GST payable:** Output GST minus Input GST.
- **Isolation Forest:** An ML method that finds data points that are "easy to isolate", meaning unusual.
- **LLM:** Large Language Model, the AI that writes the assistant's answers.
- **CRUD:** Create, Read, Update, Delete.

---

## 13. One-line summary

**AI CFO is a FastAPI + vanilla-JS finance platform for Indian businesses that combines bookkeeping, GST, compliance news, ML fraud detection and an LLM assistant. It is feature-rich and live, and its biggest gaps are security (no login), data durability, and tests.**

---
*This guide was written from reading the repository's code (structure, backend routes, models, ML, services, frontend config and pages). The app itself was not run.*
