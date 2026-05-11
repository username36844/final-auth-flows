# Production-Grade Login Flow (Cookie + Session ID Based)

This flow assumes:

* session-based auth
* cookies for auth
* server-side sessions
* no JWT access tokens
* user must verify email before login

This is a very solid architecture for SaaS apps.

---

# FRONTEND LOGIN FLOW

```text id="4q7jlwm"
User fills login form
↓
React Hook Form manages form
↓
Zod validates fields
↓
If invalid:
    show errors
    stop flow
↓
User clicks login
↓
if (isSubmitting) return
↓
Disable login button
↓
Send POST request to /api/auth/login
↓
Handle response
```

---

# BACKEND LOGIN FLOW

Route:

```text id="rjlwm7"
POST /api/auth/login
```

---

# 1. Validate Request Basics

Check:

* method is POST
* content-type is application/json
* request body exists

If invalid:

* return 400

STOP.

---

# 2. Parse JSON Safely

Try:

```text id="ef8oc8"
await req.json()
```

If invalid JSON:

* return 400

STOP.

---

# 3. Normalize Input

Before validation:

```text id="bxojrt"
trim email
lowercase email
```

---

# 4. Validate With Zod (`safeParse`)

Validate:

* email
* password

If validation fails:

* return 422 field errors

STOP.

---

# 5. Find User By Email

Query DB:

```text id="8q0qcb"
SELECT * FROM users WHERE email=normalized_email
```

---

# CASE A — User Not Found

Return generic auth error.

Example:

```json id="n3m4vo"
{
  "success": false,
  "message": "Invalid email or password"
}
```

STOP.

Never reveal:

* account existence
* whether email is registered

---

# 6. Check Email Verification

If:

```text id="zv1wmu"
email_verified === false
```

Return:

```json id="4a5d53"
{
  "success": false,
  "message": "Please verify your email before logging in"
}
```

Optional:

* resend verification CTA

STOP.

---

# 7. Verify Password

Use:

* Argon2 verify

Compare:

* incoming password
* stored password hash

---

# CASE A — Password Invalid

Return generic auth error.

```json id="70oz1f"
{
  "success": false,
  "message": "Invalid email or password"
}
```

STOP.

---

# 8. Create Session ID

Generate:

* cryptographically secure random session ID

Example:

* 32+ random bytes

---

# 9. Hash Session ID (Recommended)

Better production approach:

Store:

* hashed session ID in DB

Send:

* raw session ID in cookie

This protects sessions if DB leaks.

---

# 10. Create Session Record

Insert into:
`sessions`

Fields:

```text id="v5y8rn"
id
user_id
session_hash
expires_at
created_at
ip_address
user_agent
```

Optional:

* last_activity_at

---

# 11. Set HTTP-Only Cookie

Set cookie:

```text id="nk5i4y"
session=RAW_SESSION_ID
```

Cookie settings:

```text id="r8c0yo"
httpOnly: true
secure: true (production)
sameSite: "lax"
path: "/"
```

Optional:

* domain
* maxAge

---

# 12. Return Success Response

Return:

```json id="bz7r9y"
{
  "success": true
}
```

Frontend then:

* redirects to dashboard/app

---

# AUTHENTICATED REQUEST FLOW

Every protected request:

```text id="3f0e1q"
Browser automatically sends session cookie
↓
Server reads cookie
↓
Hash incoming session ID
↓
Find session in DB
↓
Validate session exists
↓
Check session expiry
↓
Load associated user
↓
Attach user to request context
↓
Continue request
```

---

# SESSION VALIDATION FLOW

For every protected route:

---

# 13. Read Session Cookie

Check:

* cookie exists

If missing:

* unauthorized

STOP.

---

# 14. Hash Incoming Session ID

Hash same way as stored DB hash.

---

# 15. Find Session

Lookup by:

* session hash

Include:

* user
* expiry

---

# CASE A — Session Not Found

Clear cookie.

Return:

* unauthorized

STOP.

---

# CASE B — Session Expired

Delete session.

Clear cookie.

Return:

* unauthorized

STOP.

---

# CASE C — User Deleted/Suspended

Invalidate session.

Clear cookie.

Return:

* unauthorized

STOP.

---

# CASE D — Valid Session

Attach:

* user
* session

to request context.

Continue.

---

# LOGOUT FLOW

Route:

```text id="t4vjlwm"
POST /api/auth/logout
```

---

# 16. Read Session Cookie

Get:

* session ID

---

# 17. Delete Session From DB

Delete:

* current session

OR:

* all user sessions

depending on logout type.

---

# 18. Clear Cookie

Set expired cookie.

Example:

```text id="v66g44"
Max-Age=0
```

---

# 19. Return Success

Frontend:

* redirect to login

---

# SESSION ROTATION (Recommended)

Very important production feature.

Rotate session:

* after login
* after password change
* after privilege escalation

This reduces:

* session fixation risks

---

# SESSION EXPIRY STRATEGY

Two common approaches.

---

## Absolute Expiry

Example:

* expires in 30 days no matter what

---

## Sliding Sessions (Better UX)

Each valid request:

* extends session expiry

Example:

* active users stay logged in

Most SaaS apps use this.

---

# IMPORTANT SECURITY RULES

---

# NEVER Store Raw Passwords

Only:

* Argon2id hash

---

# NEVER Store Raw Session IDs

Prefer:

* hashed session IDs

---

# ALWAYS Use HTTPS In Production

Otherwise cookies are vulnerable.

---

# ALWAYS Use HTTP-Only Cookies

Prevents JS access.

Protects against:

* XSS session theft

---

# ALWAYS Use SameSite

Recommended:

```text id="m8m0p3"
sameSite: "lax"
```

---

# FINAL LOGIN FLOW

```text id="0p4x5g"
Frontend Validation
↓
POST /api/auth/login
↓
Safe JSON Parse
↓
Normalize Input
↓
Zod Validation
↓
Find User
↓
Check Email Verification
↓
Verify Password
↓
Generate Session ID
↓
Hash Session ID
↓
Store Session
↓
Set HTTP-Only Cookie
↓
Return Success
↓
Redirect User
```

This is a strong modern auth architecture for a Next.js SaaS app using cookie-based sessions.
