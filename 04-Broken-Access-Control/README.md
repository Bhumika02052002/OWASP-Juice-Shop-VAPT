# OWASP Juice Shop — Broken Access Control

## 1. Admin Section

### Goal

Access the **administration section** without having administrator privileges.

### How to Solve

**Step 1 — Inspect JavaScript source**

Open:

```text
Developer Tools → Sources
```

Search JavaScript files for:

```text
administration
```

You can also use:

```text
Ctrl + Shift + F
```

You should find the hidden route:

```text
/#/administration
```

### Step 2 — Open the Route

Navigate to:

```text
http://localhost:3000/#/administration
```

### Impact

The application exposes an administrative route that should be restricted by proper authorization checks.

### Security Fix

Implement **server-side authorization**, not just hidden frontend routes.

---

# 2. View Basket

### Goal

View another user's shopping basket.

### Vulnerability

**IDOR / Broken Object-Level Authorization (BOLA)**

The basket is referenced by a numeric ID, and the server fails to properly verify that the requested basket belongs to the authenticated user.

### Step 1 — Capture Request in Burp

Open your basket and intercept the request.

Typical request:

```http
GET /rest/basket/2 HTTP/1.1
Host: localhost:3000
```

The important part is:

```text
/rest/basket/2
```

### Step 2 — Change Basket ID

Send the request to **Repeater**.

Change:

```http
GET /rest/basket/2
```

to another ID, for example:

```http
GET /rest/basket/1
```

Click **Send**.

### Step 3 — Observe Response

If the application returns another user's basket, the challenge is solved.

### Attack Flow

```text
Own Basket
    ↓
Capture Request
    ↓
GET /rest/basket/2
    ↓
Change ID
    ↓
GET /rest/basket/1
    ↓
Another User's Basket
```

### Impact

An attacker may access other users' private shopping data by modifying an object identifier.

### Security Fix

The backend must verify authorization:

```text
Requested Basket ID
        ↓
Does it belong to logged-in user?
        ↓
   YES → Allow
    NO → 403 Forbidden
```

**Key point:** Never rely on the frontend to enforce access control. Authorization must be checked **server-side for every object/request**.
