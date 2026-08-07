# 🚀 Charge Flow – Complete Project Guide

This document is a deep, implementation-first guide to the `Charge Flow` project as it exists in the current codebase.

---

## 1. 📌 Project Overview

### What is Charge Flow?
Charge Flow is a full-stack EV charging management platform. It helps users discover charging stations, reserve chargers, start/stop charging sessions, and track activity. It also includes an admin interface for station/charger/user/session/revenue management.

### Problem it solves
EV ecosystems usually suffer from:
- poor charger discoverability,
- uncertainty about charger availability,
- slot conflicts and double-booking,
- weak operational visibility for operators.

Charge Flow addresses these by combining:
- map-based discovery,
- booking conflict detection,
- real-time charger/session updates over Socket.io,
- centralized admin controls.

### Target users
- **EV drivers**: discover stations, book slots, manage profile/vehicles, monitor sessions.
- **Admins/operators**: manage stations/chargers/users, view active sessions, monitor alerts and revenue.

### Key features (implemented)
1. **Authentication (user + admin login)**
   - JWT-based auth.
   - Signup/login/reset-password flow.
2. **Station discovery**
   - List stations with filtering/search.
   - Station detail with charger inventory.
3. **Booking flow**
   - Create bookings with overlap conflict detection.
   - Cancel bookings; charger status updates accordingly.
4. **Charging sessions**
   - Start session, stop session, fetch active session.
   - Session completion updates booking + charger states.
5. **Real-time updates**
   - Socket.io events for booking/session/charger status changes.
6. **Admin control panel**
   - CRUD-like station/charger operations.
   - User block/unblock.
   - Revenue summary/timeseries.
   - Alerts from DB + generated operational alerts.

### Real-world use case
An EV driver opens the app, sees nearby stations, checks a charger, books 7:00-8:00 PM, arrives and starts charging. During charging, backend marks charger occupied and sends realtime updates. On stop, session is finalized, cost is calculated, booking is completed, charger becomes available again, and admin dashboards reflect the changes.

---

## 2. 🏗️ Architecture Overview

### System architecture
- **Frontend**: `chargeflow-web` (React + Vite SPA)
- **Backend**: `chargeflow-api` (Express REST + Socket.io)
- **Database**: MongoDB via Mongoose

### Architecture style
- Backend follows a **layered MVC-ish pattern**:
  - `routes` -> `controllers` -> `services` -> `models`
  - `middleware` for cross-cutting concerns.
- Frontend follows **feature/component + context-provider architecture**.
- Not microservices yet; it is a single backend service.

### Request-response flow (typical)
1. User action in React page/component.
2. API wrapper calls axios `request()` from `src/api/client.js`.
3. Backend route validates payload (Joi middleware).
4. Controller delegates to service logic.
5. Service reads/writes MongoDB via Mongoose models.
6. Controller returns JSON envelope.
7. Frontend receives response and updates state/UI.
8. If relevant, backend emits Socket.io events for live updates.

### High-level folder structure
- `chargeflow-api/` - backend service and scripts.
- `chargeflow-web/` - frontend app.
- `archive/` - CSV data source for import script.
- `ChargeFlow-Architecture.md` - target/vision architecture.
- `BACKEND_HANDOFF_FOR_FRONTEND_AI.md` - contract doc.
- `report.md` - prior generated technical report.

---

## 3. 🖥️ Frontend (Client Side)

### Tech stack
- `react`, `react-dom`
- `vite`, `@vitejs/plugin-react`
- `react-router-dom`
- `axios`
- `socket.io-client`
- `tailwindcss`, `postcss`, `autoprefixer`
- `leaflet`, `react-leaflet`
- `framer-motion`
- `lucide-react`

### Frontend folder structure (important parts)
- `src/main.jsx` - app bootstrap + provider composition.
- `src/App.jsx` - root app component.
- `src/routes/` - route map and auth guards.
- `src/context/` - `AuthContext`, `SocketContext`, `ToastContext`.
- `src/api/` - axios client + resource API wrappers.
- `src/pages/` - user and admin screens.
- `src/components/` - reusable UI blocks.
- `src/hooks/` - custom hooks (`useStations`, `useAuth`, etc.).
- `src/services/` - storage, notifications, socket helper.
- `src/styles/index.css` - global Tailwind + custom styles.

