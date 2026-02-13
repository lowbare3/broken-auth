# The Silent Server — Solution

This repository contains my solution to the **Backend Debugging Assignment**, where the goal was to fix a broken Express.js authentication API and complete the full auth flow.

## Bugs Found & Fixed

### Bug 1: Logger Middleware Missing `next()` — `middleware/logger.js`
The request logger middleware never called `next()`, causing **every single request to hang silently** (hence the name "The Silent Server"). Added `next()` after the response listener setup.

### Bug 2: Auth Middleware Missing `next()` — `middleware/auth.js`
The JWT verification middleware decoded the token and set `req.user`, but never called `next()`. This meant the protected route handler was never reached, even with a valid token.

### Bug 3: `cookieParser()` Not Registered — `server.js`
`cookie-parser` was imported but never registered as middleware (`app.use(cookieParser())`). Without it, `req.cookies` was always `undefined`, breaking the token endpoint.

### Bug 4: OTP Value Not Logged — `server.js`
The login endpoint logged `[OTP] Session abc12345 generated` but **didn't include the actual OTP number**, making it impossible to complete the verification step. Fixed to log `OTP: <value>`.

### Bug 5: Token Endpoint Reading Wrong Source — `server.js`
The `/auth/token` endpoint read the session from `req.headers.authorization` (Bearer token) instead of `req.cookies.session_token`. The session ID was set as a **cookie** during OTP verification, not as an Authorization header.

## How to Run

1. Install dependencies:
   ```bash
   npm install
   ```

2. Start the server:
   ```bash
   npm start
   ```
   Server runs at: `http://localhost:3000`

## Testing the Auth Flow

```bash
# Task 1: Login
curl -X POST http://localhost:3000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"<your_email>","password":"password123"}'

# Task 2: Verify OTP (use OTP from server logs)
curl -c cookies.txt -X POST http://localhost:3000/auth/verify-otp \
  -H "Content-Type: application/json" \
  -d '{"loginSessionId":"<from_task_1>","otp":"<from_server_logs>"}'

# Task 3: Get Token
curl -b cookies.txt -X POST http://localhost:3000/auth/token

# Task 4: Access Protected Route
curl -H "Authorization: Bearer <jwt_from_task_3>" http://localhost:3000/protected
```

## Output

See [`output.txt`](./output.txt) for the full terminal output of all 4 test commands, including the `success_flag`.
