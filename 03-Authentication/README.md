# OWASP Juice Shop — Broken Authentication

> **Category:** Broken Authentication
> **Lab:** OWASP Juice Shop
> **Focus:** Weak Passwords & Weak Password Recovery

---

## 1. Password Strength

### Goal

Log in with the administrator's account **without changing the password and without SQL Injection**.

### Target Account

```text
Email: admin@juice-sh.op
```

### Step 1 — Identify the Account

Use the administrator email:

```text
admin@juice-sh.op
```

### Step 2 — Test Weak Passwords

The challenge is based on the administrator using a **weak/default password**. Try common passwords in the lab, such as:

```text
admin
admin123
password
Password123
123456
```

For the standard Juice Shop challenge, the administrator password is:

```text
admin123
```

### Step 3 — Login

```text
Email:    admin@juice-sh.op
Password: admin123
```

### Why It Works

The password does not meet a sufficiently strong password policy. An attacker can potentially discover it through **password guessing/brute force**.

### Impact

Weak administrator passwords can lead to:

* Account takeover
* Unauthorized administrative access
* Data exposure
* Privilege escalation

### Security Fix

Use:

* Strong, unique passwords
* Password-strength enforcement
* MFA
* Rate limiting / account lockout
* Protection against credential stuffing

---

# 2. Bjoern's Favorite Pet

### Goal

Reset **Bjoern's OWASP account** using the original answer to his security question.

### Target

```text
Account: Bjoern
Mechanism: Forgot Password
```

### Step 1 — Open Forgot Password

Go to:

```text
Login → Forgot Password
```

Enter Bjoern's email:

```text
bjoern@juice-sh.op
```

The application will ask a security question.

### Step 2 — Identify the Question

The challenge asks for:

```text
Bjoern's favorite pet
```

The intended answer is:

```text
Zaya
```

### Step 3 — Reset Password

Enter:

```text
Security Answer: Zaya
```

Then set a new password and log in with Bjoern's account.

### How It Works

The application relies on a **security question with a predictable/obtainable answer**.

```text
Bjoern's account
       ↓
Forgot Password
       ↓
Security Question
       ↓
"Favorite pet?"
       ↓
Zaya
       ↓
Password Reset
       ↓
Account Access
```

### Impact

Weak security questions can allow an attacker to:

* Reset another user's password
* Take over accounts
* Bypass normal authentication

### Security Fix

Use:

* MFA-based recovery
* Secure, random password-reset tokens
* Short token expiration
* One-time reset links
* Avoid knowledge-based security questions

---

## Key Takeaway

| Challenge             | Weakness                        | Impact                          |
| --------------------- | ------------------------------- | ------------------------------- |
| Password Strength     | Weak administrator password     | Account takeover                |
| Bjoern's Favorite Pet | Weak password-recovery question | Password reset/account takeover |

**Broken Authentication = authentication or recovery mechanisms that can be bypassed, guessed, or abused.**
