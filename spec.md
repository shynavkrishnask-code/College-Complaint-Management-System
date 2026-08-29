Project Overview & Tech Stack

Project Overview

Build a full-stack College Complaint Management System (CCMS) that lets students report problems on campus (classrooms, labs, hostels, Wi-Fi, infrastructure, transport, cleanliness, etc.), routes each complaint to the right department or staff member, and lets both students and admins track it from submission through resolution. The platform replaces a manual, paper/verbal complaint process with a centralized digital tracker: students file and follow complaints; admins triage, assign, comment on, and resolve them; both sides see a shared status trail.

Tech Stack

Frontend: Next.js (Pages Router), React, Tailwind CSS, Zustand, Axios, lucide-react icons.

Backend: Node.js, Express, MongoDB, Mongoose, JSON Web Tokens, bcryptjs, multer (file uploads), helmet, morgan, compression, express-validator, express-rate-limit.

File Storage: Cloudinary (or S3-compatible bucket) for complaint image/file attachments, accessed behind a single uploadService so the provider can be swapped without touching controllers.

Optional AI Layer: Google Generative AI SDK (Gemini) or OpenRouter, used only for bonus features — complaint auto-categorization, AI-generated complaint summaries, duplicate-complaint detection — never required for the core app to function.

Optional Notifications: Nodemailer (email) as a bonus feature for status-change alerts. Real-time in-app updates can use Socket.IO as a stretch goal; the core app must work with plain HTTP polling if that's skipped.

Authentication & Roles

The authentication system must support registration and login for two roles — student and admin — JWT-based session handling, protected routes, a /auth/me profile endpoint, password hashing with bcrypt at cost factor 12, and persistent login state on the client through Zustand. Admin accounts are not self-registerable through the public form; they are seeded or created by an existing admin.

Core Workflow

Student → Submit Complaint → Admin Reviews → Assign Department / Staff → Complaint In Progress → Issue Resolved → Student Views Resolution.

Complaint statuses, in order: Submitted → Under Review → Assigned → In Progress → Resolved → Closed. Only forward transitions are allowed through the UI; an admin can also mark a complaint Closed directly from Resolved after the student confirms, or reopen a Closed complaint back to Under Review if the issue recurs.

Feature Requirements

Must-Have / Core Features

Student registration and login.

Student Dashboard — summary of the student's own complaints by status, with quick links to each.

Complaint Submission — category, description, location on campus, optional image/file attachment.

Complaint Categories — fixed list (Classroom, Laboratory, Hostel, Wi-Fi/Network, Infrastructure, Transportation, Cleanliness, Other), stored as an enum so new categories are a one-line change.

Complaint Status Tracking through the six statuses above, with a timestamped status-change entry stored for each transition.

Complaint History — a student can list and filter their own past complaints.

Complaint Details Page — full description, attachment, current status, department/staff assigned, and every admin comment/update in chronological order.

Admin Dashboard — queue of all complaints, filterable by status/category/priority/department.

Admin Complaint Management — view, triage, and act on any complaint.

Department/Staff Assignment — admin assigns a complaint to a department (and optionally a named staff member).

Admin Comments and Updates — admin can post an update/comment visible to the student, each one tied to a status change or standalone note.

Complaint Status Management — admin moves a complaint through the status pipeline.

Complaint Priority — Low / Medium / High / Critical, settable by admin at triage and editable later.

Resolution Details — a required closing note when a complaint is marked Resolved.

Search and Filter Complaints — by status, category, priority, department, date range, and free-text search on description.

CRUD/API Functionality for complaints, backed by MongoDB.

Frontend–Backend Integration via a single Axios instance with the JWT attached.

Basic Complaint Statistics — counts by status and category, shown on the admin dashboard.

Working deployed application (both client and server reachable over HTTPS).

Bonus / Optional Features (build only after core features are complete and stable)

Email notifications on status change (Nodemailer).

Real-time status notifications (Socket.IO) in place of polling.

Admin analytics dashboard — trends over time, not just current counts.

Department-wise statistics.

Complaint resolution time tracking (median/average time-to-resolve, per category and per department).

