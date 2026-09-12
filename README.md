# OWASP Juice Shop – Web Application VAPT

![OWASP Juice Shop](https://img.shields.io/badge/Target-OWASP%20Juice%20Shop-orange)
![VAPT](https://img.shields.io/badge/Assessment-Web%20Application%20VAPT-red)
![OWASP](https://img.shields.io/badge/OWASP-Top%2010-blue)
![Status](https://img.shields.io/badge/Status-In%20Progress-yellow)
![License](https://img.shields.io/badge/Project-MIT-green)

> **Web Application Vulnerability Assessment & Penetration Testing (VAPT) PoC**

This repository documents a hands-on **Web Application VAPT assessment of OWASP Juice Shop**, an intentionally vulnerable web application created by OWASP for security training, penetration-testing practice, CTFs, security awareness, and security-tool testing.

The objective of this project is to demonstrate a practical penetration-testing workflow:

**Reconnaissance → Enumeration → Attack Surface Mapping → Vulnerability Identification → Manual Testing → Exploitation/Validation → Evidence Collection → Risk Assessment → Remediation**

---

## 📌 Project Overview

**OWASP Juice Shop** is a deliberately insecure modern web application containing vulnerabilities commonly found in real-world applications.

It is built using:

* **Frontend:** Angular
* **Backend:** Node.js
* **Web Framework:** Express
* **Database:** SQLite
* **Architecture:** Web application + REST APIs
* **Purpose:** Security training, CTF, vulnerability research and security-tool testing

Juice Shop covers vulnerabilities from the **OWASP Top 10** as well as additional security weaknesses mapped to resources such as OWASP ASVS, OWASP API Security Top 10 and CWE.

Official OWASP documentation describes Juice Shop as a deliberately insecure application containing vulnerabilities from the entire OWASP Top Ten and other real-world security flaws.

---

# 🎯 Objectives

The primary objectives of this VAPT project are:

1. Understand the application's attack surface.
2. Perform reconnaissance and enumeration.
3. Identify exposed endpoints, APIs and application functionality.
4. Test authentication and authorization controls.
5. Identify injection vulnerabilities.
6. Test client-side and server-side input handling.
7. Test access-control mechanisms.
8. Analyze HTTP requests and responses.
9. Test business-logic weaknesses.
10. Review security-related HTTP headers.
11. Validate vulnerabilities manually.
12. Capture reproducible evidence.
13. Document security impact.
14. Provide remediation recommendations.
15. Map findings to relevant OWASP categories.
16. Build a professional penetration-testing portfolio.

---

# ⚠️ Legal & Ethical Disclaimer

This repository is intended for **authorized security testing and educational purposes only**.

OWASP Juice Shop is intentionally vulnerable and is specifically designed for security training.

Do **not** apply the techniques, payloads or testing methodology documented here against systems that you do not own or do not have explicit authorization to test.

All testing documented in this repository is performed against a controlled Juice Shop environment.

---

# 🧪 Target Application

| Property         | Details                                   |
| ---------------- | ----------------------------------------- |
| Application      | OWASP Juice Shop                          |
| Assessment Type  | Web Application VAPT                      |
| Environment      | Local / Authorized Lab                    |
| Application Type | Vulnerable E-Commerce Web Application     |
| Frontend         | Angular                                   |
| Backend          | Node.js / Express                         |
| APIs             | REST APIs                                 |
| Database         | SQLite                                    |
| Testing Approach | Manual + Tool-Assisted                    |
| Primary Proxy    | Burp Suite                                |
| OS               | Kali Linux / Windows                      |
| Evidence         | Screenshots, HTTP Requests/Responses, PoC |
| Status           | In Progress                               |

---

# 🏗️ VAPT Methodology

The assessment follows a structured penetration-testing workflow.

```text
                    ┌─────────────────────┐
                    │   Target Definition │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Reconnaissance      │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Enumeration         │
                    │ Endpoints / APIs    │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Attack Surface      │
                    │ Mapping             │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Vulnerability       │
                    │ Identification      │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Manual Validation   │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Exploitation / PoC  │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Evidence Collection │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Risk Assessment     │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Remediation         │
                    └─────────────────────┘
```

---

# 📂 Repository Structure

```text
OWASP-Juice-Shop-VAPT/
│
├── README.md
│
├── 01-Reconnaissance/
│   └── README.md
│
├── 02-Information-Disclosure/
│   └── README.md
│
├── 03-Authentication/
│   └── README.md
│
├── 04-Broken-Access-Control/
│   └── README.md
│
├── 05-Injection/
│   └── README.md
│
├── 06-Cross-Site-Scripting/
│   └── README.md
│
├── 07-CSRF/
│   └── README.md
│
├── 08-Business-Logic/
│   └── README.md
│
├── 09-Error-Handling/
│   └── README.md
│
└── 10-HTTP-Security/
    └── README.md
```

---

# 🧭 Assessment Navigation

## 01. Reconnaissance

Initial information gathering and attack-surface discovery.

Topics include:

* Target identification
* Technology fingerprinting
* Application mapping
* Directory and endpoint discovery
* API discovery
* JavaScript analysis
* Robots.txt
* Sitemap
* Security.txt
* HTTP methods
* Parameters
* Authentication endpoints
* API endpoints
* Client-side routes
* Interesting files
* Burp Suite proxy configuration

➡️ **[Open Reconnaissance](./01-Reconnaissance/)**

---

## 02. Information Disclosure

Testing for unintended exposure of sensitive application information.

Topics include:

* Sensitive files
* Exposed credentials
* Debug information
* Application metadata
* API responses
* Source-code references
* JavaScript information
* Backup files
* Configuration information
* Exposed endpoints
* Excessive data returned by APIs
* Security-related metadata

➡️ **[Open Information Disclosure](./02-Information-Disclosure/)**

---

## 03. Authentication

Testing authentication mechanisms and account-management functionality.

Topics include:

* Login functionality
* Authentication bypass
* Weak authentication controls
* Password policy
* Password reset
* Security questions
* Account enumeration
* Session handling
* Authentication tokens
* Brute-force protections
* Credential-related weaknesses

Example Juice Shop challenges may include:

* Login Admin
* Password Strength
* Bjoern's Favorite Pet
* Reset Password challenges
* Login-related injection challenges

➡️ **[Open Authentication](./03-Authentication/)**

---

## 04. Broken Access Control

Testing whether users can access resources or perform actions beyond their authorization level.

Topics include:

* Horizontal privilege escalation
* Vertical privilege escalation
* IDOR
* BOLA
* Unauthorized API access
* Administrative functionality
* Basket manipulation
* User-data access
* Object-level authorization
* Forced browsing
* Privilege boundary testing

Example areas:

* View Basket
* Admin Section
* Forged Review
* Forged Feedback
* Product Tampering
* Manipulate Basket

➡️ **[Open Broken Access Control](./04-Broken-Access-Control/)**

---

## 05. Injection

Testing whether untrusted input can alter application commands, queries or processing logic.

Topics include:

* SQL Injection
* NoSQL Injection
* Authentication bypass
* Command injection
* LDAP-style injection concepts
* Input validation
* Server-side injection
* API parameter manipulation
* SQLMap-assisted validation
* Manual injection testing

Testing workflow:

```text
Identify Input
      ↓
Intercept Request
      ↓
Modify Parameter
      ↓
Inject Test Input
      ↓
Observe Response
      ↓
Determine Behavior
      ↓
Validate Manually
      ↓
Document Impact
```

➡️ **[Open Injection](./05-Injection/)**

---

## 06. Cross-Site Scripting

Testing whether attacker-controlled input can execute JavaScript in a victim's browser.

Testing categories:

* Reflected XSS
* Stored XSS
* DOM XSS
* API-based XSS
* HTML injection
* Client-side sinks
* Source-to-sink analysis
* Output encoding
* Input sanitization

Testing workflow:

```text
Find Input
   ↓
Inject Controlled Payload
   ↓
Submit Request
   ↓
Observe Reflection / Storage
   ↓
Check Browser Execution
   ↓
Identify XSS Type
   ↓
Capture Evidence
   ↓
Recommend Fix
```

Tools commonly used:

* Burp Suite
* Browser DevTools
* JavaScript source inspection
* OWASP ZAP

➡️ **[Open Cross-Site Scripting](./06-Cross-Site-Scripting/)**

---

## 07. CSRF

Testing whether an attacker can cause an authenticated user to perform an unwanted state-changing action.

Testing areas:

* State-changing requests
* CSRF tokens
* SameSite cookie configuration
* Origin validation
* Referer validation
* GET-based state changes
* POST-based state changes
* Authentication/session context

Testing methodology:

```text
Identify State-Changing Request
            ↓
Check CSRF Token
            ↓
Remove / Modify Token
            ↓
Replay Request
            ↓
Observe Server Behavior
            ↓
Determine Whether Action Succeeds
```

➡️ **[Open CSRF](./07-CSRF/)**

---

## 08. Business Logic

Testing application functionality for logic flaws that may not be detected by automated scanners.

Testing areas:

* Price manipulation
* Quantity manipulation
* Coupon abuse
* Workflow bypass
* Negative values
* Unexpected state transitions
* Race conditions
* Trust-boundary violations
* Client-side validation bypass
* Server-side business-rule validation
* Payment/order logic

Business-logic testing focuses on:

> **What the application allows a legitimate user to do versus what the business should actually allow.**

➡️ **[Open Business Logic](./08-Business-Logic/)**

---

## 09. Error Handling

Testing application behavior when unexpected, malformed or invalid input is supplied.

Testing areas:

* Verbose error messages
* Stack traces
* Database errors
* API errors
* Debug information
* Invalid parameters
* Malformed JSON
* Invalid HTTP methods
* Unexpected HTTP responses
* Error-page information disclosure

The objective is to determine whether error responses reveal:

* Technology information
* File paths
* Database information
* Internal endpoints
* Framework details
* Debug information
* Sensitive application logic

➡️ **[Open Error Handling](./09-Error-Handling/)**

---

## 10. HTTP Security

Testing HTTP behavior and security controls implemented through requests, responses, cookies and security headers.

Topics include:

* HTTP methods
* HTTP status codes
* Security headers
* Cookies
* Session cookies
* CORS
* CSP
* HSTS
* X-Content-Type-Options
* Referrer-Policy
* Permissions-Policy
* Cache-Control
* HTTP request manipulation
* HTTP response analysis
* Host header behavior
* Content-Type handling

Important headers reviewed:

```text
Content-Security-Policy
Strict-Transport-Security
X-Content-Type-Options
X-Frame-Options
Referrer-Policy
Permissions-Policy
Access-Control-Allow-Origin
Access-Control-Allow-Credentials
Cache-Control
Set-Cookie
```

➡️ **[Open HTTP Security](./10-HTTP-Security/)**

---

# 🛠️ Tools Used

| Tool             | Purpose                                                      |
| ---------------- | ------------------------------------------------------------ |
| Burp Suite       | Proxy, interception, request manipulation and manual testing |
| OWASP ZAP        | Web application security testing                             |
| Nmap             | Network/service enumeration                                  |
| Nuclei           | Template-based vulnerability checks                          |
| SQLMap           | SQL injection validation                                     |
| Browser DevTools | Client-side analysis                                         |
| curl             | HTTP request testing                                         |
| jq               | JSON response analysis                                       |
| ffuf             | Endpoint/content discovery                                   |
| WhatWeb          | Technology fingerprinting                                    |
| Git/GitHub       | Documentation and version control                            |
| Kali Linux       | Security testing environment                                 |

> Tools are used to support manual validation. Automated scanner output is not treated as a confirmed vulnerability until manually reviewed.

---

# 🔎 Evidence Collection Method

Each confirmed finding should contain reproducible evidence.

Recommended evidence format:

```text
Finding
│
├── Vulnerability Name
├── Category
├── Severity
├── CWE
├── OWASP Mapping
├── Affected Endpoint
├── Affected Parameter
├── Preconditions
├── Testing Method
├── Request
├── Response
├── Proof of Concept
├── Screenshot
├── Impact
├── Root Cause
└── Remediation
```

---

# 📋 Vulnerability Reporting Format

Each vulnerability should follow a consistent report format.

### Finding Title

Example:

```text
Stored Cross-Site Scripting in Product Review Functionality
```

### Severity

```text
High / Medium / Low / Informational
```

### Category

```text
OWASP Top 10 / CWE
```

### Affected Endpoint

```text
/api/...
```

### Parameter

```text
parameter_name
```

### Description

Explain what the vulnerability is and why it occurs.

### Steps to Reproduce

1. Open the affected functionality.
2. Intercept the request using Burp Suite.
3. Identify the vulnerable parameter.
4. Modify the parameter.
5. Send the request.
6. Observe the response/application behavior.
7. Confirm the security impact.

### Evidence

Add:

* Burp request
* Burp response
* Browser screenshot
* Relevant application output

### Impact

Explain what an attacker could achieve if the vulnerability existed in a real application.

### Remediation

Provide a practical developer-focused recommendation.

---

# 📊 Risk Classification

The assessment uses the following severity categories:

| Severity         | Meaning                                                       |
| ---------------- | ------------------------------------------------------------- |
| 🔴 Critical      | Severe compromise with major business/security impact         |
| 🟠 High          | Significant security impact requiring urgent remediation      |
| 🟡 Medium        | Moderate impact or exploitation requiring specific conditions |
| 🟢 Low           | Limited security impact                                       |
| 🔵 Informational | Security observation without direct exploitable impact        |

Severity should be based on factors such as:

* Exploitability
* Impact
* Authentication requirement
* Required privileges
* User interaction
* Data exposure
* Confidentiality impact
* Integrity impact
* Availability impact

---

# 🧪 Manual VAPT Approach

This project emphasizes manual testing instead of relying only on automated scanners.

## Phase 1 – Reconnaissance

```text
Application
     ↓
Technology
     ↓
Routes
     ↓
Endpoints
     ↓
Parameters
     ↓
APIs
     ↓
Attack Surface
```

## Phase 2 – Enumeration

Identify:

* Users
* Roles
* APIs
* Parameters
* Objects
* Authentication mechanisms
* Administrative functionality
* Client-side routes
* Hidden functionality

## Phase 3 – Vulnerability Testing

Test:

```text
Authentication
Authorization
Input Validation
Session Management
Business Logic
API Security
Client-Side Security
Server-Side Security
HTTP Security
Error Handling
```

## Phase 4 – Validation

Every suspected vulnerability should be manually validated.

```text
Scanner Finding
      ↓
Manual Verification
      ↓
Reproduce
      ↓
Determine Impact
      ↓
Confirmed Finding
```

## Phase 5 – Documentation

Capture:

* Request
* Response
* Payload/test input
* Screenshot
* Endpoint
* Parameter
* Impact
* Remediation

---

# 🗂️ Finding Documentation Template

Each individual vulnerability can use the following structure:

````markdown
# Vulnerability Name

## Severity

Medium

## Category

OWASP Top 10

## Affected Component

Endpoint / Functionality

## Description

Explain the vulnerability.

## Preconditions

Mention required authentication, role or setup.

## Steps to Reproduce

1. Step one
2. Step two
3. Step three

## HTTP Request

```http
GET /api/example HTTP/1.1
Host: localhost:3000
...
````

## HTTP Response

```http
HTTP/1.1 200 OK
...
```

## Proof of Concept

Describe the controlled PoC.

## Evidence

Add screenshots here.

## Impact

Explain realistic security impact.

## Root Cause

Explain why the vulnerability exists.

## Remediation

Explain how developers can fix the issue.

## References

* OWASP
* CWE
* Relevant security documentation

````

---

# 🧩 OWASP Juice Shop Challenge Coverage

Juice Shop contains challenges across multiple security categories, including Broken Access Control, Broken Authentication, Injection, XSS, Sensitive Data Exposure, Security Misconfiguration, Unvalidated Redirects, Vulnerable Components and other categories.

The official project documentation also provides a challenge catalogue and an official companion guide containing explanations, hints and solutions.

This repository does **not** simply document challenge completion. The goal is to demonstrate the underlying **security testing methodology**.

---

# 📚 Recommended Learning References

## OWASP Juice Shop

Official project:

https://owasp-juice.shop/

OWASP project page:

https://owasp.org/www-project-juice-shop/

Official source repository:

https://github.com/juice-shop/juice-shop

Official Companion Guide:

https://pwning.owasp-juice.shop/

---

## OWASP Top 10

https://owasp.org/www-project-top-ten/

Use the OWASP Top 10 to understand common web application security risks and map relevant findings.

---

## OWASP Web Security Testing Guide

https://owasp.org/www-project-web-security-testing-guide/

Useful for understanding a structured web application penetration-testing methodology.

---

## OWASP API Security

https://owasp.org/www-project-api-security/

Useful when testing Juice Shop REST APIs, authorization and object-level access controls.

---

## CWE

https://cwe.mitre.org/

Useful for mapping vulnerabilities to standardized weakness classifications.

---

# 🧰 Suggested Burp Suite Workflow

```text
Browser
   │
   ▼
Burp Proxy
   │
   ├── HTTP History
   │
   ├── Repeater
   │
   ├── Intruder
   │
   ├── Decoder
   │
   └── Comparer
   │
   ▼
Juice Shop
````

### Proxy

Use Proxy to intercept and inspect application traffic.

### HTTP History

Use HTTP History to understand:

* Endpoints
* Parameters
* API calls
* Authentication requests
* Cookies
* Headers

### Repeater

Use Repeater for controlled manual testing and request modification.

### Intruder

Use Intruder where controlled automated parameter testing is appropriate within the lab.

### Decoder

Use Decoder for encoding/decoding values encountered during testing.

---

# 📝 Example Assessment Workflow

```text
01. Start Juice Shop
        ↓
02. Configure Burp Suite
        ↓
03. Browse Application
        ↓
04. Capture Requests
        ↓
05. Identify Endpoints
        ↓
06. Map Parameters
        ↓
07. Identify Interesting Functions
        ↓
08. Test Authentication
        ↓
09. Test Authorization
        ↓
10. Test Input Validation
        ↓
11. Test Injection
        ↓
12. Test XSS
        ↓
13. Test CSRF
        ↓
14. Test Business Logic
        ↓
15. Review HTTP Security
        ↓
16. Validate Findings
        ↓
17. Capture Evidence
        ↓
18. Assign Severity
        ↓
19. Recommend Remediation
        ↓
20. Prepare Final Report
```

---

# 📈 Assessment Progress

| Category               | Status         |
| ---------------------- | -------------- |
| Reconnaissance         | 🟡 In Progress |
| Information Disclosure | 🟡 In Progress |
| Authentication         | 🟡 In Progress |
| Broken Access Control  | 🟡 In Progress |
| Injection              | 🟡 In Progress |
| Cross-Site Scripting   | 🟡 In Progress |
| CSRF                   | 🟡 In Progress |
| Business Logic         | 🟡 In Progress |
| Error Handling         | 🟡 In Progress |
| HTTP Security          | 🟡 In Progress |

Update these statuses as findings are completed.

Recommended status values:

```text
⬜ Not Started
🟡 In Progress
🟢 Completed
🔴 Needs Review
```

---

# 📸 Evidence Standards

Screenshots should clearly show the relevant security evidence without exposing unnecessary personal information.

Recommended evidence:

```text
Evidence/
│
├── 01-Recon/
├── 02-Information-Disclosure/
├── 03-Authentication/
├── 04-Broken-Access-Control/
├── 05-Injection/
├── 06-XSS/
├── 07-CSRF/
├── 08-Business-Logic/
├── 09-Error-Handling/
└── 10-HTTP-Security/
```

Where possible, include:

* Vulnerable request
* Modified request
* Server response
* Browser result
* Before/after comparison
* Relevant Burp Suite evidence

---

# 🎓 Skills Demonstrated

This project demonstrates practical experience with:

### Web Application Security

* Reconnaissance
* Enumeration
* Vulnerability discovery
* Manual exploitation
* Vulnerability validation
* Security testing
* Risk assessment
* Remediation analysis

### OWASP

* OWASP Top 10
* OWASP Web Security Testing Guide
* OWASP API Security concepts
* Common web vulnerability classes

### Technical Skills

* HTTP/HTTPS
* REST APIs
* Cookies
* Sessions
* Authentication
* Authorization
* JavaScript
* JSON
* HTTP headers
* Browser DevTools
* Burp Suite

### Security Tools

* Burp Suite
* OWASP ZAP
* Nmap
* Nuclei
* SQLMap
* ffuf
* curl
* Kali Linux

---

# 🚀 Future Improvements

Planned improvements for this repository:

* [ ] Complete all 10 assessment categories
* [ ] Add detailed vulnerability findings
* [ ] Add Burp Suite evidence
* [ ] Add screenshots
* [ ] Add CVSS scoring
* [ ] Add CWE mapping
* [ ] Add OWASP mapping
* [ ] Add remediation references
* [ ] Add API testing section
* [ ] Add authentication testing matrix
* [ ] Add authorization testing matrix
* [ ] Add final executive summary
* [ ] Add consolidated vulnerability table
* [ ] Add final VAPT report
* [ ] Add remediation verification
* [ ] Add before/after validation

---

# 📊 Final Report Structure

Once testing is completed, the repository can be expanded into a complete professional VAPT report:

```text
Executive Summary
        ↓
Scope & Methodology
        ↓
Target Information
        ↓
Attack Surface
        ↓
Vulnerability Summary
        ↓
Detailed Findings
        ↓
Evidence
        ↓
Risk Ratings
        ↓
Business Impact
        ↓
Remediation
        ↓
Retest Results
        ↓
Conclusion
```

---

# 🎯 Project Outcome

The goal of this project is to demonstrate the complete lifecycle of a web application security assessment rather than simply solving individual Juice Shop challenges.

The final repository should allow a reviewer to understand:

> **What was tested → How it was tested → What was discovered → How it was validated → What impact it has → How it can be fixed**

This makes the project suitable as a practical **Web Application VAPT / Application Security portfolio project**.

---

# 👩‍💻 Author

**Bhumika Mishra**

Cybersecurity / VAPT Enthusiast

GitHub:
[Bhumika02052002](https://github.com/Bhumika02052002)

VAPT Portfolio Repository:
[OWASP-Juice-Shop-VAPT](https://github.com/Bhumika02052002/OWASP-Juice-Shop-VAPT)

---

# ⭐ Disclaimer

This project is created for **authorized educational security testing** using OWASP Juice Shop, an intentionally vulnerable application.

The techniques demonstrated in this repository should only be used against systems for which you have explicit authorization.

---

## 🔗 Official References

* [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/)
* [OWASP Juice Shop Official Website](https://owasp-juice.shop/)
* [OWASP Juice Shop GitHub](https://github.com/juice-shop/juice-shop)
* [OWASP Juice Shop Companion Guide](https://pwning.owasp-juice.shop/)
* [OWASP Top 10](https://owasp.org/www-project-top-ten/)
* [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
* [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)
* [MITRE CWE](https://cwe.mitre.org/)
