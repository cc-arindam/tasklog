# 02 – Auth API Spec

**Project:** TaskLog
**Scope:** Sign-up, login, token refresh, and auth middleware for the FastAPI backend
**Depends on:** `01-db-setup.md` (users table must exist)
**Status:** Draft — ready for implementation

---

## 1. Objective

Implement JWT-based authentication: users can sign up, log in, and access protected endpoints using an access token, with a refresh flow so they aren't forced to log in repeatedly.

## 2. Endpoints

### 2.1 `POST /api/v1/auth/signup`

**Request body:**
```json
{
  "email": "user@example.com",
  "password": "plaintext-password",
  "full_name": "Jane Doe"
}
```

**Behavior:**
- Normalize email to lowercase before checking uniqueness or storing.
- Validate password strength: minimum 8 characters (keep validation simple for a demo — don't over-engineer with complex regex rules unless the session specifically covers that).
- Hash password with bcrypt (`passlib`) before storing.
- On success, create the user and return the same response shape as `/login` (i.e., sign-up immediately logs the user in — no separate "verify email" step for this demo).

**Response (201):**
```json
{
  "access_token": "...",
  "refresh_token": "...",
  "token_type": "bearer",
  "user": { "id": 1, "email": "user@example.com", "full_name": "Jane Doe" }
}
```

**Errors:**
- `409` if email already registered — response body: `{ "detail": "Email already registered" }`
- `422` for validation failures (handled automatically by Pydantic schema)

### 2.2 `POST /api/v1/auth/login`

**Request body:**
```json
{
  "email": "user@example.com",
  "password": "plaintext-password"
}
```

**Behavior:**
- Look up user by lowercased email.
- Verify password against stored hash.
- On success, issue access + refresh tokens (same response shape as signup).

**Errors:**
- `401` for invalid email or password — use the **same generic message** for both cases (`{ "detail": "Invalid email or password" }`) so the API doesn't leak which part was wrong.

### 2.3 `POST /api/v1/auth/refresh`

**Request body:**
```json
{ "refresh_token": "..." }
```

**Behavior:**
- Validate the refresh token's signature and expiry.
- Issue a new access token (and, for this demo, a new refresh token too — rotate on every use to keep the security story simple and consistent).

**Response (200):**
```json
{ "access_token": "...", "refresh_token": "...", "token_type": "bearer" }
```

**Errors:**
- `401` if refresh token invalid or expired — `{ "detail": "Invalid or expired refresh token" }`

### 2.4 `GET /api/v1/auth/me`

**Behavior:** Returns the current authenticated user's profile. Used by the frontend on app load to check "am I still logged in."

**Response (200):**
```json
{ "id": 1, "email": "user@example.com", "full_name": "Jane Doe" }
```

**Errors:** `401` if no valid access token supplied.

## 3. Token Design

- **Access token:** JWT, expires in `ACCESS_TOKEN_EXPIRE_MINUTES` (see `.env`, default 30 min). Payload includes `sub` (user id) and `exp`. Do not put email or name in the payload — look the user up from `sub` on each request.
- **Refresh token:** JWT, expires in `REFRESH_TOKEN_EXPIRE_DAYS` (default 7 days). Same minimal payload pattern.
- Signing algorithm: `HS256`, secret from `JWT_SECRET_KEY` env var — never hardcode the secret.
- Token creation and decoding logic lives in `core/security.py`. Route handlers call helper functions; they never touch `jwt.encode`/`jwt.decode` directly.

## 4. Auth Middleware / Dependency

- Implement a FastAPI dependency (e.g., `get_current_user`) using `OAuth2PasswordBearer` or a simple `Depends` reading the `Authorization: Bearer <token>` header.
- Any route needing auth declares `current_user: User = Depends(get_current_user)` — do not duplicate token-parsing logic in individual route handlers.
- If the token is missing, malformed, or expired, raise `HTTPException(401)` with a consistent detail message — do not distinguish "expired" vs "invalid" in the response (avoid leaking implementation details), though you may log the distinction server-side.

## 5. Security Notes (call out during training)

- Never log passwords or full tokens, even at debug level.
- Passwords are only ever handled as `SecretStr` or immediately hashed — never persisted or returned in any response.
- This demo stores tokens in the frontend per the simplification noted in `CLAUDE.md` (localStorage acceptable for demo purposes) — worth pointing out to trainees as a deliberate simplification, not a production pattern.

## 6. Out of Scope for This Spec

- Password reset / "forgot password" flow
- Email verification
- OAuth / social login
- Rate limiting on login attempts (mention as a "in production, you'd add this" discussion point)

## 7. Acceptance Criteria

- [ ] New user can sign up and immediately receives valid tokens
- [ ] Duplicate email sign-up returns `409`
- [ ] Login with correct credentials returns valid tokens; wrong credentials return `401` with generic message
- [ ] Protected route (`/auth/me`) returns `401` without a token and `200` with a valid one
- [ ] Refresh endpoint issues a new working access token from a valid refresh token
- [ ] Expired/invalid tokens are rejected with `401` everywhere they're used
