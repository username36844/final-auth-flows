# Your Current Production Signup Flow

Based on your actual implementation decisions:

* React Hook Form
* Zod
* `isSubmitting`
* email verification required
* no auto login
* redirect to login after verification
* verification route:
  `/verify-email/confirm?token=abc`
* no rate limiting yet
* no CAPTCHA yet
* no breached password check yet

This is the clean flow you should implement right now.

---

# FRONTEND SIGNUP FLOW

```text id="o7yqk8"
User fills signup form
↓
React Hook Form manages state
↓
Zod validates fields client-side
↓
If invalid:
    show field errors
    stop flow
↓
User clicks submit
↓
if (isSubmitting) return
↓
Disable submit button
↓
Send POST request to /api/auth/signup
↓
Handle API response
```

---

# BACKEND SIGNUP FLOW

Route:

```text id="6ifntk"
POST /api/auth/signup
```

---

# 1. Check Request Type

Validate:

* method is POST
* content-type is application/json

If invalid:

* return 400

STOP.

---

# 2. Parse JSON Safely

Try:

```text id="q4jvry"
await req.json()
```

If parsing fails:

* return 400 invalid request body

STOP.

---

# 3. Normalize Input

Before validation:

```text id="n50q3e"
trim email
lowercase email
trim name
```

Example:

```text id="lz2jrf"
" Test@Example.COM "
↓
"test@example.com"
```

---

# 4. Validate Using Zod (`safeParse`)

Validate:

* email
* password
* confirmPassword
* name if needed

If validation fails:

* return 422 with field errors

STOP.

---

# 5. Check Password Match

```text id="igv8c8"
password === confirmPassword
```

If not:

* return 422

STOP.

After this:

* discard confirmPassword completely

---

# 6. Check Existing User

Query DB by normalized email.

---

# CASE A — Verified User Exists

Return:

* 409 conflict

Example:

```json id="v3s6o2"
{
  "success": false,
  "message": "Account already exists"
}
```

STOP.

---

# CASE B — Unverified User Exists

Do NOT create another account.

Flow:

```text id="cwr8rq"
Delete old verification tokens
↓
Generate new verification token
↓
Hash token
↓
Store hashed token
↓
Send verification email again
↓
Return success response
```

STOP.

This is important to avoid:

* duplicate unverified accounts
* token chaos

---

# CASE C — User Does Not Exist

Continue.

---

# 7. Hash Password

Use:

* Argon2id

With:

* salt
* pepper from env

If hashing fails:

* return 500

STOP.

---

# 8. Start Database Transaction

Everything below should succeed together.

---

# 9. Create User

Insert:

```text id="mq15q7"
id
email
password_hash
role=user
status=pending_verification
email_verified=false
created_at
```

Important:

* email column MUST be unique

This prevents race conditions.

---

# 10. Generate Verification Token

Generate:

* secure random token

Example:

* 32 random bytes

---

# 11. Hash Verification Token

Do NOT store raw token.

Store:

* hashed version only

---

# 12. Store Verification Token

Insert:

```text id="xew9ni"
user_id
token_hash
expires_at
created_at
```

---

# 13. Commit Transaction

Now:

* user exists
* token exists

Database is safe.

---

# 14. Send Verification Email

Create verification URL:

```text id="x0r1h3"
/verify-email/confirm?token=RAW_TOKEN
```

Send email containing:

* verify button
* fallback URL

---

# 15. Return Success Response

Return:

```json id="o8p8xh"
{
  "success": true,
  "message": "Verification email sent"
}
```

Status:

* 201

---

# EMAIL VERIFICATION FLOW

Route:

```text id="gm9psj"
/verify-email/confirm?token=abc
```

---

# 16. Read Token From Query Params

Check:

* token exists
* token length valid

If invalid:

* show invalid verification page

STOP.

---

# 17. Hash Incoming Token

Hash token same way as stored token hash.

---

# 18. Find Verification Token

Lookup by:

* token_hash

Include:

* user
* expiry

---

# CASE A — Token Not Found

Show:

* invalid or expired verification link

STOP.

---

# CASE B — Token Expired

Delete token optionally.

Show:

* verification expired
* resend verification CTA

STOP.

---

# CASE C — User Already Verified

Show:

* email already verified
* login button

STOP.

---

# CASE D — Valid Token

Continue.

---

# 19. Start Verification Transaction

Inside transaction:

---

## Mark User Verified

Update:

```text id="dclmse"
email_verified=true
status=active
email_verified_at=now()
```

---

## Delete Verification Token

Delete token after successful verification.

This makes token:

* single use

---

# 20. Commit Transaction

Verification complete.

---

# 21. Redirect User To Login

Redirect:

```text id="tw5kmi"
/login
```

Optional:

* show success toast/message

Example:

```text id="1flcpx"
Email verified successfully. Please login.
```

---

# FINAL FLOW

```text id="qj7mwy"
Frontend Validation
↓
POST /api/auth/signup
↓
Safe JSON Parse
↓
Normalize Input
↓
Zod safeParse
↓
Check Existing User
↓
Hash Password
↓
Create User
↓
Generate Verification Token
↓
Hash Token
↓
Store Token
↓
Send Verification Email
↓
Return Success
↓
User Clicks Email Link
↓
Validate Token
↓
Hash Incoming Token
↓
Find Token
↓
Verify User
↓
Delete Token
↓
Redirect To Login
```

This is a clean, modern, production-quality flow for your current stage.