### Key components and responsibilities
- `routes/AppRoutes.jsx`: declares all public/user/admin routes.
- `routes/ProtectedRoute.jsx`: route-level auth + role gating.
- `context/AuthContext.jsx`: login/signup/admin login, token/session persistence, user hydration (`/auth/me`), 401 auto-logout.
- `context/SocketContext.jsx`: realtime connection lifecycle, event handlers, toasts/notifications.
- `pages/MapPage.jsx` + `components/map/MapView.jsx`: station discovery UI.
- `pages/BookingPage.jsx`: booking creation and slot conflict UX.
- Admin pages (`pages/Admin*.jsx`): operational dashboards and management views.

### State management approach
- Uses **Context API + local component state** (no Redux).
- Auth is global via `AuthContext`.
- Socket connection and global event listeners via `SocketContext`.
- Notifications via `ToastContext` and `notification.service`.

### Routing system
- Implemented with `react-router-dom`.
- Public routes: login/signup/password/admin login.
- Protected user routes: map, station detail, booking, session, profile, notifications.
- Protected admin routes (role `admin` only): dashboard/stations/chargers/sessions/users/revenue/alerts.

### UI/UX decisions
- Dark premium visual style with green-gold palette.
- Map-focused interaction with stylized markers/popups.
- Responsive layouts with dedicated admin and user navigation.
- Toast feedback for live updates and key actions.

### Animations/libraries
- `framer-motion` is used heavily in landing/hero UI (`HeroSection.jsx`) with stagger/fade/float effects.
- Tailwind classes + custom CSS keyframes for shimmer/loading and marker effects.

### API integration
- Centralized in `src/api/client.js`.
- Base URL from `VITE_API_URL` env.
- Request interceptor injects JWT bearer token and request ID.
- Response interceptor normalizes errors and handles global unauthorized flow.

---

## 4. ⚙️ Backend (Server Side)

### Tech stack
- Node.js (ESM modules)
- Express
- Mongoose
- Socket.io
- Joi
- JWT (`jsonwebtoken`)

### Backend folder structure
- `src/server.js` - process bootstrap (DB + app + socket + shutdown).
- `src/app.js` - express middleware and route mounting.
- `src/config/` - env parsing, db connection, socket singleton.
- `src/routes/` - API route definitions + Joi schemas.
- `src/controllers/` - HTTP handlers (thin).
- `src/services/` - business logic.
- `src/models/` - Mongoose schemas/indexes.
- `src/middleware/` - auth, validation, logger, error handlers.
- `src/sockets/index.js` - socket auth, room wiring, emit helpers.
- `scripts/` - seed/import/smoke utilities.

### Middleware used
- `helmet` - secure headers.
- `cors` - origin allowlist (from `CLIENT_URL` env).
- `compression` - response compression.
- `express-rate-limit` - `/api/auth` protection.
- `morgan` via custom middleware - request logging.
- Joi validation middleware (`validate`).
- Auth middleware (`protect`, `requireRole`).
- Global 404 + error middleware.

### Authentication & authorization
- JWT bearer token (`Authorization: Bearer <token>`).
- Token verification in `protect` middleware.
- Blocked users rejected.
- Role authorization via `requireRole('admin')`.

### Error handling strategy
- Consistent JSON error envelope:
  ```json
  {
    "success": false,
    "error": { "code": "SOME_CODE", "message": "..." }
  }
  ```
- Global handler maps:
  - Joi/Mongoose validation errors -> `400`
  - Invalid ObjectId -> `400`
  - Duplicate keys -> `409`
  - JWT errors -> `401`
  - fallback -> `500`

### Logging
- HTTP logs with `morgan` (`combined` in prod, configurable in dev).
- Service-level logs via `utils/logger.js`.
- 5xx errors logged with stack trace.

---

## 5. 🔌 APIs (VERY IMPORTANT – DO NOT SKIP)

