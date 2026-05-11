# RESEND VERIFICATION EMAIL FLOW

This flow allows users to manually request a new verification email.

Route:

```text id="64crf0"
POST /api/auth/resend-verification
```

Request body:

```json id="j81rqf"
{
  "email": "user@example.com"
}
```

---

# FRONTEND FLOW

```text id="v2o7e7"
User enters email
↓
React Hook Form manages form
↓
Zod validates email
↓
If invalid:
    show field error
    stop flow
↓
User clicks resend button
↓
if (isSubmitting) return
↓
Disable button
↓
Send POST request
↓
Handle response
```

---

# BACKEND FLOW

---

# 1. Validate Request Basics

Check:

* method is POST
* content-type is application/json

If invalid:

* return 400

STOP.

---

# 2. Parse JSON Safely

Try:

```text id="jczjn8"
await req.json()
```

If invalid JSON:

* return 400

STOP.

---

# 3. Normalize Input

Before validation:

```text id="5xjlwm"
trim email
lowercase email
```

---

# 4. Validate With Zod (`safeParse`)

Validate:

* email

If validation fails:

* return 422 field errors

STOP.

---

# 5. Find User By Email

Query DB using normalized email.

---

# CASE A — User Not Found

Return generic success response.

Example:

```json id="o65w0i"
{
  "success": true,
  "message": "If the account exists, a verification email has been sent"
}
```

STOP.

This prevents:

* account enumeration

---

# CASE B — User Already Verified

Return:

```json id="7wwfqe"
{
  "success": false,
  "message": "Email is already verified"
}
```

STOP.

---

# CASE C — Unverified User Exists

Continue.

---

# 6. Delete Existing Verification Tokens

Delete old verification tokens for user.

This prevents:

* multiple active tokens
* token confusion

---

# 7. Generate New Verification Token

Generate:

* cryptographically secure random token

Example:

* 32 random bytes

---

# 8. Hash Verification Token

Never store raw token.

Store:

* hashed token only

---

# 9. Store Verification Token in user model

---

# 10. Send Verification Email

Create verification URL:

```text id="txjov9"
/verify-email/confirm?token=RAW_TOKEN
```

Send email containing:

* verify button
* fallback verification link

---

# 11. Return Success Response

Return:

```json id="fpq54n"
{
  "success": true,
  "message": "Verification email sent"
}
```

Status:

* 200

---

# OPTIONAL IMPROVEMENT — RESEND COOLDOWN

Very recommended later.

Before generating new token:

Check:

* last verification email timestamp

If too recent:

* block resend temporarily

Example:

* 60 second cooldown

This prevents:

* email spam
* SMTP abuse

You can add this later.

---

# FINAL RESEND VERIFICATION FLOW

```text id="hjrhx9"
User Submits Email
↓
POST /api/auth/resend-verification
↓
Safe JSON Parse
↓
Normalize Email
↓
Zod Validation
↓
Find User
↓
Check Verification Status
↓
Delete Old Verification Tokens
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
```

This is the clean production-ready resend verification flow for your current auth system.
