# ⚡ Eventrix — Full Stack Event Management System

A complete MERN stack application with JWT authentication for managing and discovering events.

## Tech Stack
- **Frontend**: React + Vite + Tailwind CSS + React Router + Axios
- **Backend**: Node.js + Express.js + MongoDB + Mongoose
- **Auth**: JWT + bcryptjs

---

## 🚀 How to Run

### Prerequisites
Make sure you have these installed:
- [Node.js](https://nodejs.org/) (v18 or above)
- [MongoDB](https://www.mongodb.com/try/download/community) (local) OR a [MongoDB Atlas](https://www.mongodb.com/atlas) free cluster

---

### Step 1 — Start MongoDB
If using local MongoDB:
```bash
mongod
```
If using MongoDB Atlas, copy your connection string.

---

### Step 2 — Setup Backend

```bash
cd backend
npm install
```

Edit `.env` if needed (default works with local MongoDB):
```
PORT=5000
MONGO_URI=mongodb://localhost:27017/eventrix
JWT_SECRET=eventrix_super_secret_jwt_key_2024

ADMIN_NAME=Eventrix Admin
ADMIN_EMAIL=admin@eventrix.com
ADMIN_PASSWORD=Admin@1234
```

**Create the admin account (run once):**
```bash
npm run seed:admin
```

Start the backend:
```bash
npm run dev
```

Backend runs at: **http://localhost:5000**

---

### Step 3 — Setup Frontend

Open a NEW terminal:
```bash
cd frontend
npm install
npm run dev
```

Frontend runs at: **http://localhost:3000**

---

### Step 4 — Open the App
Go to **http://localhost:3000** in your browser.

---

## 📁 Project Structure
```
eventrix/
├── backend/
│   ├── controllers/      # Business logic
│   ├── models/           # MongoDB schemas
│   ├── routes/           # API route definitions
│   ├── middleware/        # Auth middleware
│   ├── .env              # Environment variables
│   └── server.js         # Entry point
│
└── frontend/
    └── src/
        ├── components/   # Reusable UI (Navbar, EventCard)
        ├── context/      # Auth context (global state)
        ├── pages/        # Home, Login, Register, Events, Dashboard
        ├── routes/       # ProtectedRoute
        ├── services/     # Axios API calls
        └── App.jsx       # Route definitions
```

---

## ✅ Features
- Register & Login with JWT authentication
- Create, Edit, Delete events (CRUD)
- Browse & filter events by category/status
- Search events by name or location
- Register / Unregister for events
- Personal Dashboard (your events + registered events)
- Capacity tracker with progress bar
- Responsive dark UI

---

## 🔌 API Endpoints

### Auth
| Method | URL | Description |
|--------|-----|-------------|
| POST | /api/auth/register | Register new user |
| POST | /api/auth/login | Login |
| GET | /api/auth/me | Get current user |

### Admin (admin role required)
| Method | URL | Description |
|--------|-----|-------------|
| GET | /api/admin/stats | Platform-wide stats |
| GET | /api/admin/users | All users |
| DELETE | /api/admin/users/:id | Delete user + their events |
| PUT | /api/admin/users/:id/role | Promote/demote user role |
| GET | /api/admin/events | All events |
| DELETE | /api/admin/events/:id | Delete any event |
|--------|-----|-------------|
| GET | /api/events | Get all events |
| GET | /api/events/:id | Get event by ID |
| POST | /api/events | Create event (auth) |
| PUT | /api/events/:id | Update event (auth) |
| DELETE | /api/events/:id | Delete event (auth) |
| POST | /api/events/:id/register | Register for event (auth) |
| DELETE | /api/events/:id/register | Unregister from event (auth) |
| GET | /api/events/my/events | Get my events (auth) |
| GET | /api/events/my/registered | Get my registered events (auth) |