Notes:
- Base prefix: `/api`
- Success envelope typically `{"success": true, "data": ...}`
- Auth required where marked.

### Health

#### `GET /healthz`
- Purpose: liveness check.
- Request body: none.
- Response: `{ ok: true, ts: <number> }`
- Status: `200`

#### `GET /readyz`
- Purpose: readiness check.
- Response: `{ ok: true }`
- Status: `200`

---

### Auth APIs

#### `POST /api/auth/signup`
- Purpose: register user.
- Request body:
  ```json
  { "name": "John", "email": "john@example.com", "password": "secret123", "phone": "+91..." }
  ```
- Response:
  ```json
  { "success": true, "data": { "user": { "id": "...", "name": "John", "email": "john@example.com", "role": "user" }, "token": "jwt..." } }
  ```
- Status codes: `201`, `409` (email taken), `400` validation.

#### `POST /api/auth/login`
- Purpose: login user.
- Request body:
  ```json
  { "email": "john@example.com", "password": "secret123" }
  ```
- Response:
  ```json
  { "success": true, "data": { "user": { "id": "...", "role": "user" }, "token": "jwt..." } }
  ```
- Status codes: `200`, `401` invalid credentials, `403` blocked.

#### `GET /api/auth/me` (auth)
- Purpose: fetch current user profile.
- Response:
  ```json
  { "success": true, "data": { "user": { "id": "...", "email": "...", "vehicles": [] } } }
  ```
- Status: `200`, `401`.

#### `POST /api/auth/me/vehicles` (auth)
- Purpose: append vehicle to user profile.
- Request:
  ```json
  { "make": "Tata", "model": "Nexon EV", "batteryKWh": 40.5, "connectorType": "CCS" }
  ```
- Response includes updated user and newly added vehicle.
- Status: `201`, `400`, `401`.

#### `DELETE /api/auth/me/vehicles/:index` (auth)
- Purpose: remove vehicle by index.
- Response: updated user.
- Status: `200`, `404` if index invalid, `401`.

#### `POST /api/auth/forgot-password`
- Purpose: begin reset flow.
- Request: `{ "email": "john@example.com" }`
- Response:
  ```json
  { "success": true, "data": { "ok": true, "resetToken": "dev-only-in-non-prod" }, "message": "If that email is registered, a reset link has been sent." }
  ```
- Status: `200`.

#### `POST /api/auth/reset-password`
- Purpose: complete reset.
- Request: `{ "token": "resetToken", "newPassword": "newSecret123" }`
- Response: `{ "success": true, "data": { "ok": true }, "message": "Password reset successful. Please sign in." }`
- Status: `200`, `400` invalid/expired token.

---

### Station APIs

#### `GET /api/stations`
- Purpose: list stations (`ACTIVE`) with optional search/city/pagination and charger summary.
- Query: `search`, `city`, `page`, `limit`
- Response:
  ```json
  {
    "success": true,
    "data": [{ "id": "...", "name": "...", "location": { "lat": 28.6, "lng": 77.2 }, "chargers": [] }],
    "total": 42,
    "page": 1
  }
  ```
- Status: `200`.

#### `GET /api/stations/:id`
- Purpose: station details + chargers.
- Response: station object plus `chargers` array.
- Status: `200`, `404`.

#### `GET /api/stations/:id/chargers`
- Purpose: chargers at a station.
- Response: `{"success": true, "data": [ ...charger ] }`
- Status: `200`, `404`.

---

### Charger API

#### `GET /api/chargers/:id`
- Purpose: fetch charger detail (includes station populate).
- Response: charger object.
- Status: `200`, `404`.

---

### Booking APIs (auth)

#### `POST /api/bookings`
- Purpose: create booking with overlap conflict check.
- Request:
  ```json
  { "chargerId": "664...", "startTime": "2026-05-06T13:30:00.000Z", "endTime": "2026-05-06T14:30:00.000Z", "estimatedKWh": 18, "estimatedCost": 360 }
  ```
- Response: created booking.
- Status:
  - `201` success
  - `409 BOOKING_CONFLICT`
  - `400` invalid time/range
  - `404` charger not found

