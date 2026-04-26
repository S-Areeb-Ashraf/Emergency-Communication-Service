# Emergency Rescue Node Communication System

Frontend: Vite + React + TypeScript
Backend: FastAPI + Supabase Postgres

This project keeps the existing emergency user experience (map, geolocation, emergency nodes, AI advisor modal, current visual theme/animations) and adds:

- User session capture with dedupe rules.
- Admin email/password login.
- Admin analytics dashboard.

## 1) Prerequisites

- Node.js 20+
- npm 10+
- Python 3.10+
- Supabase project (Postgres)

## 2) Database Setup (Supabase)

1. Open Supabase SQL Editor.
2. Copy and run: `database/schema.sql`

This script creates:

- `admin_users` table
- `users` table
- indexes and constraints
- rerunnable seed data

Seeded admin credentials for local testing:

## 3) Backend Setup (FastAPI)

Location: `backend`

1. Create env file:

```bash
cd backend
copy .env.example .env
```

2. Edit `backend/.env` values:

- `DATABASE_URL` = Supabase Postgres URL
- `JWT_SECRET` = long random secret
- `BACKEND_CORS_ORIGINS` = `http://localhost:3001`

3. Create and activate virtual environment:

```bash
python -m venv .venv
.venv\Scripts\activate
```

4. Install dependencies:

```bash
pip install -r requirements.txt
```

5. Run backend:

```bash
uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
```

Health check:

- `http://127.0.0.1:8000/health`

## 4) Frontend Setup

Location: project root

1. Install dependencies:

```bash
npm install
```

2. Configure `.env.local`:

```env
GEMINI_API_KEY=your_gemini_api_key
VITE_API_BASE_URL=http://localhost:8000
```

3. Run frontend:

```bash
npm run dev
```

Expected local URL:

- `http://localhost:3001`

## 5) API Summary

- `POST /api/users/entry`
  - body: `{ "name": "...", "emergency_number": "..." }`
  - `name` required, `emergency_number` optional
  - dedupe rule:
    - same name + same emergency number => no duplicate
    - same name + different emergency number => insert new
    - same name + both empty/null emergency number => no duplicate

- `POST /api/admin/login`
  - body: `{ "email": "...", "password": "..." }`
  - returns bearer token and admin info

- `GET /api/admin/users` (protected)
  - header: `Authorization: Bearer <token>`

- `GET /api/admin/stats` (protected)
  - header: `Authorization: Bearer <token>`

## 6) Run Commands (Exact)

Frontend:

```bash
npm install
npm run dev
```

Backend:

```bash
cd backend
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
```

## 7) Quick Verification Steps

1. Run SQL from `database/schema.sql` in Supabase.
2. Start backend and frontend.
3. User flow test:
   - Login with name only -> should proceed.
   - Login again same name + empty number -> should not create duplicate.
   - Login same name + different number -> should create new row.
4. Admin flow test:
   - Open Admin Portal from login screen.
   - Login with seeded admin credentials.
   - Confirm total users and user table load.
5. Existing emergency flow test:
   - Check map/geolocation/nodes dispatch still works.
   - Check AI advisor modal still returns responses.

## 8) Troubleshooting

- `401 Invalid email or password`:
  - Verify seeded admin exists in `admin_users` and credentials are correct.

- `CORS` errors:
  - Ensure `BACKEND_CORS_ORIGINS` includes `http://localhost:3001`.

- Frontend cannot reach backend:
  - Ensure `VITE_API_BASE_URL=http://localhost:8000`.
  - Ensure backend is running on port 8000.

- AI advisor unavailable:
  - Set valid `GEMINI_API_KEY` in `.env.local`.

- Supabase connection issues:
  - Recheck `DATABASE_URL` in `backend/.env`.
  - Ensure the host matches your exact Supabase project ref: `db.<project-ref>.supabase.co`.
  - If your DB password contains special characters (for example `@`, `#`, `%`), URL-encode them in `DATABASE_URL` (for example `@` becomes `%40`).
