
## ⚡ What is Eventrix?

Eventrix is a **Full Stack Web App** where users can create events, browse events, and register for them. It's built using the **MERN stack** — MongoDB, Express, React, Node.js.

---

## 🧱 The Big Picture — How the Two Sides Work

```
BROWSER (React)  ←→  EXPRESS SERVER (Node.js)  ←→  MongoDB Database
   Frontend              Backend (API)                  Data Storage
   Port 3000             Port 5000
```

The frontend never talks to the database directly. It always asks the backend, and the backend talks to MongoDB.

---

## 📁 Folder Structure Explained

```
eventrix/
├── backend/        ← Node.js + Express server
└── frontend/       ← React app (what users see)
```

---

## 🔵 BACKEND — Deep Dive

### `server.js` — The Entry Point

This is the first file that runs. It does 3 things:

1. Creates an Express app
2. Connects to MongoDB
3. Starts listening on port 5000

```js
app.use(cors())         // allows frontend (3000) to talk to backend (5000)
app.use(express.json()) // lets the server read JSON from requests
```

---

### `models/` — The Database Blueprints

Think of models as **table definitions** in a regular database. They describe what data looks like.

**`User.js`** — defines what a user looks like in MongoDB:

```
name, email, password, role (user/admin)
```

It also **hashes the password** automatically before saving using `bcryptjs`. So passwords are never stored as plain text.

**`Event.js`** — defines what an event looks like:

```
title, description, date, time, location,
category, capacity, registeredCount, registeredUsers[], organizer, status
```

`registeredUsers` is an array of User IDs — this is how MongoDB links two collections together (called a **reference**).

---

### `middleware/authMiddleware.js` — The Security Guard

Middleware is code that runs **between** the request arriving and the controller handling it.

```
Request → [authMiddleware checks token] → Controller
```

It reads the `Authorization: Bearer <token>` header, verifies the JWT, and attaches the user to `req.user`. If the token is missing or fake, it stops the request with a 401 error.

---

### `controllers/` — The Business Logic

Controllers are the functions that actually **do the work**.

**`authController.js`** has 3 functions:

- `register` — creates a new user, returns a JWT token
- `login` — checks email + password, returns a JWT token
- `getMe` — returns the currently logged-in user's info

**`eventController.js`** has 8 functions:

- `getAllEvents` — fetches all events (supports search/filter via query params)
- `getEventById` — fetches one event by its ID
- `createEvent` — creates a new event (organizer = logged-in user)
- `updateEvent` — updates event (only the organizer or admin can do this)
- `deleteEvent` — deletes event (only organizer or admin)
- `registerForEvent` — adds user to the event's `registeredUsers` array
- `unregisterFromEvent` — removes user from that array
- `getMyEvents` / `getMyRegisteredEvents` — dashboard data

---

### `routes/` — The URL Map

Routes connect a **URL + HTTP method** to a **controller function**.

```
POST /api/auth/register  →  authController.register
POST /api/auth/login     →  authController.login
GET  /api/events         →  eventController.getAllEvents
POST /api/events         →  [protect middleware] → eventController.createEvent
```

The `protect` middleware is added to routes that require login. Without a valid token, those routes reject you.

---

### `.env` — Secret Config File

```
PORT=5000
MONGO_URI=mongodb://localhost:27017/eventrix
JWT_SECRET=eventrix_super_secret_jwt_key_2024
```

These are sensitive values kept out of your code. `dotenv` loads them as `process.env.PORT` etc.

---

## 🟠 FRONTEND — Deep Dive

### `main.jsx` — The Starting Point

This is the React entry file. It mounts the entire app into the `<div id="root">` in `index.html`.

---

### `context/AuthContext.jsx` — Global Login State

This uses React's **Context API** to share the logged-in user across the whole app without passing props everywhere.

It stores:

- `user` — the logged-in user object
- `login(token, user)` — saves token to localStorage, sets user
- `logout()` — clears everything

Any component can call `useAuth()` to get the current user.

---

### `services/` — API Communication Layer