#### `GET /api/bookings/my`
- Purpose: current user bookings.
- Response: array of bookings with populated station/charger fields.
- Status: `200`, `401`.

#### `DELETE /api/bookings/:id`
- Purpose: cancel own booking (or admin cancel).
- Response: updated booking with `status: CANCELLED`.
- Status: `200`, `400` invalid transition, `403`, `404`.

#### `PATCH /api/bookings/:id/cancel`
- Purpose: backward-compatible alias of cancel.
- Behavior/status same as `DELETE`.

---

### Session APIs (auth)

#### `POST /api/sessions/start`
- Purpose: start charging session.
- Request:
  ```json
  { "chargerId": "664...", "bookingId": "665..." }
  ```
- Response: created session.
- Status: `201`, `400`, `403`, `404`, `409`.

#### `POST /api/sessions/:id/stop`
- Purpose: stop active session and finalize cost.
- Request:
  ```json
  { "energyConsumed": 12.4 }
  ```
- Response: completed session.
- Status: `200`, `400`, `403`, `404`.

#### `GET /api/sessions/active`
- Purpose: get current user active session.
- Response: session with populated charger/station.
- Status: `200`, `404` (no active session), `401`.

---

### Admin APIs

#### `POST /api/admin/login`
- Purpose: admin login (must be user role `admin`).
- Body: `{ "email": "...", "password": "..." }`
- Status: `200`, `403` non-admin, `401`.

All routes below require admin token:

#### Stations
- `GET /api/admin/stations` - list stations (`q`, `status` optional)
- `POST /api/admin/stations` - create station
- `PUT /api/admin/stations/:id` - update station
- `DELETE /api/admin/stations/:id` - delete station + chargers

#### Chargers
- `GET /api/admin/chargers` - list chargers (`stationId` optional)
- `PUT /api/admin/chargers/:id` - update charger fields
- `PATCH /api/admin/chargers/:id/toggle` - toggle enabled state

#### Sessions
- `GET /api/admin/sessions` - recent sessions
- `GET /api/admin/sessions/active` - active sessions

#### Users
- `GET /api/admin/users` - list users
- `PATCH /api/admin/users/:id/block` - block/unblock user (non-admin only)

#### Revenue + Alerts
- `GET /api/admin/revenue/summary` - aggregate totals/today/top stations
- `GET /api/admin/revenue/timeseries?days=7` - date bucketed revenue/session counts
- `GET /api/admin/alerts` - persisted and generated alerts

Example summary response:
```json
{
  "success": true,
  "data": {
    "totalRevenue": 12345,
    "totalEnergy": 987,
    "revenueToday": 432,
    "sessionsToday": 12,
    "revenueByStation": [{ "stationId": "...", "stationName": "CP Station", "revenue": 1200, "sessions": 33 }]
  }
}
```

---

## 6. 🗄️ Database Design

### Database used
- MongoDB (Atlas/local), connected via Mongoose.

### Connection setup
- `src/config/env.js` validates `MONGO_URI`.
- `src/config/db.js` runs `mongoose.connect()` with:
  - `autoIndex: !env.isProd`
  - `serverSelectionTimeoutMS: 8000`

### Models and schema details

### 1) `User`
- File: `src/models/User.model.js`
- Fields:
  - `name: String` (required)
  - `email: String` (required, unique, indexed, lowercase)
  - `passwordHash: String` (`select:false`)
  - `role: enum(user|operator|admin)`
  - `isBlocked: Boolean`
  - `phone`, `avatar`
  - reset token fields
  - `vehicles: [{ make, model, batteryKWh, connectorType }]`
- Relationships: referenced by bookings/sessions/stations(operator).
- Indexing: `email` unique/indexed.

### 2) `Station`
- File: `src/models/Station.model.js`
- Fields:
  - metadata (`name`, `description`, `address`)
  - `location` GeoJSON Point `[lng, lat]`
  - `pricingPerKWh`, `status`, `amenities`, etc.
  - `operator` ref `User`
- Relationships: one-to-many with chargers.
- Indexing:
  - `2dsphere` on `location`
  - text index on `name` + `address.city`

