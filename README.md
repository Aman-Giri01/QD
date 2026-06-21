# Quick Drop — Real-Time Grocery Delivery App

Live Demo: [Quick Drop](https://qd-f.onrender.com/)

Quick Drop is a full-stack grocery delivery app I built from scratch. It handles the complete flow — customers browse and order, admins manage everything from a dashboard, and delivery agents get assigned orders in real time with live GPS tracking.

The part I'm most proud of is the order assignment engine. It doesn't just pick the nearest agent — it factors in distance, acceptance rate, past performance, and workload fairness to score and assign the best available agent every time.

---

## What it does

- Orders get assigned to delivery agents in real time via WebSockets, with a 30-second response window before auto-reassignment
- Customers can track their delivery live on a map (Leaflet + Socket.IO)
- There's a Pantry OS feature — users can scan barcodes to add items, get expiry alerts, and generate AI recipes from whatever's in their pantry using GPT-4o
- Payments work via PayPal and Stripe with automatic INR→USD conversion
- Admins get a full dashboard with order analytics, agent performance, and audit logs
- Push notifications through Firebase, emails via SendGrid, OTP via Twilio

---

## Tech Stack

**Frontend** — React 19, Vite 7, Tailwind CSS 4, Socket.IO Client, Leaflet, Chart.js, Stripe React SDK, @zxing (barcode scanning)

**Backend** — Node.js, Express 5, MongoDB + Mongoose 9, Socket.IO, Redis, Bull (job queues), JWT, bcryptjs, Cloudinary, OpenAI SDK, PayPal REST API, Stripe SDK, SendGrid, Twilio, Firebase Admin

---

## Project Structure

```
QD-try/
├── Backend/
│   ├── server.js
│   └── src/
│       ├── config/          # DB, Redis, Cloudinary, Stripe, Bull setup
│       ├── models/          # 17 Mongoose schemas
│       ├── controllers/     # admin / user / shop / agent
│       ├── routes/          # REST API routes
│       ├── services/        # business logic (12 services)
│       ├── middlewares/     # auth, error handling, rate limiting
│       ├── validators/      # Joi schemas
│       ├── jobs/            # cron jobs (expiry alerts, auto-reorder)
│       └── utils/           # JWT, email, OTP helpers
│
└── frontend/
    └── src/
        ├── context/         # Auth, Cart, Location, Pantry, Theme
        ├── pages/           # 23 pages
        ├── component/       # Auth, Layout, UI, Delivery, Pantry
        └── services/        # 13 API service modules
```

---

## Architecture

```
Browser (React + Socket.IO Client)
        |
        |── HTTP REST ──▶ Express 5 Server
        |── WebSocket ──▶ Socket.IO Server
                              |
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
           MongoDB          Redis        External APIs
                        (cache/locks)   Cloudinary, PayPal,
                                        Stripe, OpenAI,
                                        SendGrid, Twilio,
                                        Firebase
```

---

## Getting Started

You'll need Node.js v18+, MongoDB, and Redis running before starting.

```bash
git clone <repo-url>
cd QD-try

# Backend
cd Backend
npm install
# create .env file (see below)
npm start

# Frontend (new terminal)
cd frontend
npm install
npm run dev
```

Backend runs on port 5000, frontend on port 5173.

---

## Environment Variables

Create a `.env` file inside `Backend/`:

```env
# Database
MONGODB_URI=
REDIS_URL=

# Auth
JWT_SECRET=
JWT_EXPIRE=

# SendGrid
SENDGRID_API_KEY=
SENDGRID_FROM_EMAIL=

# Twilio
TWILIO_ACCOUNT_SID=
TWILIO_AUTH_TOKEN=
TWILIO_PHONE_NUMBER=

# Cloudinary
CLOUDINARY_CLOUD_NAME=
CLOUDINARY_API_KEY=
CLOUDINARY_API_SECRET=

# Payments
PAYPAL_CLIENT_ID=
PAYPAL_SECRET_ID=
PAYPAL_MODE=sandbox
PAYPAL_USD_RATE=83
STRIPE_SECRET_KEY=

# OpenAI
OPENAI_API_KEY=

# Firebase
FIREBASE_SERVICE_ACCOUNT=

# Store location (used for agent assignment radius)
STORE_LAT=
STORE_LON=
ASSIGNMENT_RADIUS_KM=5

# App
PORT=5000
FRONTEND_URL=http://localhost:5173
NODE_ENV=development
```

---

## API Routes

**Public** `/shop/`
- `GET /products` — browse with filters, search, pagination
- `GET /products/:id` — product detail
- `GET /categories` — all categories and subcategories

**User** `/user/`
- Auth (register, login, logout, verify email, forgot/reset password, OTP)
- Profile, Cart, Addresses, Orders, Pantry, AI Recipes, Help Desk, Notifications

**Admin** `/admin/`
- Dashboard analytics, Products, Categories, Orders, Agents, Help Tickets, Audit Logs

**Agent** `/agent/`
- Login, accept/reject orders, update delivery status, live location updates, toggle online/offline

---

## Roles

| Role | What they can do |
|---|---|
| User | Browse products, place orders, track delivery, manage pantry, get AI recipes |
| Admin | Full control — products, orders, agents, users, analytics |
| Delivery Agent | Receive order assignments via WebSocket, update status with live GPS |

---

## Security

- Passwords hashed with bcryptjs (10 salt rounds)
- JWT stored in HTTP-only cookies
- Rate limiting on login and OTP endpoints
- All inputs validated with Joi
- Redis distributed locks to prevent double-assigning agents
- Stock reserved during order processing to prevent overselling
- CORS locked to frontend origin only

---

