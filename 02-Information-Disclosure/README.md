Sensitive Data Exposure - Confidential Document

Vulnerability

Sensitive Data Exposure / Publicly Accessible Confidential File

Target

OWASP Juice Shop - Local Lab

How I Found It

Intercepted application traffic using Burp Suite.

Found the /ftp/ directory through endpoint enumeration.

Opened the directory listing and identified accessible documents.

Sent the file request to Burp Repeater.

Requested the confidential document directly.

Server returned HTTP/1.1 200 OK without requiring authorization.

Evidence

GET /ftp/<confidential-file> HTTP/1.1
Host: localhost:3000

Response:

HTTP/1.1 200 OK

The confidential document was accessible through a public web path.

Impact

Unauthorized users can access confidential information.

Sensitive business or internal information may be disclosed.

Exposed documents can support further attacks.

Severity

Medium

Root Cause

Sensitive files are stored in a web-accessible directory without proper
access control.

Solution

Remove confidential files from public/static directories.

Store sensitive files outside the web root.

Implement server-side authorization before serving protected files.

Disable directory listing where not required.

Review and restrict access to backup and temporary files.

Verification

After remediation, unauthenticated requests to the confidential file should
return 401 Unauthorized, 403 Forbidden, or another appropriate response,
rather than 200 OK.

Tools

Burp Suite

# 2. Exposed Credentials

## Vulnerability Overview

**Vulnerability:** Exposed Credentials in Client-Side JavaScript
**Category:** Sensitive Data Exposure
**Severity:** Medium
**Target:** `http://localhost:3000`
**Testing Environment:** OWASP Juice Shop Lab

### Description

During the security assessment, valid testing-account credentials were found hardcoded in the client-side JavaScript bundle.

Since JavaScript code is downloaded and executed in the user's browser, an attacker can inspect the source code using browser Developer Tools and recover credentials embedded within it.

### Discovery

The issue was identified using the following steps:

1. Opened the OWASP Juice Shop application.
2. Opened **Browser Developer Tools → Sources**.
3. Located the application's JavaScript bundle.
4. Searched the source code for credential-related keywords.
5. Found the variables `testingUsername` and `testingPassword`.
6. The following credentials were exposed:

```javascript
testingUsername = `testing@juice-sh.op`;
testingPassword = `IamUsedForTesting`;
```

### Evidence

| Parameter      | Value                  |
| -------------- | ---------------------- |
| Username       | `testing@juice-sh.op`  |
| Password       | `IamUsedForTesting`    |
| Location       | Client-side JavaScript |
| Authentication | Testing account        |

### Impact

An attacker who can access the application's JavaScript bundle may:

* Discover valid credentials.
* Authenticate using the exposed account.
* Access functionality available to that account.
* Potentially reuse the credentials if they are used in other environments.

The actual impact depends on the privileges assigned to the exposed account.

### Root Cause

The application contains valid authentication credentials directly inside client-side JavaScript.

Client-side code should be considered publicly accessible because users can download, inspect, and analyze JavaScript bundles.

### Remediation

* Never hardcode passwords or sensitive credentials in frontend JavaScript.
* Remove the exposed credentials from client-side code.
* Immediately revoke or rotate the exposed credentials.
* Store sensitive credentials securely on the server side.
* Use environment variables or a secure secrets-management solution for server-side secrets.
* Scan source code and production JavaScript bundles for accidentally exposed secrets.
* Apply least-privilege permissions to testing accounts.

### Verification

After remediation:

1. Inspect the JavaScript bundle again.
2. Search for the exposed username and password.
3. Confirm that the credentials are no longer present.
4. Verify that the exposed credentials have been revoked or rotated.
5. Confirm that the old credentials can no longer be used for authentication.

### Tools Used

* Browser Developer Tools
* Burp Suite
* OWASP Juice Shop
* Kali Linux
* JavaScript Source Inspection

### Security Note

This finding was identified in an **authorized local OWASP Juice Shop laboratory environment** for cybersecurity learning and VAPT practice.


Browser

OWASP Juice Shop

# 3. Login MC SafeSearch

## Goal

Log in with MC SafeSearch's original credentials **without using SQL Injection or any other authentication bypass**.

## Step 1 — Find MC SafeSearch's Email

I searched the product reviews to identify whether MC SafeSearch had posted any reviews.

### Product