### 3) `Charger`
- File: `src/models/Charger.model.js`
- Fields:
  - `ocppId` unique
  - `station` ref
  - `type`, `connectorType`, `powerKW`, `pricePerKWh`
  - `isEnabled`, `status`
  - `currentBooking`, `currentSession` refs
  - `lastHeartbeat`
- Indexes: `ocppId` unique, `station`, `status`.

### 4) `Booking`
- File: `src/models/Booking.model.js`
- Fields:
  - refs: `user`, `charger`, `station`, optional `session`
  - `startTime`, `endTime`
  - estimate fields, `status`, `paymentStatus`, `cancelReason`
- Relationships:
  - many bookings per user/charger/station.
- Indexes:
  - compound `{ charger, startTime, endTime }`
  - `{ user, createdAt: -1 }`
- Validation hook: `endTime > startTime`.

### 5) `ChargingSession`
- File: `src/models/Session.model.js`
- Fields:
  - refs: `booking`, `user`, `charger`, `station`
  - `startTime`, `endTime`, `energyConsumed`, `cost`, `status`
  - `meterReadings[]` and `stopReason`
- Indexes:
  - `{ user, status }`
  - `{ charger, startTime: -1 }`

### 6) `Alert`
- File: `src/models/Alert.model.js`
- Fields: `type`, `message`, `stationId`, `chargerId`, `severity`, `createdAt`.
- Indexes: `{createdAt:-1}`, `{severity:1}`.

### Sample documents
```json
{
  "id": "665...",
  "name": "DLF CyberHub Fast Charge",
  "location": { "lat": 28.495, "lng": 77.089 },
  "status": "ACTIVE",
  "pricingPerKWh": 18
}
```

```json
{
  "id": "666...",
  "user": "663...",
  "charger": "664...",
  "status": "CONFIRMED",
  "startTime": "2026-05-06T13:30:00.000Z",
  "endTime": "2026-05-06T14:30:00.000Z",
  "estimatedKWh": 18,
  "estimatedCost": 360
}
```

---

## 7. 🔗 External Integrations

### Integrated today
1. **MongoDB Atlas / MongoDB**
   - Main persistent database.
2. **OpenStreetMap tile service**
   - Frontend map tiles via Leaflet default tile URL.
3. **Dicebear avatar generation**
   - `auth.service.js` creates avatar URL at signup.

### Mentioned but not fully implemented as runtime integrations
- OCPP gateway as separate service (documented in architecture file).
- Redis adapter/caching.
- Payment gateway.

---

## 8. 📦 Libraries & Dependencies (VERY DETAILED)

### Backend dependencies
- `bcryptjs`: password hashing and comparison (`User` model + auth service).
- `compression`: gzip/compressed responses in `app.js`.
- `cors`: browser cross-origin control in `app.js`.
- `csv-parser`: CSV ingestion in `scripts/import-stations-from-csv.mjs`.
- `dotenv`: load env vars in `config/env.js`.
- `express`: HTTP server framework.
- `express-rate-limit`: throttle auth endpoints.
- `helmet`: security headers.
- `joi`: request/env schema validation.
- `jsonwebtoken`: JWT sign/verify (`utils/jwt.js`).
- `mongodb-memory-server`: optional in-memory fallback logic in `config/db.js`.
- `mongoose`: ODM for MongoDB models/queries/indexes.
- `morgan`: HTTP request logging middleware.
- `socket.io`: realtime server and rooms/events.
- `nodemon` (dev): autorestart during local development.

### Frontend dependencies
- `axios`: HTTP client with interceptors.
- `framer-motion`: animated hero/landing interactions.
- `leaflet`: map rendering engine.
- `lucide-react`: icons across components.
- `react`, `react-dom`: UI runtime.
- `react-leaflet`: React bindings for Leaflet.
- `react-router-dom`: routing and route protection.
- `socket.io-client`: realtime client connection.
- `@vitejs/plugin-react`: Vite React support.
- `tailwindcss`: utility-first styling.
- `postcss`: CSS processing pipeline.
- `autoprefixer`: vendor prefixing.
- `vite`: dev server + bundler.

