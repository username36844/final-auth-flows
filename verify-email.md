# EMAIL VERIFICATION ROUTE FLOW

Route:

```text id="a5s0md"
GET /verify-email/confirm?token=abc
```

This route is responsible for:

* validating verification token
* activating user account
* deleting verification token
* redirecting user to login

---

# EMAIL VERIFICATION FLOW

---

# 1. Read Token From Query Params

Read:

```text id="8tnrmu"
token
```

from URL query params.

Example:

```text id="x7g2vw"
/verify-email/confirm?token=abc
```

---

# 2. Validate Token Presence

Check:

* token exists
* token is string
* token length valid

---

# CASE A — Token Missing

Return/show:

```text id="1t05k4"
Invalid verification link
```

STOP.

---

# CASE B — Invalid Token Format

Example:

* extremely short token
* malformed token

Return/show:

```text id="tw0p1h"
Invalid verification link
```

STOP.

Do not query DB unnecessarily.

---

# 3. Hash Incoming Token

Hash incoming token same way as stored token hash.

Never store or compare raw token directly.

---

# 4. Find Verification Token In Database

Lookup by:

* token_hash

Include:

* user
* expiry
* created_at

---

# CASE A — Token Not Found

Return/show:

```text id="cr0qie"
Verification link is invalid or expired
```

STOP.

---

# CASE B — Token Expired

Optional:

* delete expired token

Return/show:

```text id="5imyn7"
Verification link has expired
```

Optional CTA:

* resend verification email

STOP.

---

# CASE C — User Already Verified

Return/show:

```text id="s9jmmw"
Email already verified
```

Show:

* login button/link

STOP.

---

# CASE D — Valid Token

Continue.

---

# 5. Start Database Transaction

Everything below should happen atomically.

---

# 6. Mark User As Verified

Update user:

```text id="1dj7mt"
email_verified=true
status=active
email_verified_at=now()
```

---

# 7. Delete Verification Token

Delete token from DB.

This makes token:

* single use
* non-reusable

---

# 8. Commit Transaction

Verification complete.

---

# 9. Redirect User To Login

Redirect:

```text id="l1f5fg"
/login
```

Optional:

* success query param

Example:

```text id="z9hr2v"
/login?verified=true
```

Useful for:

* success toast/message

---

# LOGIN PAGE AFTER REDIRECT

Example flow:

```text id="0z6m4q"
/login?verified=true
↓
Show:
"Email verified successfully. Please login."
```

---

# FINAL VERIFY EMAIL FLOW

```text id="4x9k40"
User Clicks Verification Email Link
↓
GET /verify-email/confirm?token=abc
↓
Read Token
↓
Validate Token Format
↓
Hash Token
↓
Find Verification Token
↓
Check Expiry
↓
Check User Verification Status
↓
Start Transaction
↓
Mark User Verified
↓
Delete Verification Token
↓
Commit Transaction
↓
Redirect To /login?verified=true
```

This is the clean production-grade verification route flow for your current auth architecture.
