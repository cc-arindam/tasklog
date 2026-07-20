# 01 – Database Setup Spec

**Project:** TaskLog
**Scope:** MySQL schema design, migrations, and local Docker setup
**Depends on:** `CLAUDE.md` (root) for global conventions
**Status:** Draft — ready for implementation

---

## 1. Objective

Set up the MySQL database for TaskLog, including schema for users, tasks, and daily updates, wired up through SQLAlchemy models and Alembic migrations, running inside Docker alongside the backend service.

This spec defines *what* to build. Claude should implement exactly this schema unless a limitation is discovered — in that case, flag it before deviating.

## 2. Tech Choices (recap from CLAUDE.md)

- **Database:** MySQL 8
- **ORM:** SQLAlchemy 2.x
- **Migrations:** Alembic
- **Driver:** PyMySQL (`mysql+pymysql://`)
- **Container:** MySQL runs as a `db` service in `docker-compose.yml`, with a named volume for persistence

## 3. Entities & Schema

### 3.1 `users`

| Column          | Type            | Constraints                          |
|-----------------|-----------------|---------------------------------------|
| id              | BIGINT          | PK, autoincrement                     |
| email           | VARCHAR(255)    | UNIQUE, NOT NULL                      |
| password_hash   | VARCHAR(255)    | NOT NULL                              |
| full_name       | VARCHAR(150)    | NOT NULL                              |
| is_active       | BOOLEAN         | NOT NULL, DEFAULT TRUE                |
| created_at      | DATETIME        | NOT NULL, DEFAULT CURRENT_TIMESTAMP   |
| updated_at      | DATETIME        | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |

Notes:
- `email` is the login identifier. Case-insensitive uniqueness — normalize to lowercase before insert/lookup.
- `password_hash` stores bcrypt hash only. Never store plaintext, never log this field.
- `is_active` reserved for future soft-disable of accounts (e.g., after too many failed logins) — not required for MVP auth flow, but include the column now to avoid a later migration.

### 3.2 `tasks`

| Column       | Type            | Constraints                          |
|--------------|-----------------|---------------------------------------|
| id           | BIGINT          | PK, autoincrement                     |
| user_id      | BIGINT          | FK → users.id, NOT NULL, ON DELETE CASCADE |
| title        | VARCHAR(255)    | NOT NULL                              |
| description  | TEXT            | NULLABLE                              |
| status       | ENUM('pending','in_progress','completed') | NOT NULL, DEFAULT 'pending' |
| due_date     | DATE            | NOT NULL                              |
| completed_at | DATETIME        | NULLABLE                              |
| created_at   | DATETIME        | NOT NULL, DEFAULT CURRENT_TIMESTAMP   |
| updated_at   | DATETIME        | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |

Notes:
- `due_date` (not `created_at`) is what drives "today's pending tasks" and reminder logic — a task is "for" the day it's due, not the day it was created.
- `completed_at` is set when `status` transitions to `completed`; cleared if reopened. Set this in the service layer, not via a DB trigger.
- Index `(user_id, due_date)` — this is the primary query pattern (dashboard: "give me this user's tasks for today").

### 3.3 `daily_updates`

| Column       | Type            | Constraints                          |
|--------------|-----------------|---------------------------------------|
| id           | BIGINT          | PK, autoincrement                     |
| user_id      | BIGINT          | FK → users.id, NOT NULL, ON DELETE CASCADE |
| update_date  | DATE            | NOT NULL                              |
| summary_text | TEXT            | NOT NULL                              |
| created_at   | DATETIME        | NOT NULL, DEFAULT CURRENT_TIMESTAMP   |
| updated_at   | DATETIME        | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |

Notes:
- One row per user per day. Enforce with a UNIQUE constraint on `(user_id, update_date)` — regenerating the update for a day should overwrite the existing row (upsert), not create duplicates.
- `summary_text` is the generated/edited daily standup-style note (completed + pending tasks compiled into readable text). Generation logic lives in `services/`, not in the DB layer — this table just stores the result.

## 4. Relationships (ERD summary)

```
users (1) ───< (many) tasks
users (1) ───< (many) daily_updates
```

- One user has many tasks.
- One user has many daily updates (max one per calendar date).
- No direct relationship between `tasks` and `daily_updates` at the DB level — the link is computed at generation time by querying tasks for that `user_id` + date range, not stored as a foreign key. Do not add a `task_id` column to `daily_updates`.

## 5. Migrations

- Use Alembic for every schema change — no `Base.metadata.create_all()` outside of throwaway local scratch testing.
- Migration file naming: Alembic's autogenerated slug is fine (e.g., `xxxx_create_users_table.py`), but the `-m` message must be descriptive: `alembic revision --autogenerate -m "create users table"`.
- Order of initial migrations:
  1. `create users table`
  2. `create tasks table`
  3. `create daily_updates table`
- Each migration must have a working `downgrade()` — don't leave it as `pass`.

## 6. Docker / Compose Setup

`db` service requirements in `docker-compose.yml`:

- Image: `mysql:8`
- Environment: `MYSQL_DATABASE`, `MYSQL_USER`, `MYSQL_PASSWORD`, `MYSQL_ROOT_PASSWORD` (sourced from `.env`, never hardcoded)
- Named volume for `/var/lib/mysql` so data survives `docker compose down` (but not `docker compose down -v`)
- Healthcheck so the backend service waits for MySQL to be ready before starting (`depends_on: condition: service_healthy`)
- Expose port 3306 to host only for local debugging convenience — not required for backend-to-db communication, which happens over the Docker network by service name (`db`)

## 7. Seed Data (local dev only)

Provide a small seed script (`backend/app/db/seed.py`) that creates:
- 1 demo user (e.g., `demo@tasklog.dev`)
- 4–5 sample tasks across today, yesterday, and tomorrow's due dates, with a mix of statuses
- 1 sample daily update for yesterday

This is for demo/training convenience only — must not run automatically in any non-local environment.

## 8. Out of Scope for This Spec

- API endpoint implementation (covered in a separate spec, e.g. `02-auth-api.md`)
- Frontend data fetching
- Reminder/notification delivery mechanism (email, push, etc.) — this spec only ensures the data needed for reminders is queryable

## 9. Acceptance Criteria

- [ ] `docker compose up` brings up a healthy MySQL container with the `tasklog` database created
- [ ] SQLAlchemy models exist for `User`, `Task`, `DailyUpdate` matching the schema above
- [ ] Alembic migrations run cleanly from scratch (`alembic upgrade head`) and can be rolled back (`alembic downgrade -1`) without error
- [ ] Unique constraints (`users.email`, `daily_updates.user_id + update_date`) are enforced and verified with a quick manual test or unit test
- [ ] Seed script populates demo data successfully for local development