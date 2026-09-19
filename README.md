<div align="center">

# 🏠 SmartHostel Management System

**A comprehensive, full-stack platform to digitize and streamline hostel management operations.**

[![Live Demo](https://img.shields.io/badge/Live_Demo-Visit_App-6366f1?style=for-the-badge)](https://ample-learning-production.up.railway.app)
[![Node.js](https://img.shields.io/badge/Node.js-18+-339933?style=for-the-badge&logo=node.js&logoColor=white)](https://nodejs.org)
[![React](https://img.shields.io/badge/React-18-61DAFB?style=for-the-badge&logo=react&logoColor=black)](https://reactjs.org)
[![TypeScript](https://img.shields.io/badge/TypeScript-5-3178C6?style=for-the-badge&logo=typescript&logoColor=white)](https://www.typescriptlang.org)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)](https://www.postgresql.org)
[![Redis](https://img.shields.io/badge/Redis-7-DC382D?style=for-the-badge&logo=redis&logoColor=white)](https://redis.io)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com)

**Live Demo:** [https://ample-learning-production.up.railway.app](https://ample-learning-production.up.railway.app)

| Role | Email | Password |
|------|-------|----------|
| Student | `student@demo.com` | `Student@123` |
| Committee | `committee@demo.com` | `Committee@123` |
| Admin | `admin@demo.com` | `Admin@123` |

</div>

---

## Table of Contents

- [Features](#features)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [API Reference](#api-reference)
- [Deployment](#deployment)

---

## Features

| Feature | Description |
|---------|-------------|
| **Role-Based Access Control** | Dedicated portals for Admin, Committee members, and Students |
| **QR Attendance Tracking** | Real-time meal attendance via QR code scanning (isolated microservice) |
| **Complaints & Maintenance** | Track, manage, and resolve hostel issues with sentiment analysis |
| **Mess Menu Management** | View and update daily/weekly mess menus with meal schedules |
| **Inventory Management** | Track hostel assets, stock levels, and low-stock alerts |
| **Real-Time Notifications** | Instant alerts via Socket.IO + async email delivery via Redis pub/sub |
| **Analytics & Dashboards** | Visual data representation using Recharts |
| **Google OAuth** | Sign in with Google alongside traditional JWT authentication |
| **Payment Integration** | Razorpay integration for mess fee payments |
| **Email Service** | Transactional emails via SendGrid SMTP (OTP, password reset, notifications) |
| **Token Refresh** | Secure access + refresh token rotation with Redis blacklisting |

---

## Tech Stack

### Frontend (client/)

| Category | Technology |
|----------|-----------|
| Framework | React 18 + TypeScript + Vite |
| Styling | Tailwind CSS v4, Lucide React icons |
| State Management | Zustand (with localStorage persistence) |
| Forms & Validation | React Hook Form + Zod |
| Routing | React Router DOM v6 |
| Real-Time | Socket.IO Client |
| HTTP Client | Axios (with request/response interceptors) |
| Analytics | Recharts |
| Payments | Razorpay Checkout |
| Utilities | clsx, tailwind-merge, react-hot-toast, date-fns |

### Backend (server/)

| Category | Technology |
|----------|-----------|
| Runtime | Node.js + Express 5 + TypeScript |
| Database | PostgreSQL 16 + Prisma ORM v6 |
| Caching & Pub/Sub | Redis 7 (ioRedis) |
| Authentication | JWT (access + refresh tokens), bcryptjs, Google OAuth 2.0 |
| Real-Time | Socket.IO (WebSockets) |
| Email | Nodemailer via SendGrid SMTP |
| Background Jobs | node-cron (scheduled tasks) |
| Security | Helmet, CORS, Express Rate Limit, compression, morgan |
| Validation | Zod |

### Microservices (services/)

#### Attendance Service (services/attendance-service/)

Isolated high-frequency meal/attendance marking with its own failure domain.

- **Stack:** Node.js, Express 5, TypeScript, Prisma v6, ioRedis, JWT
- **RBAC:** Committee / Warden / Admin only; ABAC prevents self-marking
- **Port:** 5002

#### Notification Worker (services/notification-worker/)

Decoupled async email delivery. If this crashes, the main API keeps running.

- **Stack:** Node.js, TypeScript, ioRedis, Nodemailer
- **Pattern:** Subscribes to Redis `notifications:email` pub/sub channel
- **No exposed port** — background worker only, not a HTTP server

### Infrastructure

| Tool | Purpose |
|------|---------|
| Docker + Docker Compose | Full-stack local orchestration |
| PostgreSQL 16-alpine | Primary relational database |
| Redis 7-alpine | Caching, token blacklisting, pub/sub messaging |
| Railway | Cloud deployment (backend + frontend) |
| Nginx | Frontend static file serving in Docker |

---

## Architecture

```
+--------------------------------------------------------------+
|                  ample-learning (Frontend)                    |
|              React 18 + Vite + TypeScript                    |
|       https://ample-learning-production.up.railway.app       |
+----------------------------+---------------------------------+
                             | REST API + WebSocket
                             v
+--------------------------------------------------------------+
|                 SmartHostel (Core API)                        |
|             Node.js + Express 5 + Prisma ORM                 |
|     https://smarthostel-production-2304.up.railway.app       |
+---------+---------------------------+--------------------------+
          |                           |
          v                           v
+------------------+       +-------------------------+
|  PostgreSQL 16   |       |        Redis 7          |
|  (Primary DB)    |       |  Cache + Pub/Sub +      |
|                  |       |  Token Blacklist         |
+------------------+       +-----------+-------------+
                                       | Pub/Sub
                                       v
                           +-------------------------+
                           |   Notification Worker   |
                           |   (Async Email Queue)   |
                           +-------------------------+
```

---

## Project Structure

```
SmartHostel/
├── client/                        # React + Vite frontend
│   ├── src/
│   │   ├── components/            # Reusable UI components
│   │   │   ├── auth/              # ProtectedRoute, etc.
│   │   │   └── layout/            # Navbar, Sidebar
│   │   ├── pages/                 # Route-level page components
│   │   │   ├── auth/              # Login, Register, OAuth callback
│   │   │   ├── student/           # Student dashboard & pages
│   │   │   ├── committee/         # Committee pages
│   │   │   └── admin/             # Admin dashboard & pages
│   │   ├── stores/                # Zustand state stores
│   │   └── lib/                   # Axios API client, socket.io, utils
│   ├── .env.example
│   └── Dockerfile
│
├── server/                        # Node.js + Express core API
│   ├── src/
│   │   ├── controllers/           # Route handlers
│   │   ├── routes/                # API route definitions
│   │   ├── services/              # Business logic layer
│   │   ├── middleware/            # Auth, error handling, validation
│   │   ├── sockets/               # Socket.IO event handlers
│   │   ├── jobs/                  # Background cron jobs
│   │   ├── config/                # DB, Redis, env config
│   │   ├── validators/            # Zod schemas
│   │   ├── utils/                 # Logger, JWT, email, bcrypt
│   │   └── seed.ts                # Database seeder
│   ├── prisma/                    # Prisma schema & migrations
│   ├── .env.example
│   ├── entrypoint.sh              # Docker entrypoint (migrate + seed + start)
│   └── Dockerfile
│
├── services/
│   ├── attendance-service/        # QR attendance microservice (port 5002)
│   └── notification-worker/       # Async email worker (Redis pub/sub)
│
├── docker-compose.yml
└── README.md
```

---

## Getting Started

### Prerequisites

- Node.js v18+
- Docker and Docker Compose
- Git

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/SmartHostel.git
cd SmartHostel
```

### 2. Environment Setup

```bash
# Backend
cp server/.env.example server/.env
# Fill in: DATABASE_URL, JWT secrets, SendGrid key, Razorpay keys, Google OAuth

# Frontend
cp client/.env.example client/.env
# Fill in: VITE_API_URL, VITE_SOCKET_URL
```

---

## Running with Docker (Recommended)

Starts everything — PostgreSQL, Redis, Backend API, Attendance Service, Notification Worker, and Frontend.

```bash
docker-compose up -d --build
```

| Service | URL |
|---------|-----|
| Frontend | http://localhost:3000 |
| Backend API | http://localhost:5001 |
| Attendance Service | http://localhost:5002 |
| Notification Worker | (no port — Redis worker) |

---

## Running Locally (Without Docker)

#### Step 1 — Start databases only
```bash
docker-compose up -d postgres redis
```

#### Step 2 — Backend Server
```bash
cd server
npm install
npm run db:generate
npm run db:migrate
npm run db:seed
npm run dev
```

#### Step 3 — Frontend Client
```bash
cd client
npm install
npm run dev
```

#### Step 4 — Attendance Microservice (optional)
```bash
cd services/attendance-service
npm install
npm run dev
```

#### Step 5 — Notification Worker (optional)
```bash
cd services/notification-worker
npm install
npm run dev
```

---

## Environment Variables

### Backend (server/.env)

```env
NODE_ENV=development
PORT=5000

# Database
DATABASE_URL=postgresql://postgres:postgres123@localhost:5432/hostel_db
DB_CONNECTION_LIMIT=5

# Redis
REDIS_URL=redis://localhost:6379

# JWT
JWT_SECRET=your-32-char-secret-here
JWT_REFRESH_SECRET=your-32-char-refresh-secret
JWT_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d

# Email — SendGrid SMTP
SMTP_HOST=smtp.sendgrid.net
SMTP_PORT=587
SMTP_USER=apikey
SMTP_PASS=your-sendgrid-api-key
FROM_EMAIL=noreply@yourdomain.com

# URLs
APP_URL=http://localhost:5173
CLIENT_URL=http://localhost:3000

# Security
QR_SECRET=your-qr-secret-key

# Rate Limiting
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX=100

# Razorpay
RAZORPAY_KEY_ID=rzp_test_xxxxxxxxxxxx
RAZORPAY_KEY_SECRET=your-razorpay-secret

# Google OAuth — https://console.cloud.google.com/apis/credentials
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret
GOOGLE_CALLBACK_URL=http://localhost:5000/api/v1/auth/google/callback
```

### Frontend (client/.env)

```env
VITE_API_URL=http://localhost:5000/api/v1
VITE_SOCKET_URL=http://localhost:5000
```

---

## API Reference

Base URL: `https://smarthostel-production-2304.up.railway.app/api/v1`

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|:----:|
| GET | /health | Health check | No |
| POST | /auth/register | Register new user | No |
| POST | /auth/login | Login with email/password | No |
| POST | /auth/verify-otp | Verify email OTP | No |
| POST | /auth/resend-otp | Resend OTP | No |
| POST | /auth/refresh | Refresh access token | No |
| POST | /auth/forgot-password | Request password reset | No |
| POST | /auth/reset-password | Reset password | No |
| GET | /auth/google | Initiate Google OAuth | No |
| GET | /auth/profile | Get current user profile | Yes |
| POST | /auth/logout | Logout (blacklist token) | Yes |
| GET | /complaints | List complaints | Yes |
| POST | /complaints | Create complaint | Yes |
| PATCH | /complaints/:id | Update complaint status | Admin |
| GET | /menu | Get mess menu | Yes |
| POST | /menu | Create/update menu | Admin |
| GET | /attendance | Attendance records | Yes |
| GET | /notifications | Get notifications | Yes |
| GET | /rebates | Mess rebate requests | Yes |
| POST | /rebates | Apply for rebate | Student |
| GET | /inventory | Inventory items | Admin |
| GET | /analytics | Dashboard analytics | Admin |
| GET | /workers | Staff management | Admin |

---

## Deployment

Deployed on **Railway** with the following services:

| Railway Service | Role | URL |
|----------------|------|-----|
| SmartHostel | Backend API | smarthostel-production-2304.up.railway.app |
| ample-learning | Frontend | ample-learning-production.up.railway.app |
| Postgres | Database | (internal Railway service) |
| Redis | Cache and Pub/Sub | (internal Railway service) |

### Deploy Your Own Fork

1. Fork this repository
2. Create a new Railway project at https://railway.app
3. Add PostgreSQL and Redis plugins
4. Deploy `server/` as the backend service
5. Deploy `client/` as the frontend service
6. Set all environment variables in each Railway service
7. Add your domains to Google Cloud Console (https://console.cloud.google.com/apis/credentials):
   - Authorized JavaScript Origins: `https://your-frontend.up.railway.app`
   - Authorized Redirect URIs: `https://your-backend.up.railway.app/api/v1/auth/google/callback`

---

Made with ❤️ by Syam Tammireddi
```