Why chosen (summary):
- These libraries create a modern React + Node stack with fast iteration (Vite/Nodemon), robust API and schema handling (Axios/Joi/Mongoose), and built-in realtime capability (Socket.io), which directly fits charging availability use cases.

---

## 9. 🔐 Environment Variables

### Backend (`chargeflow-api/.env`)
- `NODE_ENV` - runtime mode (`development|production|test`).
- `PORT` - backend server port.
- `MONGO_URI` - MongoDB connection string.
- `JWT_SECRET` - secret used to sign/verify JWT.
- `JWT_EXPIRES_IN` - JWT TTL (e.g., `7d`).
- `CLIENT_URL` - comma-separated allowed frontend origins for CORS and Socket.io.
- `LOG_LEVEL` - morgan format level in dev.

Example (safe placeholders):
```env
NODE_ENV=production
PORT=5000
MONGO_URI=mongodb+srv://<user>:<encoded-pass>@<cluster>.mongodb.net/chargeflow?retryWrites=true&w=majority
JWT_SECRET=<long-random-secret>
JWT_EXPIRES_IN=7d
CLIENT_URL=https://your-frontend.vercel.app,http://localhost:5173
LOG_LEVEL=combined
```

### Frontend (`chargeflow-web/.env`)
- `VITE_API_URL` - backend API base URL.
- `VITE_SOCKET_URL` - backend socket origin.
- `VITE_MAP_TILE_URL` - tile server URL.

Example:
```env
VITE_API_URL=https://your-backend.onrender.com/api
VITE_SOCKET_URL=https://your-backend.onrender.com
VITE_MAP_TILE_URL=https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png
```

---

## 10. 🔄 Data Flow Walkthrough

Feature walkthrough: **Create a booking**

1. **User action**
   - User selects charger and slot in `BookingPage`.
2. **Frontend API call**
   - `bookingApi.create(payload)` -> `request('post', '/bookings', { data })`.
   - Axios attaches JWT and request ID.
3. **Backend route**
   - `POST /api/bookings` in `booking.routes.js`.
   - `protect` verifies JWT.
   - `validate(createSchema)` validates IDs and date range.
4. **Controller -> Service**
   - `booking.controller.createBooking` calls `bookingService.createBooking`.
5. **Core business logic**
   - Checks date validity and past bookings.
   - Loads charger and verifies status.
   - Runs overlap query:
     - `existing.start < new.end && existing.end > new.start`.
   - Creates booking with `status: CONFIRMED`.
   - If charger was available, marks it `RESERVED`.
6. **Database updates**
   - Booking inserted.
   - Charger updated (status/currentBooking).
7. **Realtime events**
   - `emitBookingCreated` to user room.
   - `emitChargerStatusUpdate` to charger/station rooms.
8. **Response**
   - API responds `201` with booking JSON.
9. **UI update**
   - Frontend renders new booking and receives socket confirmation/toast.

---

## 11. 🧠 Core Logic & Algorithms

### 1) Booking conflict detection
- Core algorithm in `booking.service.js`.
- Uses interval overlap logic with Mongo query.
- Prevents double booking on same charger.

### 2) Charger/session state machine behavior
- Booking creation: `AVAILABLE -> RESERVED` (if applicable).
- Session start: `RESERVED|AVAILABLE -> OCCUPIED`.
- Session stop: `OCCUPIED -> AVAILABLE`.
- Booking status transitions: `CONFIRMED -> IN_PROGRESS -> COMPLETED`, or `CANCELLED`.

### 3) Station list shaping
- `station.controller.listStations` does:
  - filter + pagination,
  - second query for chargers,
  - grouping chargers per station,
  - payload normalization to frontend-friendly shape (`location {lat,lng}`).

### 4) Admin status mapping
- Admin controller maps UI status labels to DB enums and back.
- Keeps frontend simpler while preserving backend domain enums.

---

## 12. 🧪 Testing & Debugging

### Testing tools present
- No formal Jest/Vitest test suite in current repo.
- Available scripts/utilities:
  - `chargeflow-api/scripts/api-smoke.mjs`
  - `chargeflow-api/_smoke.mjs`
  - `chargeflow-api/_http_test.mjs`

