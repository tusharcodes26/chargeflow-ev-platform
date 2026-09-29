# ⚡ ChargeFlow — EV Charging Station Management Platform

**ChargeFlow** is a modern, full-stack EV Charging Station Management platform that enables EV drivers to discover charging stations, reserve chargers, manage real-time charging sessions, and track charging history. It also features a comprehensive Operator Admin Dashboard for station, charger, user, session, and revenue analytics.

---

## 🏗️ Repository Architecture

This repository is organized as a monorepo containing two core packages:

* **[`chargeflow-web`](./chargeflow-web)**: React 18 + Vite 5 frontend single-page application (SPA) with interactive Leaflet maps, dynamic filtering, real-time WebSocket updates, and sleek dark UI.
* **[`chargeflow-api`](./chargeflow-api)**: Express.js REST API with MongoDB (Mongoose), Socket.io real-time server, JWT authentication, and Joi schema validation.

---

## 🛠️ Technology Stack

* **Frontend**: React 18, Vite 5, Tailwind CSS 3, Framer Motion, Lucide Icons, Leaflet Maps, Socket.io Client, Axios.
* **Backend**: Node.js, Express.js, MongoDB Atlas (Mongoose ORM) with in-memory MongoDB fallback for local development, Socket.io, JWT Auth, Joi Validation, Helmet, Rate Limiter.
* **Testing & Tools**: Postman API Collection, Vercel Serverless Function entry point, Render Blueprint configuration.

---

## 🔑 Demo Accounts

All pre-seeded demo accounts use the password: **`demo1234`**

| Role | Email | Features |
| :--- | :--- | :--- |
| **Driver / User** | `sahib@chargeflow.dev` | View stations, reserve chargers, vehicle management, session history |
| **Driver / User** | `demo@chargeflow.dev` | Demo user profile & active charging sessions |
| **Admin / Operator** | `admin@chargeflow.dev` | Full management dashboard, station & charger management, analytics |

---

## 🚀 Quick Start Guide (Local Development)

### 1. Prerequisites
* **Node.js v20+** installed on your system.

### 2. Install Dependencies

```bash
# Install backend dependencies
cd chargeflow-api
npm install

# Install frontend dependencies
cd ../chargeflow-web
npm install
```

### 3. Database Seeding

To populate your database (MongoDB Atlas or local in-memory MongoDB) with stations, chargers, users, and dummy bookings:

```bash
cd chargeflow-api
npm run seed
```

---

## 🏃 Running the Application

Open two terminal windows:

### Terminal 1: Start Backend API
```bash
cd chargeflow-api
npm run dev
```
* Backend API server runs on **`http://localhost:5050`**.
* *Note: If `MONGO_URI` is left blank in `.env`, the server automatically spins up an in-memory MongoDB server for local development.*

### Terminal 2: Start Frontend Web App
```bash
cd chargeflow-web
npm run dev
```
* Open your browser and navigate to **`http://localhost:5173`**.

---

## 🌐 Deployment Guide (Vercel)

Both the frontend web app and backend REST API are fully configured for Vercel deployment.

### Step 1: Deploy Backend API (`chargeflow-api`)
1. Go to [Vercel.com](https://vercel.com) ➔ **Add New Project**.
2. Select your repository: **`tusharcodes26/chargeflow-ev-platform`**.
3. Set **Root Directory** to `chargeflow-api`.
4. Set **Framework Preset**: `Other`.
5. Add Environment Variables:
   * `NODE_ENV`: `production`
   * `MONGO_URI`: `mongodb+srv://<user>:<password>@cluster0.vwrxbbf.mongodb.net/chargeflow?retryWrites=true&w=majority`
   * `JWT_SECRET`: `your_random_secret_string`
   * `CLIENT_URL`: `*`
6. Click **Deploy** and copy your live API URL (e.g. `https://chargeflow-api.vercel.app`).

### Step 2: Deploy Frontend Web App (`chargeflow-web`)
1. Click **Add New Project** on Vercel again.
2. Select the repository **`tusharcodes26/chargeflow-ev-platform`**.
3. Set **Root Directory** to `chargeflow-web`.
4. Set **Framework Preset**: `Vite`.
5. Add Environment Variables:
   * `VITE_API_URL`: `https://<YOUR-API-URL>/api`
   * `VITE_SOCKET_URL`: `https://<YOUR-API-URL>`
6. Click **Deploy**.

---

## 📮 Postman API Collection

A ready-to-import Postman Collection is included in the repository:
* File path: **[`ChargeFlow_Postman_Collection.json`](./ChargeFlow_Postman_Collection.json)**

### How to use in Postman:
1. Open Postman ➔ Click **Import**.
2. Drag and drop `ChargeFlow_Postman_Collection.json`.
3. Use pre-configured requests for `Register Admin`, `Admin Login`, `User Login`, `List Stations`, and `Create Station`.

---

## 📜 License

This project is licensed under the MIT License.
