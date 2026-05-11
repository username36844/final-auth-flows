# FRONTEND LOGOUT FLOW

```text id="y6yitf"
User clicks logout button
↓
if (isLoggingOut) return
↓
Disable logout button
↓
Send POST request to /api/auth/logout
↓
Handle API response
↓
Clear frontend auth state/cache
↓
Redirect user to /login
```

---

# BACKEND LOGOUT FLOW

Route:

```text id="2uyd1u"
POST /api/auth/logout
```

---

# 1. Validate Request Basics

Check:

* method is POST

If invalid:

* return 400

STOP.

---

# 2. Read Session Cookie

Read:

```text id="5a4knh"
session
```

from cookies.

---

# CASE A — Session Cookie Missing

User is already logged out.

Still return success.

Example:

```json id="g10a3g"
{
  "success": true
}
```

STOP.

Logout should be idempotent.

---

# 3. Hash Session ID

Hash incoming session ID same way as stored hash.

---

# 4. Find Session

Lookup session by:

* session hash

---

# CASE A — Session Not Found

Clear cookie anyway.

Return success.

STOP.

This handles:

* expired sessions
* deleted sessions
* stale cookies

---

# CASE B — Valid Session Exists

Continue.

---

# 5. Delete Session From Database

Delete current session record.

This invalidates authentication server-side.

---

# 6. Clear Session Cookie

Set expired cookie:

```text id="n0wggl"
session=""
Max-Age=0
Expires=past_date
Path=/
HttpOnly
Secure
SameSite=Lax
```

Important:

* cookie config must match original cookie settings

---

# 7. Return Success Response

Return:

```json id="1lg1ga"
{
  "success": true
}
```

Status:

* 200

---

# FRONTEND POST-LOGOUT FLOW

After success:

```text id="t6odrq"
Clear auth/user state
↓
Clear cached protected data
↓
Redirect to /login
```

---

# FINAL LOGOUT FLOW

```text id="4ny3zt"
User Clicks Logout
↓
POST /api/auth/logout
↓
Read Session Cookie
↓
Hash Session ID
↓
Find Session
↓
Delete Session
↓
Clear Cookie
↓
Return Success
↓
Clear Frontend Auth State
↓
Redirect To Login
```