**Juice Shop "Permafrost" 2020 Edition**

### Review

```text
mc.safesearch@juice-sh.op

🧊 Let it go, let it go 🎶
Can't hold it back anymore 🎶
Let it go, let it go 🎶
Turn away and slam the door ❄️
```

From the review, I identified MC SafeSearch's email address:

```text
mc.safesearch@juice-sh.op
```

## Step 2 — Find the Password

The email address was easy to discover, but the password was not directly visible.

I checked the challenge hint:

> **Listen to "Protect Ya Passwordz"**

The song contains a clue stating that MC SafeSearch's initial password is based on his first dog's name.

### Password Clue

First dog's name:

```text
Mr. Noodles
```

The clue specifies that the letter **O** should be replaced with **0**.

Therefore, the password is:

```text
Mr. N00dles
```

## Step 3 — Login

Use the following credentials on the Juice Shop Login page:

```text
Email:    mc.safesearch@juice-sh.op
Password: Mr. N00dles
```

Do not use SQL Injection or any authentication bypass.

## Result

Successfully authenticated using MC SafeSearch's original credentials.

### Credentials

| Field    | Value                       |
| -------- | --------------------------- |
| Email    | `mc.safesearch@juice-sh.op` |
| Password | `Mr. N00dles`               |

## Vulnerability Category

**Sensitive Data Exposure / OSINT**

## Key Learning

This challenge demonstrates how seemingly harmless public information, such as **product reviews and challenge-related clues**, can be combined to recover authentication credentials.

An attacker should always consider publicly exposed information when assessing the security of an application.

## Tools Used

* OWASP Juice Shop
* Browser
* Burp Suite
* Product Review Analysis
* OSINT

## Security Note

This testing was performed against the authorized local OWASP Juice Shop laboratory environment for cybersecurity learning and VAPT practice.

# 5. Login Cloud Admin

## Vulnerability Category

**Sensitive Data Exposure**

## Challenge

**Login Cloud Admin**

### Objective

Log in to the Cloud Admin's account **without using SQL Injection or any other authentication bypass**.

---

## Step 1 — Open the Administration Page

Navigate to:

```text
http://localhost:3000/#/administration
```

The Administration page requires authentication.

---

## Step 2 — Capture the Request

Open **Burp Suite → Proxy → HTTP History**.

Reload:

```text
/#/administration
```

Observe the requests generated by the application.

Look for interesting API requests and responses related to users, administration, or application data.

---

## Step 3 — Inspect Application Sources

Open browser:

```text
DevTools → Sources
```

Press:

```text
Ctrl + Shift + F
```

Search for keywords:

```text
cloud
admin
administration
email
password
```

Inspect JavaScript files and API responses for information unintentionally exposed to the client.

---

## Step 4 — Identify Cloud Admin Credentials

The objective is to discover the **Cloud Admin's legitimate credentials** through information exposed by the application.

Record the discovered credentials:

```text
Email: <Cloud Admin email>
Password: <Cloud Admin password>
```

No SQL Injection or brute force is required.

---

## Step 5 — Login Normally

Go to:

```text
http://localhost:3000/#/login
```

Enter the discovered Cloud Admin credentials using the normal login form.

---

## Step 6 — Verify Access

After successful authentication, open:

```text
http://localhost:3000/#/administration
```

If the account has Cloud Admin privileges, the administration page becomes accessible and the challenge is solved.

---

## Attack Flow

```text
Administration Page
        ↓
Burp / DevTools Analysis
        ↓
Find Exposed Information
        ↓
Identify Cloud Admin Credentials
        ↓
Normal Login
        ↓
Access Administration
```

## Tools Used

* OWASP Juice Shop
* Burp Suite Community Edition
* Browser DevTools
* JavaScript Source Analysis

## Impact

If administrator credentials are exposed through client-side resources or application responses, an attacker may obtain valid credentials and gain unauthorized administrative access.

## Remediation

* Never expose administrator credentials in client-side code.
* Keep sensitive information on the server side.
* Properly protect administrative endpoints.
* Implement strong authentication and authorization.
* Review API responses for sensitive information exposure.
* Remove unnecessary administrative information from public resources.

## Key Learning

**Sensitive Data Exposure can reveal valid credentials without requiring a technical authentication bypass. Always inspect client-side resources, API responses, and application functionality during a VAPT assessment.**