These files use **Axios** to make HTTP requests to the backend.

`api.js` is the Axios instance. It:

- Sets the base URL to `/api` (proxied to port 5000 via Vite config)
- Automatically attaches the JWT token to every request via an **interceptor**
- Redirects to `/login` if a 401 response comes back

`authService.js` and `eventService.js` are just clean wrappers:

```js
export const createEvent = (data) => api.post('/events', data)
export const loginUser = (data) => api.post('/auth/login', data)
```

---

### `routes/ProtectedRoute.jsx` — Frontend Route Guard

If a user tries to visit `/dashboard` without being logged in, this component redirects them to `/login` automatically.

```jsx
return user ? children : <Navigate to="/login" />
```

---

### `pages/` — The Screens

|Page|What it does|
|---|---|
|`Home.jsx`|Landing page with hero, stats, features|
|`Login.jsx`|Email + password form → calls loginUser → saves token|
|`Register.jsx`|Sign up form → calls registerUser → saves token|
|`Events.jsx`|Lists all events with search + category + status filters|
|`EventDetail.jsx`|Shows one event, register/unregister button, edit/delete for organizer|
|`EventForm.jsx`|Create or Edit event form (same component, detects edit via URL `:id`)|
|`Dashboard.jsx`|Shows your created events + events you registered for, with stats|

---

### `components/` — Reusable UI Pieces

**`Navbar.jsx`** — appears on every page. Shows different links depending on whether you're logged in or not.

**`EventCard.jsx`** — the card shown in the events grid. Displays title, date, location, category badge, capacity progress bar.

---

## 🔄 Complete User Flow

```
1. User visits /            → Sees Home page

2. User clicks "Sign Up"    → Fills Register form
                            → POST /api/auth/register
                            → Backend hashes password, saves to MongoDB
                            → Returns JWT token
                            → Token saved to localStorage
                            → Redirected to /dashboard

3. User visits /events      → GET /api/events
                            → Backend fetches all events from MongoDB
                            → React renders EventCards in a grid

4. User clicks an event     → GET /api/events/:id
                            → Sees full event detail
                            → Clicks "Register for Event"
                            → POST /api/events/:id/register (with JWT token)
                            → Backend adds user to registeredUsers array
                            → registeredCount increases by 1

5. User visits /dashboard   → GET /api/events/my/events
                            → GET /api/events/my/registered
                            → Sees their own events + events they joined

6. User clicks "Create"     → Fills EventForm
                            → POST /api/events (with JWT token)
                            → Backend saves event with organizer = user._id

7. User logs out            → Token removed from localStorage
                            → user state set to null
                            → Redirected to home
```

---

## 🔐 How JWT Authentication Works

```
1. User logs in
2. Backend creates a token: jwt.sign({ id: user._id }, SECRET)
   Token looks like: eyJhbGci...  (3 parts separated by dots)

3. Frontend stores token in localStorage

4. Every API request sends: Authorization: Bearer eyJhbGci...

5. Backend middleware decodes the token,
   finds the user in MongoDB,
   attaches them to req.user

6. Controller can now use req.user to know who's making the request
```

The token expires in **7 days**. No database lookup needed to verify — the secret key is enough.

---

## 🛠️ NPM Packages Used & Why

**Backend:**

|Package|Why|
|---|---|
|`express`|Creates the web server and routes|
|`mongoose`|Talks to MongoDB with schemas|
|`jsonwebtoken`|Creates and verifies JWT tokens|
|`bcryptjs`|Hashes passwords securely|
|`cors`|Allows frontend on port 3000 to call backend on 5000|
|`dotenv`|Loads `.env` variables into `process.env`|
|`nodemon`|Restarts server automatically when you save a file|

**Frontend:**

|Package|Why|
|---|---|
|`react-router-dom`|Client-side page navigation without page reloads|
|`axios`|Makes HTTP requests to the backend|
|`react-toastify`|Shows pop-up notifications (success/error)|
|`tailwindcss`|Utility CSS classes for styling|

---
