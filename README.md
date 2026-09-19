# 🏠 Iconic Estates India
### *"Where Capital Meets Opportunity"*

A full-stack Indian luxury real-estate platform: a customer-facing property portal, a passwordless customer account area, a role-based admin/CRM back office, a Node.js/Express REST API, a Python FastAPI analytics microservice, and a MySQL database — styled as a premium dark/gold design system.

---

## Table of Contents

- [Features](#features)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [1. Database](#1-database)
  - [2. Node API](#2-node-api)
  - [3. Python analytics service](#3-python-analytics-service)
  - [4. Frontend](#4-frontend)
- [Roles and Access](#roles-and-access)
- [API Overview](#api-overview)
- [Customer Portal (Passwordless Login)](#customer-portal-passwordless-login)
- [Building for Production](#building-for-production)
- [Security Notes](#security-notes)
- [Troubleshooting](#troubleshooting)

---

## Features

### Public site
- **Property listings** — search, filter, and compare Residential, Commercial, Agricultural, and Luxury properties
- **Property detail pages** with image galleries
- **Wishlist** and **Compare** (side-by-side comparison bar)
- **EMI / investment calculator**
- **Market intelligence** page — pricing trends and area insights
- **Experts directory** — builders, civil engineers, interior/exterior designers
- **Lead capture** — contact forms, "Schedule Site Visit", and "Request Callback" inquiries
- **Floating contact widget** with WhatsApp deep links

### Customer Portal
- **Passwordless login via OTP** sent to the mobile number on an existing lead/inquiry
- View your own leads/inquiries and their status
- **In-portal messaging** with the sales team per record
- **Document upload/download** per record (e.g. KYC, agreements)

### Admin / CRM back office
- Role-based dashboard (`super_admin`, `admin`, `agent`)
- Property management (CRUD, media)
- Leads and inquiries pipeline with spam/status tracking
- Builders, civil engineers, interior/exterior designer directories
- CRM: notes, follow-ups, customer detail panel, in-app notifications
- Email/CSV templates for outreach (`json2csv` export)
- Market data management
- User management (super admin only)
- Analytics dashboard (charts via Recharts, backed by the Python service)

### Analytics microservice (Python/FastAPI)
- **ROI calculator** — investment return projections
- **Recommendations** — property suggestions
- **Market intelligence** — aggregated pricing/trend data
- **Ownership journey** — post-purchase milestone tracking

---

## Tech Stack

| Layer | Technology |
| --- | --- |
| Frontend | React 18 (Create React App), React Router 6, Recharts, Axios, react-hot-toast, react-image-gallery |
| Node API | Node.js 18+, Express 4, mysql2, JWT (`jsonwebtoken`), bcryptjs, Multer, Helmet, express-rate-limit, Morgan, Nodemailer |
| Analytics service | Python 3.11+, FastAPI, Uvicorn, Pydantic v2, httpx, NumPy |
| Database | MySQL 8 / MariaDB 10.6+ |
| Auth | JWT — separate secrets/flows for Admin (email+password) and Customer Portal (mobile OTP) |

---

## Architecture

```
                 ┌───────────────────────────────┐
                 │   React SPA (CRA, :3000)      │
                 │   Public site · Admin · Portal │
                 └───────────────┬────────────────┘
                                 │ axios (proxy → :5000)
                 ┌───────────────┴────────────────┐
                 │   Node.js / Express API (:5000) │
                 │   auth, properties, leads,      │
                 │   inquiries, crm, portal, ...    │
                 └───────────────┬────────────────┘
                                 │ SQL (mysql2)
                 ┌───────────────┴────────────────┐
                 │   MySQL — iconic_estates_india  │
                 └───────────────┬────────────────┘
                                 │
                 ┌───────────────┴────────────────┐
                 │  Python FastAPI (:8000)         │
                 │  ROI · Recommendations ·        │
                 │  Market · Ownership Journey      │
                 └──────────────────────────────────┘
```

The frontend talks primarily to the Node API (proxied at `/api` in development,
via `frontend/package.json`'s `proxy` field). The Node API and the FastAPI
service run independently on ports `5000` and `8000` and both read from the
same MySQL database.

---

## Project Structure

```
iconic-estates-india/
├── backend-node/            # Node.js + Express REST API (port 5000)
│   ├── config/db.js         # MySQL connection pool
│   ├── controllers/         # Route handlers (auth, properties, crm, portal, ...)
│   ├── middleware/          # auth.js (admin JWT+RBAC), customerAuth.js (OTP JWT), upload.js
│   ├── routes/              # Express routers, one per resource
│   ├── utils/                # mailer.js (SMTP), notify.js
│   ├── seed/seed.js
│   └── server.js
│
├── backend-python/          # Python FastAPI analytics microservice (port 8000)
│   ├── main.py
│   └── routers/             # roi.py, recommendations.py, market.py, journey.py
│
├── database/
│   ├── schema.sql           # base schema (run first)
│   ├── migration_*.sql      # incremental migrations (CRM, notifications, media, spam status)
│   ├── gen_seed.py          # regenerates seed.sql
│   └── seed.sql             # generated seed data
│
└── frontend/                # React (CRA) — public site + admin + customer portal
    ├── public/images/       # experts/builders/property imagery
    └── src/
        ├── components/      # PropertyCard, FilterPanel, LeadForm, CompareBar, ...
        ├── pages/            # HomePage, PropertiesPage, MarketPage, CalculatorPage, ...
        │   ├── admin/        # AdminDashboard, AdminLeads, AdminProperties, ...
        │   └── portal/       # CustomerLogin, CustomerPortal
        ├── context/AuthContext.js
        ├── services/         # api.js, wishlist.js, compare.js, recentlyViewed.js
        └── config/company.js # centralized contact details
```

---

## Getting Started

### Prerequisites

- **Node.js** 18 or newer
- **Python** 3.11 or newer
- **MySQL** 8.0 (or MariaDB 10.6+)

### 1. Database

```bash
# Start MySQL/MariaDB, then create the schema:
mysql -u root -p < database/schema.sql

# Apply migrations, in order:
mysql -u root -p iconic_estates_india < database/migration_crm.sql
mysql -u root -p iconic_estates_india < database/migration_crm_p2.sql
mysql -u root -p iconic_estates_india < database/migration_crm_p3.sql
mysql -u root -p iconic_estates_india < database/migration_crm_p4.sql
mysql -u root -p iconic_estates_india < database/migration_crm_p5.sql
mysql -u root -p iconic_estates_india < database/migration_notifications.sql
mysql -u root -p iconic_estates_india < database/migration_property_media.sql
mysql -u root -p iconic_estates_india < database/migration_spam_status.sql

# Regenerate and load seed data (24 properties, 6 builders, experts, etc.)
python3 database/gen_seed.py
mysql -u root -p iconic_estates_india < database/seed.sql
```

For production, create a dedicated, least-privilege DB user instead of using
`root`:

```sql
CREATE USER 'iconic_user'@'localhost' IDENTIFIED BY 'YourSecurePassword';
GRANT ALL PRIVILEGES ON iconic_estates_india.* TO 'iconic_user'@'localhost';
FLUSH PRIVILEGES;
```

### 2. Node API

```bash
cd backend-node
cp .env.example .env   # then fill in the values below
npm install
npm run dev             # nodemon, or `npm start` for plain node
```

Fill in `backend-node/.env`:

| Variable | Purpose |
| --- | --- |
| `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME` | MySQL connection |
| `JWT_SECRET` | Signs admin session tokens |
| `JWT_EXPIRES_IN` | Admin token lifetime (e.g. `1d`) |
| `CUSTOMER_JWT_SECRET` | Signs customer-portal OTP session tokens (falls back to `JWT_SECRET` if unset — set a distinct value in production) |
| `ADMIN_EMAIL`, `ADMIN_PASSWORD`, `ADMIN_NAME` | Bootstrap admin account used by `npm run seed` |
| `PORT` | API port (default `5000`) |
| `ALLOWED_ORIGINS` | Comma-separated origins allowed by CORS |
| `UPLOAD_DIR` | Directory for uploaded files (default `uploads/`) |
| `SMTP_HOST`, `SMTP_PORT`, `SMTP_SECURE`, `SMTP_USER`, `SMTP_PASSWORD` | Optional — enables outbound email (OTP delivery, CRM "Send Email"). Without these, OTPs are returned directly in the API response for local testing — see [Customer Portal](#customer-portal-passwordless-login) |

Seed the bootstrap admin account:

```bash
npm run seed
```

Runs on `http://localhost:5000`. Health check: `GET /health`.

### 3. Python analytics service

```bash
cd backend-python
python3 -m venv venv && source venv/bin/activate   # Windows: venv\Scripts\activate
pip install -r requirements.txt
uvicorn main:app --reload --port 8000
```

Runs on `http://localhost:8000`. Health check: `GET /health`. Interactive docs
at `http://localhost:8000/docs` (FastAPI's built-in Swagger UI).

Set `ALLOWED_ORIGINS` in this service's environment (or a `.env` file, if you
add one) if the frontend or Node API run on non-default hosts/ports.

### 4. Frontend

```bash
cd frontend
npm install
npm start
```

Opens `http://localhost:3000`. Requests to `/api/*` are proxied to the Node API
on `http://localhost:5000` via the `proxy` field in `frontend/package.json`.
Calls to the analytics service (port `8000`) are made directly — check
`frontend/src/services/api.js` for the base URL and update it if you run that
service on a different host/port.

---

## Roles and Access

**Admin side** (`backend-node/middleware/auth.js`, email + password, JWT):

| Role | Access |
| --- | --- |
| `super_admin` | Full access, including creating new admin/agent accounts (`POST /api/auth/register`) and user management |
| `admin` | Properties, leads, inquiries, CRM, builders, experts, templates, market data, analytics |
| `agent` | Restricted to the routes each controller explicitly allows for agents |

**Customer side** (`backend-node/middleware/customerAuth.js`, mobile OTP, a
separate JWT): customers can only view and act on the lead/inquiry record tied
to their own verified mobile number — there is no customer role that can see
other customers' data.

---

## API Overview

**Node API** (`http://localhost:5000/api`):

| Route prefix | Responsibilities |
| --- | --- |
| `/auth` | Admin login, registration (super admin only), current user, password change |
| `/properties` | Property listings and detail CRUD |
| `/leads` | Lead pipeline (New → Contacted → Qualified → Closed/Lost) |
| `/inquiries` | Contact/site-visit/callback inquiries, spam status |
| `/builders` | Builder directory |
| `/experts` | Civil engineers, interior/exterior designers |
| `/market` | Market intelligence data |
| `/crm` | Notes, customer detail, CRM actions |
| `/templates` | Outreach templates, CSV export |
| `/notifications` | In-app admin notifications |
| `/portal` | Customer portal — OTP auth, own records, messages, documents |
| `/analytics` | Dashboard metrics for the admin panel |

**Python analytics API** (`http://localhost:8000/api`):

| Route prefix | Responsibilities |
| --- | --- |
| `/roi` | Investment ROI calculations |
| `/recommendations` | Property recommendations |
| `/market` | Market intelligence |
| `/journey` | Ownership journey / milestone tracking |

---

## Customer Portal (Passwordless Login)

The customer portal (`/portal` routes, `frontend/src/pages/portal/`) does not
use passwords. A customer requests an OTP with the mobile number they used on
a lead or inquiry; the system verifies that number is on file, generates a
6-digit code valid for 10 minutes, and emails it to the address associated
with that record if SMTP is configured (`backend-node/utils/mailer.js`).

**If SMTP is not configured**, `request-otp` returns the code directly in the
JSON response (clearly labeled) so the flow still works end-to-end during
local development. **Do not leave SMTP unconfigured in production** — see
[Security Notes](#security-notes).

---

## Building for Production

```bash
cd frontend
npm run build      # outputs to frontend/build/
```

Serve `frontend/build/` as static files behind your web server or CDN, run the
Node API under a process manager (PM2, systemd, Docker), and run the FastAPI
service with a production ASGI setup (e.g. `uvicorn` behind Gunicorn workers,
or Uvicorn directly with `--workers`).

Set `ALLOWED_ORIGINS` on **both** backends to your real production origins —
see [Security Notes](#security-notes) regarding the current CORS configuration
in `backend-node/server.js`.

---

## Security Notes

Please review these before deploying publicly:

- **CORS currently reflects any origin.** `backend-node/server.js` configures
  CORS with `origin: true`, which allows requests from *any* origin (with
  credentials). This is convenient for local development across ports but
  should be replaced with an explicit allowlist built from
  `process.env.ALLOWED_ORIGINS` before deploying.
- **`.env` files are gitignored** (`backend-node/.env`, `*.env`) — verify they
  were never committed in your history, and rotate `JWT_SECRET`,
  `CUSTOMER_JWT_SECRET`, `DB_PASSWORD`, and `ADMIN_PASSWORD` if they ever were.
- **Set a distinct `CUSTOMER_JWT_SECRET`.** It silently falls back to
  `JWT_SECRET` if unset, which means a leaked customer-portal token secret
  would also compromise admin sessions. Set both, and set them differently.
- **OTPs are returned in the API response when SMTP isn't configured.** This
  is intentional for local development (see above) but is a real
  authentication bypass if left enabled in production. Confirm SMTP is
  configured, and consider gating the `dev_otp` field behind
  `NODE_ENV !== 'production'` explicitly if it isn't already.
- **Rate limiting is in place** for the general API (300 req/15 min) and login
  (10 req/10 min) via `express-rate-limit` — review these thresholds against
  your expected traffic before going live.
- **File uploads** are handled by `multer` (`backend-node/middleware/upload.js`)
  and served statically from `/uploads`. Confirm file-type/size validation
  meets your requirements before accepting uploads from the public internet.
- **Bootstrap admin credentials** (`ADMIN_EMAIL` / `ADMIN_PASSWORD` in `.env`)
  are used only by `npm run seed` to create the first `super_admin` account —
  change the password immediately after first login, and remove/rotate the
  `.env` values afterward if you don't plan to reseed.

---

## Troubleshooting

**Frontend can't reach the API.**
Confirm the Node API is running on port `5000` (or update the `proxy` field in
`frontend/package.json`), and that `backend-node/.env` has valid DB credentials.

**Analytics charts on the admin dashboard are empty.**
Confirm the FastAPI service is running on port `8000` and reachable from the
browser — it's called directly by the frontend, not proxied through Node.

**OTP login says "No enquiry found."**
The customer portal only issues OTPs for mobile numbers already present on a
`leads` or `inquiries` record. Submit a contact/site-visit enquiry with that
number first, or add it via the admin panel.

**"Insufficient privileges" when creating a new admin user.**
`POST /api/auth/register` requires the `super_admin` role — `admin` and
`agent` accounts cannot create new users.

**Migrations fail with a missing table/column error.**
Run `database/schema.sql` first, then the `migration_*.sql` files in the order
listed in [Getting Started](#1-database) — several depend on tables created by
earlier migrations.