### How to debug
- Backend:
  - run `npm run dev` in `chargeflow-api`.
  - inspect request logs and structured error messages.
  - check env validation failures on boot.
- Frontend:
  - run `npm run dev` in `chargeflow-web`.
  - inspect browser console/network tab.
  - verify `VITE_API_URL` and `VITE_SOCKET_URL`.

### Common issues and fixes
1. **CORS blocked**
   - add current Vercel origin to backend `CLIENT_URL`.
2. **Atlas connection failure (`ReplicaSetNoPrimary`)**
   - fix Atlas network allowlist and URI/user credentials.
3. **401 loops**
   - invalid/expired token; AuthContext auto-logout is expected.
4. **Socket not connecting**
   - mismatched socket URL or CORS origin mismatch.

---

## 13. 🚀 Deployment

### Run locally
Backend:
```bash
cd chargeflow-api
cp .env.example .env
npm install
npm run seed   # optional
npm run dev
```

Frontend:
```bash
cd chargeflow-web
cp .env.example .env
npm install
npm run dev
```

### Build steps
- Frontend: `npm run build` -> output `dist`.
- Backend: runtime start with `npm start`.

### Deployment platform fit
- **Backend**: Render (web service).
- **Frontend**: Vercel (Vite app).

Recommended deployment mapping:
- Render root: `chargeflow-api`
- Vercel root: `chargeflow-web`

No directory restructure required.

### CI/CD
- No explicit CI pipeline config in current repository.
- Typical flow is Git push -> platform auto-deploy.

---

## 14. 📁 Complete Folder Structure (IMPORTANT)

Below is the current project tree (excluding `node_modules` and transient files):

```text
chargeflow/
├── BACKEND_HANDOFF_FOR_FRONTEND_AI.md
├── ChargeFlow-Architecture.md
├── report.md
├── guide.md
├── package-lock.json
├── archive/
│   └── ev-charging-stations-india.csv
├── chargeflow-api/
│   ├── README.md
│   ├── package.json
│   ├── package-lock.json
│   ├── yarn.lock
│   ├── _http_test.mjs
│   ├── _smoke.mjs
│   ├── scripts/
│   │   ├── api-smoke.mjs
│   │   ├── seed.js
│   │   └── import-stations-from-csv.mjs
│   └── src/
│       ├── server.js
│       ├── app.js
│       ├── config/
│       │   ├── env.js
│       │   ├── db.js
│       │   └── socket.js
│       ├── routes/
│       │   ├── index.js
│       │   ├── auth.routes.js
│       │   ├── station.routes.js
│       │   ├── charger.routes.js
│       │   ├── booking.routes.js
│       │   ├── session.routes.js
│       │   └── admin.routes.js
│       ├── controllers/
│       │   ├── auth.controller.js
│       │   ├── station.controller.js
│       │   ├── charger.controller.js
│       │   ├── booking.controller.js
│       │   ├── session.controller.js
│       │   └── admin.controller.js
│       ├── services/
│       │   ├── auth.service.js
│       │   ├── booking.service.js
│       │   └── session.service.js
│       ├── models/
│       │   ├── User.model.js
│       │   ├── Station.model.js
│       │   ├── Charger.model.js
│       │   ├── Booking.model.js
│       │   ├── Session.model.js
│       │   └── Alert.model.js
│       ├── middleware/
│       │   ├── auth.middleware.js
│       │   ├── validate.middleware.js
│       │   ├── logger.middleware.js
│       │   └── error.middleware.js
│       ├── sockets/
│       │   └── index.js
│       └── utils/
│           ├── ApiError.js
│           ├── asyncHandler.js
│           ├── jwt.js
│           └── logger.js
└── chargeflow-web/
    ├── README.md
    ├── package.json
    ├── package-lock.json
    ├── index.html
    ├── vite.config.js
    ├── tailwind.config.js
    ├── postcss.config.js
    ├── public/
    │   └── hero-ev.png
    ├── dist2/                      # generated artifact directory currently in repo
    └── src/
        ├── main.jsx
        ├── App.jsx
        ├── api/
        │   ├── client.js
        │   ├── errors.js
        │   ├── station.api.js
        │   ├── booking.api.js
        │   └── admin.api.js
        ├── context/
        │   ├── AuthContext.jsx
        │   ├── SocketContext.jsx
        │   └── ToastContext.jsx
        ├── hooks/
        │   ├── useAuth.js
        │   ├── useSocket.js
        │   ├── useStations.js
        │   ├── useGeolocation.js
        │   └── useToast.js
        ├── layouts/
        │   └── AppLayout.jsx
        ├── routes/
        │   ├── AppRoutes.jsx
        │   └── ProtectedRoute.jsx
        ├── services/
        │   ├── storage.service.js
        │   ├── socket.service.js
        │   └── notification.service.js
        ├── styles/
        │   └── index.css
        ├── utils/
        │   ├── constants.js
        │   ├── formatters.js
        │   └── mockData.js
        ├── components/
        │   ├── common/
        │   ├── layout/
        │   ├── map/
        │   ├── station/
        │   ├── charger/
        │   ├── booking/
        │   ├── landing/
        │   └── admin/
        └── pages/
            ├── LoginPage.jsx
            ├── ForgotPasswordPage.jsx
            ├── ResetPasswordPage.jsx
            ├── LogoutPage.jsx
            ├── MapPage.jsx
            ├── StationDetailPage.jsx
            ├── BookingPage.jsx
            ├── BookingsPage.jsx
            ├── ActiveSessionPage.jsx
            ├── NotificationsPage.jsx
            ├── ProfilePage.jsx
            ├── AdminLoginPage.jsx
            ├── AdminDashboardPage.jsx
            ├── AdminStationsPage.jsx
            ├── AdminChargersPage.jsx
            ├── AdminSessionsPage.jsx
            ├── AdminUsersPage.jsx
            ├── AdminRevenuePage.jsx
            └── AdminAlertsPage.jsx
```