Student feedback after resolution — a short rating/comment once a complaint is Resolved or Closed.

Complaint resolution rating (e.g., 1–5 stars).

Duplicate complaint detection — flag likely duplicates by category + location + time window, optionally AI-assisted.

AI-based complaint categorization — suggest a category from the free-text description.

AI-generated complaint summaries — condense a long description for the admin queue view.

Image-based issue classification — classify the attached photo (e.g., "broken furniture", "leak").

Automatic escalation for unresolved complaints — auto-raise priority or notify a higher role if a complaint sits in one status past a configurable SLA window.

Mobile-responsive / PWA interface.

Frontend Pages

The application uses the Next.js Pages Router. The root / page redirects authenticated students to /dashboard, authenticated admins to /admin/dashboard, and unauthenticated users to /login.

/ – Landing page: what the platform is, how the complaint flow works, login/register CTAs.

/login – Email/password login for both students and admins (role is resolved server-side from the account, not chosen on the form).

/register – Student self-registration only.

/dashboard – Student dashboard: complaint counts by status, recent complaints, "New Complaint" CTA.

/complaints/new – Complaint submission form: category, description, location, file attachment.

/complaints – Student's own complaint list/history with status filter and search.

/complaints/[id] – Complaint details page: description, attachment, status timeline, admin comments, (student-only) feedback/rating once resolved.

/admin/dashboard – Admin console: complaint queue, basic statistics (counts by status/category), filter/sort/pagination.

/admin/complaints/[id] – Admin complaint view: assign department/staff, set priority, post comments, change status, enter resolution details.

/settings – Profile info, password change, (admin) role/department details.

Backend Architecture

Routes: HTTP routing, request validation via express-validator, middleware composition (auth, role check, validation, error handler).

Controllers: Request parsing and response shaping only — never talk directly to MongoDB.

Services: Business logic ownership — complaint CRUD, status-transition rules, assignment logic, statistics aggregation, file-upload handling, (bonus) AI categorization and notification dispatch.

Models: Mongoose schemas only.

Config Layer: Centralizes environment variables, MongoDB connection, and (if used) Socket.IO/mail transport setup.

Database Collections

Users: name, email, password (select: false), role: student | admin, department (admin only, optional), createdAt, lastLogin.

Complaints: title, description, category (enum), location, attachmentUrl, submittedBy (ref Users), assignedDepartment, assignedStaff (ref Users, optional), priority: low | medium | high | critical, status: submitted | under_review | assigned | in_progress | resolved | closed, resolutionDetails, createdAt, updatedAt.

ComplaintUpdates: complaintId (ref Complaints), authorId (ref Users), type: status_change | comment, previousStatus, newStatus, message, createdAt — one row per status change or admin comment, giving the full audit trail shown on the details page.

Departments: name, description, staffMembers (ref Users, optional) — can start as a simple hardcoded enum on Complaints and be promoted to its own collection once department-wise stats are needed.

Notifications (bonus): owner (ref Users), complaintId, type, message, isRead, createdAt.

API Endpoints

Auth

POST /api/auth/register – Register a new student account.

POST /api/auth/login – Authenticate user and issue JWT.

GET /api/auth/me – Fetch current user profile.

Complaints (student-facing)

POST /api/complaints – Submit a new complaint (multipart, with optional attachment).

GET /api/complaints – List the current student's own complaints, filter/search/paginate.

GET /api/complaints/:id – Fetch a single complaint with its full update history.

POST /api/complaints/:id/feedback – (bonus) Submit rating/feedback after resolution.

Complaints (admin-facing)

GET /api/admin/complaints – List all complaints, filter by status/category/priority/department, search, paginate.

PUT /api/admin/complaints/:id/assign – Assign a department and/or staff member.

PUT /api/admin/complaints/:id/priority – Set or change priority.

PUT /api/admin/complaints/:id/status – Move to the next status; requires resolutionDetails when moving to resolved.

POST /api/admin/complaints/:id/comment – Add an admin comment/update.

GET /api/admin/stats – Aggregated counts by status/category (and department, once Departments exists).

Notifications (bonus)

