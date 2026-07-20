# CLAUDE.md

This file gives Claude Code the context it needs to work on this project correctly and consistently. Keep it updated as the project evolves — it is the single source of truth for how Claude should behave in this repo.

## Project Overview

**TaskLog** is a daily activity to-do application. Users log in, track daily tasks, get reminders for pending items, and generate a written "daily update" summary of what they worked on.

Core features:
- User sign-up and login (JWT-based auth)
- Create, update, complete, and delete daily tasks
- Dashboard reminder of pending/overdue tasks for the day
- "Generate Daily Update" — compiles completed + pending tasks into a short written summary the user can copy/share (e.g., for standups)
- Daily activity log/history view (past days' tasks and updates)

Target users: individuals who want a lightweight daily task + standup-note tool. This is a demo/training project, not a production SaaS — favor clarity and correctness over premature optimization.

## Tech Stack

| Layer      | Technology                          |
|------------|--------------------------------------|
| Backend    | FastAPI (Python 3.11+)              |
| Frontend   | React (Vite, JavaScript)                |
| Database   | MySQL 8                             |
| Auth       | JWT (access + refresh tokens)       |
| ORM        | SQLAlchemy 2.x + Alembic (migrations) |
| Containerization | Docker + Docker Compose        |
| Version Control | Git                             |

Frontend and backend are **fully decoupled**: React talks to FastAPI only via REST API calls (no server-side rendering, no shared codebase). Each has its own Dockerfile and can be developed/run independently.

## Repository Structure

```
tasklog/
├── backend/
│   ├── app/
│   │   ├── main.py              # FastAPI app entrypoint
│   │   ├── core/                # config, security, settings
│   │   ├── models/               # SQLAlchemy models
│   │   ├── schemas/              # Pydantic schemas (request/response)
│   │   ├── api/
│   │   │   └── v1/
│   │   │       ├── auth.py
│   │   │       ├── tasks.py
│   │   │       └── updates.py
│   │   ├── services/             # business logic, kept out of route handlers
│   │   ├── db/                   # session, base, migrations helpers
│   │   └── tests/
│   ├── alembic/                  # migration scripts
│   ├── requirements.txt
│   └── Dockerfile
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── api/                  # axios/fetch client, one file per resource
│   │   ├── context/              # auth context, global state
│   │   ├── hooks/
│   │   └── App.jsx
│   ├── package.json
│   └── Dockerfile
├── docker-compose.yml
├── .env.example
└── CLAUDE.md
```

Claude should not restructure these folders without being asked. If a new top-level concern comes up (e.g., background jobs, caching), propose the folder before creating it.

## Development Commands

```bash
# Start everything (backend + frontend + MySQL)
docker compose up --build

# Backend only, inside container
docker compose exec backend bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# Run backend tests
docker compose exec backend pytest

# Create a new migration
docker compose exec backend alembic revision --autogenerate -m "description"
docker compose exec backend alembic upgrade head

# Frontend only, inside container
docker compose exec frontend sh
npm run dev

# Frontend tests
docker compose exec frontend npm run test
```

Always assume the person running these commands is a fresher developer — prefer explicit, readable commands over clever one-liners when suggesting new scripts.

## API Conventions

- All endpoints are versioned under `/api/v1/`
- Use plural nouns for resources: `/api/v1/tasks`, `/api/v1/tasks/{task_id}`
- Auth endpoints: `/api/v1/auth/signup`, `/api/v1/auth/login`, `/api/v1/auth/refresh`
- Every protected route requires a valid JWT in the `Authorization: Bearer <token>` header
- Request/response bodies are always validated via Pydantic schemas — never accept raw dicts in route handlers
- Return consistent error shapes:
  ```json
  { "detail": "human-readable message" }
  ```
- Use proper HTTP status codes (401 for auth failures, 404 for missing resources, 422 for validation errors — this is FastAPI's default, don't override it)

## Database Conventions

- Tables: snake_case, plural (`users`, `tasks`, `daily_updates`)
- Every table has `id` (PK), `created_at`, `updated_at`
- Foreign keys named `<resource>_id` (e.g., `user_id`)
- All schema changes go through Alembic migrations — never hand-edit the database or use `create_all()` outside of local scratch testing
- Passwords are always hashed (bcrypt via `passlib`) — never store or log plaintext passwords

## Frontend Conventions

- Functional components with hooks only — no class components
- API calls live in `src/api/`, one file per resource (`tasksApi.js`, `authApi.js`) — components should never call `fetch`/`axios` directly
- Auth token storage: For this demo project, tokens are stored in localStorage for simplicity. In production, consider httpOnly cookies or more secure patterns. Always call this out as a training simplification if teaching security topics.
- Keep components small and single-purpose; page-level components live in `pages/`, reusable UI in `components/`

## Environment Variables

Defined in `.env` (see `.env.example` for the template). Never commit `.env`. Key variables:

```
DATABASE_URL=mysql+pymysql://user:password@db:3306/tasklog
JWT_SECRET_KEY=
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
REFRESH_TOKEN_EXPIRE_DAYS=7
VITE_API_BASE_URL=http://localhost:8000/api/v1
```

## Port Configuration

When running via Docker Compose:
- **Backend API**: `http://localhost:8000` (FastAPI with auto-reload)
- **Frontend Dev Server**: `http://localhost:5173` (Vite default)
- **MySQL**: `localhost:3306` (exposed for direct db access if needed)

Ports are mapped in `docker-compose.yml`. Frontend makes API calls to the backend via the `VITE_API_BASE_URL` environment variable.

## CORS Configuration

- **Development**: Backend must allow `http://localhost:5173` (frontend origin)
- Configured in `backend/app/main.py` using FastAPI's `CORSMiddleware`
- Allow credentials (for JWT cookies if httpOnly pattern is adopted later)
- Allowed methods: GET, POST, PUT, DELETE, PATCH
- Keep CORS permissive for local development; tighten for production deployments

## Testing Conventions

**Backend (pytest):**
- Test files: `backend/app/tests/test_*.py`
- Aim for >80% coverage on `services/` (business logic)
- Test route handlers for status codes and response shapes
- Use pytest fixtures for db sessions and test users
- Mock external dependencies where appropriate

**Frontend (Vitest + React Testing Library):**
- Test files: `*.test.jsx` or `*.test.js` alongside components
- Focus on user interactions and component behavior, not implementation details
- Mock API calls in `src/api/` modules
- Test authentication flows and protected route behavior
- Run tests with `npm run test`

## Logging

**Backend:**
- Use Python's `logging` module — avoid `print()` statements
- Log level: `INFO` for development, `WARNING` or `ERROR` for production
- Structured logs (JSON format) preferred for easier parsing in production
- Log authentication events (login, token refresh, failures) for audit trails
- Do not log sensitive data (passwords, full tokens)

**Frontend:**
- Use `console.error()` for errors, `console.warn()` for warnings
- Avoid leaving debug `console.log()` statements in committed code
- Consider a logging service (e.g., Sentry) for production error tracking (outside the scope of this demo)

## Git Workflow

- Branch naming: `feature/<short-description>`, `fix/<short-description>`
- Commit messages: imperative mood, short summary line (`Add task reminder logic`, not `Added` or `Adding`)
- `main` branch should always be in a runnable state (`docker compose up` works)
- Do not commit `.env`, `node_modules/`, `__pycache__/`, or `venv/` — ensure `.gitignore` covers these

## What Claude Should Do

- Ask before introducing a new library/dependency not already in `requirements.txt` or `package.json`
- Write tests alongside new backend endpoints (pytest) where practical
- Keep business logic out of route handlers — put it in `services/`
- Explain non-obvious decisions briefly in code comments, since this repo is used for teaching
- When generating example data or docs, keep tone and content appropriate for a developer training session (clear, minimal jargon, good comments)

## What Claude Should NOT Do

- Don't rewrite the auth flow to use sessions/cookies unless explicitly asked — this project is JWT-based by design
- Don't switch ORMs or database engines
- Don't merge frontend and backend into a single deployable unit
- Don't add production-hardening features (rate limiting, complex caching, etc.) unless the session is specifically covering that topic — keep the demo scope-controlled