---

## 15. ⚠️ Known Limitations

1. **No `/api/stations/nearby` backend route currently**
   - Frontend API wrapper has `nearby()` method but backend routes include only `/stations`, `/:id`, `/:id/chargers`.
2. **No formal automated test suite**
   - only smoke scripts/manual flow testing.
3. **`mongodb-memory-server` fallback mismatch**
   - DB fallback exists in code, but `MONGO_URI` is required by env validation.
4. **Session socket room auth could be stricter**
   - comment notes ownership checks should be enforced via DB.
5. **`dist2/` artifact committed**
   - generated build assets are in repo (can be cleaned for source-only hygiene).
6. **Mock-mode remnants in frontend**
   - wrappers still contain optional mock behavior paths.
7. **No payment workflow integration yet**
   - payment status fields exist but no gateway integration.

---

## 16. 🔮 Future Improvements

### Feature improvements
- Implement real nearby geo endpoint (`$geoNear`) and full geospatial filtering.
- Add payment integration and booking confirmation/payment states.
- Add live session telemetry UI with `sessionTick` consumption.
- Add operator role UX distinct from admin.

### Scalability improvements
- Introduce Redis for:
  - Socket.io adapter (multi-instance),
  - caching station/charger hot data,
  - distributed rate limiting.
- Split OCPP gateway into dedicated service.
- Add background jobs for booking expiry/alerts/aggregations.

### Quality/performance improvements
- Add Jest/Vitest + integration tests + e2e tests.
- Introduce CI pipeline (lint/test/build/deploy gates).
- Add OpenTelemetry + metrics dashboards.
- Add API docs generation (OpenAPI/Swagger).

---

## 17. 📖 Summary (Simple Explanation)

Charge Flow is a complete EV charging web app:
- The **frontend** lets users find stations on a map, reserve chargers, and track charging.
- The **backend** stores everything in MongoDB, protects APIs with JWT, and enforces booking/session rules.
- **Socket.io** keeps the app live by pushing booking/session/charger updates instantly.
- Admin tools help manage stations, chargers, users, sessions, revenue, and alerts.

In short: it is a practical, modern EV charging management platform with both user and operator workflows, built on React + Express + MongoDB + realtime events.

