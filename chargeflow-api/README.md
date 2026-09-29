# ChargeFlow API

Express.js REST API and Socket.io Real-time Backend for **ChargeFlow** — EV Charging Station Management Platform.

---

## 🛠️ Features

* **Authentication**: JWT authentication with user, operator, and admin role-based authorization.
* **Database**: MongoDB integration via Mongoose ORM with automatic in-memory MongoDB fallback in development.
* **Stations & Chargers**: Station discovery, spatial geo-queries, status tracking (Available, Occupied, Reserved, Offline).
* **Bookings & Sessions**: Charger reservation system, session start/stop, charging progress tracking.
* **Admin Management**: Operator dashboard endpoints for station/charger management, user blocking, and revenue analytics.
* **Real-time Engine**: Socket.io server for live charger state updates.
* **Vercel Serverless Ready**: Configured with Vercel serverless function handler (`api/index.js`).

---

## 🚀 Quick Start

### 1. Environment Setup
Copy `.env.example` to `.env`:

```bash
cp .env.example .env
```

Environment variables inside `.env`:

```env
NODE_ENV=development
PORT=5050
MONGO_URI=mongodb+srv://<user>:<password>@cluster0.vwrxbbf.mongodb.net/chargeflow?retryWrites=true&w=majority
JWT_SECRET=replace_me_with_a_long_random_string_at_least_32_chars
JWT_EXPIRES_IN=7d
CLIENT_URL=http://localhost:5173
```

*(Note: If `MONGO_URI` is left blank or omitted, the server will boot an in-memory MongoDB instance automatically in development mode).*

### 2. Install Dependencies
```bash
npm install
```

### 3. Seed Database
Populate database with sample stations, chargers, users, and dummy bookings:

```bash
npm run seed
```

### 4. Start Development Server
```bash
npm run dev
```

The API will start listening at **`http://localhost:5050`**.

---

## 📮 API Documentation & Postman Collection

Import the included Postman Collection file located at root:
`../ChargeFlow_Postman_Collection.json`

### Key Endpoints Overview

| Method | Endpoint | Description | Auth Required |
| :--- | :--- | :--- | :--- |
| `POST` | `/api/auth/signup` | Register user or admin (`"role": "admin"`) | No |
| `POST` | `/api/auth/login` | Driver user login | No |
| `POST` | `/api/admin/login` | Operator / Admin login | No |
| `GET` | `/api/auth/me` | Current user profile | Yes (Bearer Token) |
| `GET` | `/api/stations` | List stations with search & filters | No |
| `GET` | `/api/admin/stations` | List stations (Admin view) | Yes (Admin) |
| `POST` | `/api/admin/stations` | Create new charging station | Yes (Admin) |