GET /api/notifications – List the current user's notifications.

PUT /api/notifications/:id/read – Mark one as read.

Folder Structure

Frontend Structure

client/
└── src/
    ├── components/
    │   ├── ComplaintForm/
    │   ├── ComplaintCard/
    │   ├── ComplaintTimeline/
    │   ├── StatusBadge/
    │   └── ProtectedRoute/
    ├── pages/
    │   ├── _app.js
    │   ├── index.js
    │   ├── login.js
    │   ├── register.js
    │   ├── dashboard.js
    │   ├── settings.js
    │   ├── complaints/
    │   │   ├── index.js
    │   │   ├── new.js
    │   │   └── [id].js
    │   └── admin/
    │       ├── dashboard.js
    │       └── complaints/
    │           └── [id].js
    ├── store/
    │   └── authStore.js
    └── services/
        └── api.js

Backend Structure

server/
└── src/
    ├── config/
    │   ├── env.js
    │   └── db.js
    ├── routes/
    │   ├── authRoutes.js
    │   ├── complaintRoutes.js
    │   ├── adminRoutes.js
    │   └── notificationRoutes.js
    ├── controllers/
    │   ├── authController.js
    │   ├── complaintController.js
    │   ├── adminController.js
    │   └── notificationController.js
    ├── services/
    │   ├── authService.js
    │   ├── complaintService.js
    │   ├── uploadService.js
    │   ├── statsService.js
    │   └── notificationService.js
    ├── models/
    │   ├── User.js
    │   ├── Complaint.js
    │   ├── ComplaintUpdate.js
    │   ├── Department.js
    │   └── Notification.js
    └── middleware/
        ├── auth.js
        ├── requireRole.js
        └── errorHandler.js

Development Phases

Phase 1: Project setup — Next.js, Express, MongoDB connection, JWT auth (register/login/me), Zustand auth store, role-based route protection.

Phase 2: Complaint core — submission form with attachment upload, complaint CRUD, student dashboard and complaint list/detail pages.

Phase 3: Admin core — admin queue, assignment, priority, status transitions, comments, ComplaintUpdates audit trail, resolution details.

Phase 4: Search, filter, pagination, and basic statistics on both student and admin sides.

Phase 5 (bonus): Email/real-time notifications, analytics dashboard, department-wise stats, resolution-time tracking, student feedback/rating.

Phase 6 (bonus): AI-assisted categorization, AI summaries, duplicate detection, auto-escalation, PWA polish.

UI, Security, and Outcome

UI and UX Requirements

Clean, responsive UI with Tailwind; loading states and empty states for lists; a status pipeline shown visually (e.g., a stepper) on the complaint details page; color-coded status and priority badges; an admin queue that's scannable at a glance (status, priority, category, age, assigned-to).

Security Requirements

Hash passwords with bcrypt at cost 12; sign and verify JWTs with JWT_SECRET; set HTTP security headers via helmet; apply CORS limited to CLIENT_URL; rate-limit auth endpoints via express-rate-limit; validate every request body with express-validator; enforce role checks server-side on every admin route (never trust a client-side role flag); validate uploaded file type/size before storage; never expose another student's complaints through the student-facing endpoints.

Final Expected Outcome

A student can log in, file a complaint with a category, description, location, and photo, and track it as it moves from Submitted to Closed. An admin can see every complaint, triage and assign it, leave updates, and resolve it with a closing note. Both sides share one source of truth for status, with a full audit trail of who changed what and when — replacing the manual, ad-hoc complaint process with a centralized system.

AI Coding Agent Implementation Instructions

Build phase by phase in the order above; do not start bonus features before all core (Must-Have) features work end-to-end. Follow the folder structure. Keep controllers thin — no direct Mongoose calls in controllers, that logic belongs in services. Enforce role checks (student vs admin) in middleware, not ad hoc in each controller. Every status change must write a ComplaintUpdate row, never just mutate Complaint.status silently. Treat every secret (JWT_SECRET, MongoDB URI, Cloudinary keys, mail/AI API keys) as process.env, never hardcoded. Report the list of files created or changed at the end of every phase.