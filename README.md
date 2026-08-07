# ⚡ ChargeFlow

**ChargeFlow** is a full-stack EV Charging Station Management platform that helps users discover charging stations, reserve chargers, start/stop charging sessions in real-time, and track charging analytics. It also includes an admin dashboard for station, charger, user, session, and revenue management.

---

## 🏗️ Repository Architecture

This repository is organized as a monorepo containing two main packages:

* **[chargeflow-api](./chargeflow-api)**: Express.js backend REST API, MongoDB/Mongoose database integration, and Socket.io realtime servers.
* **[chargeflow-web](./chargeflow-web)**: React + Vite frontend SPA with Leaflet maps and interactive dashboard styles.

---

## 🛠️ Technology Stack

* **Frontend**: React 18, Vite 5, Tailwind CSS 3, React Router 6, Axios, Leaflet maps, Socket.io client.
* **Backend**: Node.js, Express.js, MongoDB (via Mongoose), Socket.io, JWT authentication, Joi schema validation.
* **Database**: MongoDB (Atlas cloud cluster or in-memory MongoDB fallback in development).

---

## 🚀 Quick Start Guide

To run this project locally, execute the following steps:

### 1. Prerequisite
Ensure you have **Node.js v20+** installed on your system.

### 2. Install Dependencies
Run the install command inside both directories:

```bash
# Install backend dependencies
cd chargeflow-api
npm install

# Install frontend dependencies (in a new terminal or after navigating back)
cd ../chargeflow-web
npm install
```

### 3. Seed the Database
To populate your database with mock stations, chargers, users, and dummy bookings, run the seed command inside the backend folder:

```bash
cd ../chargeflow-api
npm run seed
```

This seeds the system with three default accounts (password for all is `demo1234`):
* **User/Driver**: `sahib@chargeflow.dev`
* **User/Driver**: `demo@chargeflow.dev`
* **Admin/Operator**: `admin@chargeflow.dev`

---

## 🏃 Running the Project

Open two separate terminal windows or split your IDE terminal:

### Terminal 1: Start Backend API
```bash
cd chargeflow-api
npm run dev
```
* The backend API server will run on port `5050` (`http://localhost:5050`).
* Note: If you do not have MongoDB running locally, the server automatically spins up a local **in-memory MongoDB database** in development mode if the `MONGO_URI` variable is omitted or cleared from `.env`.

### Terminal 2: Start Frontend Web App
```bash
cd chargeflow-web
npm run dev
```
* Open your browser and navigate to **`http://localhost:5173`** to access the web app.

---

## 🔒 Configuration & Environment Variables

If you need to customize ports or database settings, modify the `.env` files:

* **[chargeflow-api/.env](./chargeflow-api/.env)**:
  * `PORT=5050`
  * `MONGO_URI` (MongoDB connection string)
  * `JWT_SECRET` (used for JWT signature validation)
* **[chargeflow-web/.env](./chargeflow-web/.env)**:
  * `VITE_API_URL=http://localhost:5050/api`
  * `VITE_SOCKET_URL=http://localhost:5050`
