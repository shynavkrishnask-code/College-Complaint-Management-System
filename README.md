# 🏛️ College Complaint Management System (CCMS)

A full-stack campus grievance management platform. Students file and track complaints; admins triage, assign, update, and resolve them — all with a full audit trail.

---

## ✨ Features

- **Student Portal** — Register, file complaints with attachments, track status, view timeline history
- **Admin Console** — View all complaints, assign departments, set priority, change status, post comments, resolve tickets
- **6-Stage Lifecycle** — Submitted → Under Review → Assigned → In Progress → Resolved → Closed
- **File Attachments** — Upload images or PDFs (stored locally by default, Cloudinary-ready)
- **Full Audit Trail** — Every status change and comment is timestamped and attributed
- **No MongoDB Required Locally** — Falls back to an in-memory store automatically

---

## 🛠️ Tech Stack

| Layer      | Technology                                    |
|------------|-----------------------------------------------|
| Frontend   | Next.js (Pages Router), Tailwind CSS, Zustand |
| Backend    | Node.js, Express, Mongoose (MongoDB)          |
| Auth       | JWT + bcryptjs (cost factor 12)               |
| Uploads    | Multer (local disk) → Cloudinary (production) |
| Security   | helmet, cors, express-rate-limit              |

---

## 📁 Project Structure

```
AI Automation Project/
├── client/          # Next.js frontend
│   ├── src/
│   │   ├── pages/   # App pages (dashboard, complaints, admin)
│   │   ├── components/
│   │   ├── store/   # Zustand auth store
│   │   └── services/# Axios API service
│   └── .env.local   # Client environment variables
└── server/          # Express backend
    ├── src/
    │   ├── config/  # DB, env, seed script
    │   ├── models/  # Mongoose schemas
    │   ├── routes/  # Express routers
    │   ├── controllers/
    │   ├── services/
    │   └── middleware/
    ├── uploads/     # Local file attachments (auto-created)
    └── .env         # Server environment variables
```

---

## 🚀 Running Locally

### Prerequisites
- Node.js ≥ 18
- npm ≥ 9
- MongoDB (optional — app works without it using in-memory fallback)

---

### Step 1 — Clone the repository

```bash
git clone <your-repo-url>
cd "AI Automation Project"
```

---

### Step 2 — Configure environment variables

**Server** (`server/.env`) — already pre-filled with defaults, edit if needed:
```env
PORT=5000
MONGO_URI=mongodb://localhost:27017/ccms
JWT_SECRET=ccms_super_secret_jwt_key_2024
CLIENT_URL=http://localhost:3000
CLOUDINARY_URL=       # leave blank to use local file storage
```

**Client** (`client/.env.local`) — already created:
```env
NEXT_PUBLIC_API_URL=http://localhost:5000/api
```

---

### Step 3 — Install dependencies

```bash
# Install server dependencies
cd server
npm install

# Install client dependencies
cd ../client
npm install
```

---

### Step 4 — Start the backend server

```bash
cd server
npm run dev
```

You should see:
```
🚀 CCMS Backend server running on port 5000
📂 Mode: In-Memory Fallback   ← (if MongoDB not running)
[Seed] In-Memory Admin seeded successfully: admin@ccms.edu (password: admin123)
```

> **Health check:** Open [http://localhost:5000/api/health](http://localhost:5000/api/health)

---

### Step 5 — Start the frontend client

Open a **new terminal window**:

```bash
cd client
npm run dev
```

Client runs at [http://localhost:3000](http://localhost:3000)

---

## 🔑 Default Accounts

| Role    | Email              | Password  |
|---------|--------------------|-----------|
| Admin   | admin@ccms.edu     | admin123  |
| Student | Register yourself → [/register](http://localhost:3000/register) | — |

> ⚠️ The default admin account is seeded automatically when the server starts. It is stored in-memory (or MongoDB if connected), so it will be re-seeded on every restart when using the in-memory fallback.

---

## 🗺️ Key Pages

| URL                          | Role    | Description                              |
|------------------------------|---------|------------------------------------------|
| `/`                          | All     | Landing page (redirects if logged in)    |
| `/login`                     | All     | Login (both students and admins)         |
| `/register`                  | Student | Student self-registration                |
| `/dashboard`                 | Student | Complaint stats & recent tickets         |
| `/complaints/new`            | Student | Submit a new complaint                   |
| `/complaints`                | Student | List & filter own complaints             |
| `/complaints/[id]`           | Student | Complaint details + timeline             |
| `/admin/dashboard`           | Admin   | Full queue + stats dashboard             |
| `/admin/complaints/[id]`     | Admin   | Triage, assign, prioritize, resolve      |
| `/settings`                  | All     | Account profile information              |

---

## 🔌 API Endpoints

| Method | Endpoint                              | Auth         | Description                     |
|--------|---------------------------------------|--------------|---------------------------------|
| POST   | `/api/auth/register`                  | Public       | Register student                |
| POST   | `/api/auth/login`                     | Public       | Login (any role)                |
| GET    | `/api/auth/me`                        | JWT          | Get current user profile        |
| POST   | `/api/complaints`                     | Student JWT  | Submit complaint (multipart)    |
| GET    | `/api/complaints`                     | Student JWT  | List own complaints             |
| GET    | `/api/complaints/:id`                 | Student JWT  | Get complaint + timeline        |
| GET    | `/api/admin/complaints`               | Admin JWT    | List all complaints (w/ filters)|
| GET    | `/api/admin/complaints/:id`           | Admin JWT    | Get any complaint + timeline    |
| PUT    | `/api/admin/complaints/:id/assign`    | Admin JWT    | Assign department/staff         |
| PUT    | `/api/admin/complaints/:id/priority`  | Admin JWT    | Set priority                    |
| PUT    | `/api/admin/complaints/:id/status`    | Admin JWT    | Change status                   |
| POST   | `/api/admin/complaints/:id/comment`   | Admin JWT    | Post admin comment              |
| GET    | `/api/admin/stats`                    | Admin JWT    | Get dashboard statistics        |

---

## ☁️ Production Deployment

1. Set `CLOUDINARY_URL` in `server/.env` for file storage
2. Set `MONGO_URI` to your MongoDB Atlas connection string
3. Set `JWT_SECRET` to a long random secret
4. Set `CLIENT_URL` to your deployed frontend URL
5. Deploy server to Railway/Render/Fly.io
6. Deploy client to Vercel (set `NEXT_PUBLIC_API_URL` in Vercel env settings)

---

## 📄 License

MIT
