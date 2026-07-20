# 05 – Frontend Auth Flow Spec

**Project:** TaskLog
**Scope:** React sign-up/login screens, auth context, protected routing
**Depends on:** `02-auth-api.md` (backend endpoints must exist)
**Status:** Draft — ready for implementation

---

## 1. Objective

Build the React screens and supporting state for sign-up, login, logout, and route protection, so an unauthenticated user is always routed to login and an authenticated user's session persists across page reloads.

## 2. Pages

### 2.1 `pages/SignupPage.jsx`

- Fields: full name, email, password (with a visible "show password" toggle — good small UX detail for trainees to notice).
- On submit: call `authApi.signup()`. On success, store tokens (see Section 4) and redirect to `/dashboard`. On failure, show the error message returned from the API (`detail` field) in a visible, non-blocking way (inline text, not a browser `alert()`).
- Link to `/login` for existing users.

### 2.2 `pages/LoginPage.jsx`

- Fields: email, password.
- On submit: call `authApi.login()`. Same success/failure handling pattern as signup.
- Link to `/signup` for new users.

### 2.3 Route Protection

- `components/ProtectedRoute.jsx`: wraps any route that requires auth. Checks auth context state; if not authenticated, redirect to `/login`. While the initial "am I logged in" check is in flight (see 4.3), show a simple loading state rather than flashing the login page and then redirecting again.
- Unauthenticated users hitting any protected URL directly (e.g., pasting `/dashboard` into the browser) must be redirected to `/login`, not shown a broken/empty page.

## 3. `api/authApi.js`

Exports:
- `signup({ email, password, fullName })`
- `login({ email, password })`
- `refresh(refreshToken)`
- `getMe()`
- `logout()` — client-side only: clears stored tokens and resets auth context; no backend call needed for this demo (no token blacklist/revocation endpoint in scope).

All requests go through a shared `apiClient` (axios instance) with `VITE_API_BASE_URL` as the base URL.

## 4. Auth State (`context/AuthContext.jsx`)

### 4.1 State shape

```js
{
  user: null | { id, email, fullName },
  accessToken: null | string,
  refreshToken: null | string,
  isLoading: boolean,   // true during initial "check session" on app load
  isAuthenticated: boolean
}
```

### 4.2 Storage

- Store `accessToken` and `refreshToken` in `localStorage` for this demo. This is a deliberate simplification (already flagged in `CLAUDE.md`) — worth calling out to trainees as a talking point on XSS risk vs. httpOnly cookies, without needing to implement the more complex cookie-based pattern for this session.

### 4.3 On App Load

- Read tokens from `localStorage`. If an access token exists, call `getMe()` to validate it and populate `user`.
- If `getMe()` returns `401`, attempt one silent `refresh()` using the stored refresh token; if that also fails, clear storage and treat the user as logged out.
- Set `isLoading: false` once this check resolves, either way — this is what `ProtectedRoute` waits on.

### 4.4 Axios Interceptor

- Attach `Authorization: Bearer <accessToken>` to every outgoing request automatically via an axios request interceptor — components should never manually set this header.
- On a `401` response, attempt a single silent token refresh and retry the original request once. If the refresh also fails, clear auth state and redirect to `/login`. Avoid infinite retry loops — cap it at one retry.

## 5. Component Behavior Notes

- Disable the submit button and show a loading indicator while a signup/login request is in flight — don't allow double-submits.
- Password field: minimum client-side validation (8 characters) purely for immediate feedback; the backend remains the source of truth for validation.
- After logout, redirect to `/login` and ensure no protected data lingers in memory/state (clear `user` from context).

## 6. Out of Scope for This Spec

- "Remember me" vs. session-only login distinction
- Password reset / forgot-password UI
- Social login buttons
- Multi-tab session sync (e.g., logging out in one tab affecting others) — fine to mention as an advanced extension topic

## 7. Acceptance Criteria

- [ ] New user can sign up through the UI and land on the dashboard already logged in
- [ ] Existing user can log in and land on the dashboard
- [ ] Refreshing the page while logged in keeps the user logged in (session persists)
- [ ] Visiting a protected route while logged out redirects to `/login`
- [ ] An expired access token is silently refreshed once; if refresh also fails, the user is logged out and redirected
- [ ] Logout clears tokens and user state, and returns the user to `/login`
