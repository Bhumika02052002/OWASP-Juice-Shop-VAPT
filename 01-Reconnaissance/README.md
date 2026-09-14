# 01 – Reconnaissance

> **OWASP Juice Shop Web Application VAPT – Professional Reconnaissance Methodology**

Reconnaissance is the first phase of a Web Application Vulnerability Assessment and Penetration Testing (VAPT) engagement.

The objective is to collect information about the target **before vulnerability exploitation**.

A professional reconnaissance process attempts to understand:

* Target scope
* Domain and subdomain structure
* DNS records
* IP addresses
* Open ports
* Running services
* Technology stack
* Web server
* WAF/CDN
* Directories and files
* Application routes
* API endpoints
* JavaScript assets
* Parameters
* HTTP methods
* Authentication boundaries
* Cookies and session mechanisms
* Publicly accessible resources
* Historical URLs
* Potential attack surfaces

The results are documented as an **attack-surface inventory** and used by the subsequent VAPT phases.

---

# 🗺️ Professional Reconnaissance Methodology

```text
                         TARGET
                           │
                           ▼
                  Scope Identification
                           │
                           ▼
                 Passive Reconnaissance
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
          WHOIS          DNS          OSINT
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                  Subdomain Enumeration
                           │
                           ▼
                    DNS Resolution
                           │
                           ▼
                  IP/Host Enumeration
                           │
                           ▼
                    Port Enumeration
                           │
                           ▼
                 Service Enumeration
                           │
                           ▼
               Technology Fingerprinting
                           │
                           ▼
                     WAF Detection
                           │
                           ▼
                 Web Server Analysis
                           │
                           ▼
                Directory/File Discovery
                           │
                           ▼
                   URL Enumeration
                           │
                           ▼
                JavaScript Enumeration
                           │
                           ▼
                  API Discovery
                           │
                           ▼
                Parameter Discovery
                           │
                           ▼
                Authentication Mapping
                           │
                           ▼
                   Burp Suite Mapping
                           │
                           ▼
                  Attack Surface Map
                           │
                           ▼
                    Evidence Collection
```

---

# 🎯 Reconnaissance Objectives

The objectives of this phase are:

* Identify the target.
* Confirm the authorized scope.
* Identify domains and subdomains where applicable.
* Identify DNS information.
* Identify IP addresses.
* Identify open ports.
* Identify running services.
* Identify web technologies.
* Identify WAF/CDN infrastructure.
* Discover directories and files.
* Discover historical URLs.
* Identify JavaScript assets.
* Discover API endpoints.
* Identify parameters.
* Identify authentication endpoints.
* Identify cookies and sessions.
* Identify HTTP methods.
* Identify interesting headers.
* Build an endpoint inventory.
* Build a parameter inventory.
* Collect reproducible evidence.

---

# ⚠️ Scope & Authorization

This project uses **OWASP Juice Shop**, an intentionally vulnerable web application designed for security education and authorized testing.

The primary target for this project is a local laboratory instance:

```text
http://127.0.0.1:3000
```

Target variable:

```bash
export TARGET="http://127.0.0.1:3000"
```

For a remote authorized laboratory:

```bash
export TARGET="http://<TARGET-IP>:3000"
```

For real-world engagements, replace the target with the domain explicitly included in the engagement scope.

> **Important:** Passive and active reconnaissance techniques must only be performed against assets that are explicitly authorized.

---

# 🧰 Reconnaissance Toolkit

The following tools can be used depending on the engagement scope.

| Category            | Tools                               |
| ------------------- | ----------------------------------- |
| Identity            | `whois`                             |
| DNS                 | `dig`, `nslookup`, `dnsenum`        |
| Subdomains          | `subfinder`, `assetfinder`, `amass` |
| Host discovery      | `httpx`                             |
| Port scanning       | `nmap`                              |
| Technology          | `WhatWeb`, `Wappalyzer`             |
| WAF                 | `wafw00f`                           |
| Web crawling        | `katana`                            |
| URL discovery       | `gau`, `waybackurls`                |
| Directory discovery | `ffuf`, `gobuster`                  |
| Content discovery   | `ffuf`, `feroxbuster`               |
| JavaScript          | `grep`, `linkfinder`                |
| HTTP analysis       | `curl`                              |
| Proxy               | Burp Suite                          |
| Automated checks    | Nuclei                              |
| Browser analysis    | Chrome/Firefox DevTools             |

---

# 1. Target Identification

Create a target variable:

```bash
export TARGET="http://127.0.0.1:3000"
```

Check connectivity:

```bash
curl -I "$TARGET"
```

Check HTTP status:

```bash
curl -s -o /dev/null -w "%{http_code}\n" "$TARGET"
```

Check response size:

```bash
curl -s -o /dev/null -w "%{size_download}\n" "$TARGET"
```

Check response time:

```bash
curl -s -o /dev/null \
-w "Status: %{http_code}\nTime: %{time_total}s\n" \
"$TARGET"
```

Record:

```text
Target:
Protocol:
Host:
Port:
Status:
Response Time:
```

---

# 2. WHOIS Reconnaissance

## Purpose

WHOIS can provide registration information for publicly registered domains.

Typical information can include:

* Registrar
* Registration dates
* Nameservers
* Domain status
* Registry information

Example:

```bash
whois example.com
```

Search specific information:

```bash
whois example.com | grep -Ei \
"Registrar|Creation|Expiration|Name Server|Status"
```

### Juice Shop Lab

For:

```text
127.0.0.1
```

WHOIS is **not applicable**.

Document:

```text
WHOIS Status: N/A
Reason: Target is a local IP/laboratory instance.
```

This is better professional documentation than running irrelevant commands and claiming results.

---

# 3. DNS Reconnaissance

DNS reconnaissance is useful when the target is a domain.

Example:

```bash
dig example.com
```

A record:

```bash
dig example.com A
```

AAAA record:

```bash
dig example.com AAAA
```

MX record:

```bash
dig example.com MX
```

NS record:

```bash
dig example.com NS
```

TXT record:

```bash
dig example.com TXT
```

SOA:

```bash
dig example.com SOA
```

All common records:

```bash
dig example.com ANY
```

> Some DNS servers restrict or refuse `ANY` queries.

---

# 4. DNS Enumeration

Using `dnsenum`:

```bash
dnsenum example.com
```

Possible information:

```text
A records
MX records
NS records
Subdomains
Nameservers
DNS information
```

For the local Juice Shop:

```text
DNS Enumeration: N/A
```

because the primary target is:

```text
127.0.0.1:3000
```

---

# 5. Subdomain Enumeration

Subdomain enumeration is primarily applicable to domain-based targets.

Examples:

```text
example.com
api.example.com
dev.example.com
staging.example.com
admin.example.com
```

---

## Subfinder

```bash
subfinder -d example.com -silent
```

Save results:

```bash
subfinder -d example.com -silent \
-o subdomains.txt
```

---

## Assetfinder

```bash
assetfinder --subs-only example.com
```

Save:

```bash
assetfinder --subs-only example.com \
> assetfinder-subdomains.txt
```

---

## Amass

Passive enumeration:

```bash
amass enum -passive -d example.com
```

Save:

```bash
amass enum -passive \
-d example.com \
-o amass-subdomains.txt
```

---

# 6. Combine & Deduplicate Subdomains

Combine results:

```bash
cat subdomains.txt \
assetfinder-subdomains.txt \
amass-subdomains.txt \
| sort -u > all-subdomains.txt
```

Count results:

```bash
wc -l all-subdomains.txt
```

View:

```bash
cat all-subdomains.txt
```

---

# 7. Subdomain Takeover Recon

During an authorized assessment, potentially dangling DNS records may be identified.

First inspect DNS:

```bash
dig CNAME subdomain.example.com
```

Check resolution:

```bash
dig subdomain.example.com
```

Record:

```text
Subdomain
CNAME
Resolved IP
Provider
HTTP Response
```

> A dangling CNAME is an observation, not automatically a confirmed takeover vulnerability. Verification requires provider-specific validation.

---

# 8. DNS Resolution of Hosts

Resolve discovered hosts:

```bash
for host in $(cat all-subdomains.txt); do
    echo "===== $host ====="
    dig +short "$host"
done
```

Save:

```bash
for host in $(cat all-subdomains.txt); do
    dig +short "$host"
done | sort -u > resolved-ips.txt
```

---

# 9. HTTP Host Discovery

Use `httpx` to identify live web services:

```bash
httpx -l all-subdomains.txt
```

More detailed:

```bash
httpx \
-l all-subdomains.txt \
-status-code \
-title \
-tech-detect \
-web-server
```

Save:

```bash
httpx \
-l all-subdomains.txt \
-status-code \
-title \
-tech-detect \
-web-server \
-o live-hosts.txt
```

---

# 10. IP Address Enumeration

For a known hostname:

```bash
dig +short example.com
```

For local Juice Shop:

```bash
hostname -I
```

Loopback:

```bash
ip addr show lo
```

Record:

```text
Hostname
IP Address
Port
Protocol
```

---

# 11. Port Scanning

## Basic Scan

```bash
nmap -Pn <TARGET-IP>
```

## Common Web Ports

```bash
nmap -Pn \
-p 80,443,3000,5000,8000,8080,8443 \
<TARGET-IP>
```

## Service Detection

```bash
nmap -Pn -sV \
-p 3000 \
<TARGET-IP>
```

## Default Scripts

```bash
nmap -Pn -sC -sV \
-p 3000 \
<TARGET-IP>
```

## Output to File

```bash
nmap -Pn -sC -sV \
-p 3000 \
<TARGET-IP> \
-oN nmap-service.txt
```

---

# 12. Full TCP Port Enumeration

For an authorized target where a full port assessment is in scope:

```bash
nmap -Pn -p- <TARGET-IP>
```

Then perform service detection only on discovered ports:

```bash
nmap -Pn -sV \
-p <DISCOVERED-PORTS> \
<TARGET-IP>
```

Example:

```text
80,443,3000,8080
```

---

# 13. Technology Fingerprinting

## WhatWeb

```bash
whatweb "$TARGET"
```

Detailed:

```bash
whatweb -a 3 "$TARGET"
```

Record:

```text
Frontend Framework
Backend Framework
Web Server
JavaScript
Cookies
Libraries
Technology Indicators
```

---

# 14. Browser Technology Fingerprinting

Use:

```text
Browser
↓
F12
↓
Network
↓
Response Headers
```

Look for:

```text
Server
X-Powered-By
Content-Type
Set-Cookie
Framework indicators
JavaScript libraries
```

---

# 15. WAF Detection

Use `wafw00f`:

```bash
wafw00f "$TARGET"
```

Example:

```bash
wafw00f http://example.com
```

Record:

```text
WAF Detected:
Provider:
Evidence:
```

For local Juice Shop:

```text
WAF: Not expected / N/A
```

Do not report absence of a WAF as a vulnerability by itself.

---

# 16. HTTP Header Enumeration

Inspect headers:

```bash
curl -I "$TARGET"
```

Full response:

```bash
curl -i "$TARGET"
```

Save:

```bash
curl -I "$TARGET" \
> evidence/04-http/headers.txt
```

Record:

```text
Server
Content-Type
Content-Length
Set-Cookie
Location
Cache-Control
CSP
HSTS
CORS
X-Content-Type-Options
X-Frame-Options
Referrer-Policy
```

---

# 17. HTTP Methods

Check OPTIONS:

```bash
curl -i -X OPTIONS "$TARGET"
```

Record:

```text
Allow:
Access-Control-Allow-Methods:
```

Observed methods may include:

```text
GET
POST
PUT
PATCH
DELETE
OPTIONS
HEAD
```

The existence of a method does not automatically indicate a vulnerability.

---

# 18. Redirect Enumeration

Check redirects:

```bash
curl -I "$TARGET"
```

Follow redirects:

```bash
curl -I -L "$TARGET"
```

Record:

```text
Original URL
Redirect Status
Location
Final URL
```

---

# 19. Directory & File Discovery

Directory discovery identifies resources that may not be linked from the main application.

---

## FFUF

```bash


ffuf \
-u "$TARGET/FUZZ" \
-w /usr/share/wordlists/dirb/common.txt \
-mc 200,204,301,302,307,401,403
```

Save:

```bash
ffuf \
-u "$TARGET/FUZZ" \
-w /usr/share/wordlists/dirb/common.txt \
-mc 200,204,301,302,307,401,403 \
-o ffuf-results.json \
-of json
```

---

# 20. FFUF File Extension Discovery

```bash
ffuf \
-u "$TARGET/FUZZ" \
-w /usr/share/wordlists/dirb/common.txt \
-e .js,.json,.txt,.xml,.html
```

Useful for identifying:

```text
JavaScript
JSON
XML
Text
HTML
```

---

# 21. Gobuster Directory Discovery

```bash
gobuster dir \
-u "$TARGET" \
-w /usr/share/wordlists/dirb/common.txt
```

With extensions:

```bash
gobuster dir \
-u "$TARGET" \
-w /usr/share/wordlists/dirb/common.txt \
-x js,json,txt,xml
```

Save:

```bash
gobuster dir \
-u "$TARGET" \
-w /usr/share/wordlists/dirb/common.txt \
-o gobuster.txt
```

---

# 22. Feroxbuster

Alternative content discovery tool:

```bash
feroxbuster \
-u "$TARGET" \
-w /usr/share/wordlists/dirb/common.txt
```

Use one primary tool consistently and document the others as alternatives.

---

# 23. Important Public Files

Check:

```bash
curl -i "$TARGET/robots.txt"
```

```bash
curl -i "$TARGET/sitemap.xml"
```

```bash
curl -i "$TARGET/security.txt"
```

```bash
curl -i "$TARGET/.well-known/security.txt"
```

```bash
curl -i "$TARGET/manifest.json"
```

```bash
curl -i "$TARGET/favicon.ico"
```

---

# 24. Common Sensitive File Recon

In an authorized lab, check whether common resources are exposed:

```text
.env
.git/
.git/config
package.json
package-lock.json
robots.txt
sitemap.xml
backup files
configuration files
debug files
source maps
```

Example status check:

```bash
for path in \
.env \
.git/config \
package.json \
package-lock.json \
robots.txt \
sitemap.xml \
manifest.json; do

    echo "===== $path ====="

    curl -s -o /dev/null \
    -w "%{http_code} %{size_download}\n" \
    "$TARGET/$path"

done
```

> A `200` response is an observation. The content must be reviewed to determine whether sensitive information is actually exposed.

---

# 25. Web Crawling

Use `katana` to crawl the application.

```bash
katana \
-u "$TARGET"
```

Save results:

```bash
katana \
-u "$TARGET" \
-o katana-urls.txt
```

JavaScript crawling:

```bash
katana \
-u "$TARGET" \
-js-crawl \
-o katana-js.txt
```

The crawl can help identify:

* Routes
* Links
* API paths
* JavaScript files
* Parameters
* Forms

---

# 26. Historical URL Discovery

Historical URLs can reveal old application functionality.

## GAU

```bash
gau example.com
```

Save:

```bash
gau example.com \
> gau-urls.txt
```

---

## Waybackurls

```bash
waybackurls example.com
```

Save:

```bash
waybackurls example.com \
> wayback-urls.txt
```

Combine:

```bash
cat gau-urls.txt \
wayback-urls.txt \
| sort -u > historical-urls.txt
```

> Historical URL discovery is applicable to public domains, not a local `127.0.0.1` Juice Shop instance.

---

# 27. URL Parameter Extraction

Extract URLs containing parameters:

```bash
grep "=" historical-urls.txt \
> parameterized-urls.txt
```

Example:

```text
/search?q=test
/product?id=1
/item?itemId=10
```

These URLs can become candidates for later:

```text
SQL Injection
XSS
IDOR
Business Logic
Input Validation
```

Do not classify them as vulnerabilities during reconnaissance.

---

# 28. JavaScript Enumeration

Find JavaScript files from crawling:

```bash
grep -Ei '\.js($|\?)' katana-urls.txt \
> javascript-files.txt
```

Search JavaScript references:

```bash
grep -RniE \
'api/|rest/|graphql|login|register|admin|token|password|user' \
.
```

Search for endpoint-related strings:

```bash
grep -RniE \
'api|rest|graphql|endpoint|route' \
.
```

---

# 29. JavaScript Source Map Recon

Look for source maps:

```text
.js.map
```

Search:

```bash
grep -Ei '\.js\.map' \
javascript-files.txt
```

Source maps may contain useful development information.

Document only what is actually exposed.

---

# 30. API Discovery

API discovery can be performed using:

```text
Burp HTTP History
+
Burp Site Map
+
Browser Network tab
+
JavaScript analysis
+
Katana
+
Application behavior
```

Search for:

```text
/api/
/rest/
/graphql
```

Also inspect actual requests rather than relying only on wordlists.

---

# 31. API Request Analysis

For each API request record:

```text
HTTP Method
Endpoint
Query Parameters
Path Parameters
Headers
Cookies
Authorization
Content-Type
Request Body
Response Code
Response Body
```

Example:

```http
POST /api/example HTTP/1.1
Host: 127.0.0.1:3000
Content-Type: application/json

{
    "example": "value"
}
```

---

# 32. Parameter Discovery

Identify parameters in:

### Query

```text
?id=
?q=
?search=
?item=
?quantity=
```

### Path

```text
/users/{id}
/products/{id}
/orders/{id}
```

### JSON

```json
{
  "id": 1,
  "email": "example@example.com",
  "quantity": 1
}
```

### Form

```text
username
password
email
comment
message
```

### Headers

```text
Authorization
Origin
Referer
Content-Type
Cookie
User-Agent
```

---

# 33. Authentication Reconnaissance

Identify:

```text
Login
Registration
Logout
Password Reset
Profile
Session
Token
Authorization
```

In Burp HTTP History search:

```text
login
register
authenticate
logout
password
token
session
```

Record:

```text
Authentication Endpoint
HTTP Method
Username/Email Parameter
Password Parameter
Cookie
Token
Response Code
Authentication Boundary
```

---

# 34. Cookie Reconnaissance

Inspect:

```http
Set-Cookie:
```

Record:

```text
Cookie Name
Secure
HttpOnly
SameSite
Path
Domain
Expiration
```

Example:

```bash
curl -I "$TARGET" | grep -i "set-cookie"
```

Cookie weaknesses should be validated during the HTTP Security phase.

---

# 35. CORS Reconnaissance

Inspect:

```text
Access-Control-Allow-Origin
Access-Control-Allow-Credentials
Access-Control-Allow-Methods
Access-Control-Allow-Headers
```

Baseline:

```bash
curl -i "$TARGET"
```

Authorized test:

```bash
curl -i \
-H "Origin: https://example.com" \
"$TARGET"
```

Document:

```text
Origin:
Allow-Origin:
Allow-Credentials:
Allow-Methods:
Allow-Headers:
```

Do not classify CORS as vulnerable solely because a CORS header exists.

---

# 36. Burp Suite Professional Workflow

Burp Suite should be the central tool for application-layer reconnaissance.

```text
Browser
   │
   ▼
Burp Proxy
   │
   ├── HTTP History
   │
   ├── Target / Site Map
   │
   ├── Repeater
   │
   └── Request/Response Analysis
           │
           ▼
      Endpoint Inventory
```

---

# 37. Burp Proxy Configuration

Browser proxy:

```text
Host: 127.0.0.1
Port: 8080
```

Open:

```text
Proxy → HTTP history
```

Browse:

```text
Homepage
Products
Search
Login
Register
Basket
Profile
Orders
Reviews
Feedback
```

---

# 38. Burp HTTP History

For each important request capture:

```text
Method
Host
Port
URL
Parameters
Headers
Cookies
Request Body
Response Status
Response Length
Response Content-Type
```

Useful filters:

```text
api
rest
login
register
admin
user
basket
order
search
```

---

# 39. Burp Site Map

Open:

```text
Target → Site map
```

Expected structure conceptually:

```text
Juice Shop
│
├── Application Routes
├── API
├── JavaScript
├── Static Assets
├── Authentication
└── Other Resources
```

Take a screenshot after the application has been sufficiently crawled.

---

# 40. Burp Repeater

Send interesting requests to Repeater:

```text
HTTP History
     ↓
Right Click
     ↓
Send to Repeater
     ↓
Inspect
     ↓
Modify Safe Input
     ↓
Send
     ↓
Compare Response
```

During reconnaissance, Repeater is useful for determining:

* Required parameters
* Optional parameters
* Authentication requirements
* Status code behavior
* Object identifiers
* API response structure
* Error behavior

---

# 41. Browser Developer Tools

Use:

```text
F12
```

Important tabs:

```text
Network
Sources
Application
Console
Elements
Security
```

### Network

Identify:

```text
API requests
XHR/fetch
HTTP methods
Parameters
JSON
Cookies
Authentication
```

### Sources

Identify:

```text
JavaScript
Routes
Libraries
Source maps
```

### Application

Review:

```text
Cookies
Local Storage
Session Storage
IndexedDB
```

---

# 42. Nuclei Reconnaissance

Nuclei can be used for controlled automated checks.

Basic:

```bash
nuclei -u "$TARGET"
```

Technology checks:

```bash
nuclei -u "$TARGET" -tags tech
```

Save results:

```bash
nuclei \
-u "$TARGET" \
-tags tech \
-o nuclei-tech.txt
```

Automated output must be manually verified.

---

# 43. Reconnaissance Automation

A simple workflow can collect basic information:

```bash
#!/bin/bash

TARGET="http://127.0.0.1:3000"

mkdir -p recon-output

echo "[+] Target"
curl -I "$TARGET" \
> recon-output/http-headers.txt

echo "[+] Technology"
whatweb "$TARGET" \
> recon-output/whatweb.txt

echo "[+] OPTIONS"
curl -i -X OPTIONS "$TARGET" \
> recon-output/options.txt

echo "[+] Robots"
curl -i "$TARGET/robots.txt" \
> recon-output/robots.txt

echo "[+] Sitemap"
curl -i "$TARGET/sitemap.xml" \
> recon-output/sitemap.xml

echo "[+] Nmap"
nmap -Pn -sC -sV -p 3000 127.0.0.1 \
-oN recon-output/nmap.txt

echo "[+] Recon completed"
```

This script is intended for the authorized Juice Shop laboratory.

---

# 44. Evidence Collection

Evidence should be reproducible.

Recommended structure:

```text
01-Reconnaissance/
│
├── README.md
│
├── evidence/
│   │
│   ├── 01-target/
│   │   ├── target-response.txt
│   │   └── target-screenshot.png
│   │
│   ├── 02-nmap/
│   │   ├── nmap-basic.txt
│   │   ├── nmap-service.txt
│   │   └── nmap-screenshot.png
│   │
│   ├── 03-technology/
│   │   ├── whatweb.txt
│   │   └── technology-screenshot.png
│   │
│   ├── 04-http/
│   │   ├── headers.txt
│   │   ├── options.txt
│   │   └── http-screenshot.png
│   │
│   ├── 05-burp/
│   │   ├── site-map.png
│   │   ├── http-history.png
│   │   ├── api-request.png
│   │   └── api-response.png
│   │
│   ├── 06-javascript/
│   │   ├── js-analysis.txt
│   │   └── javascript-screenshot.png
│   │
│   └── 07-discovery/
│       ├── robots.txt
│       ├── sitemap.xml
│       └── discovery-results.txt
│
├── endpoint-inventory.md
├── parameter-inventory.md
└── findings.md
```

---

# 45. Evidence Naming Convention

Use predictable names.

```text
01-target.png
02-nmap-service.png
03-whatweb.png
04-http-headers.png
05-burp-site-map.png
06-burp-http-history.png
07-api-request.png
08-api-response.png
09-javascript.png
10-directory-discovery.png
```

For multiple screenshots:

```text
burp-api-login-request.png
burp-api-login-response.png
burp-basket-request.png
burp-basket-response.png
```

---

# 46. Endpoint Inventory

Maintain an endpoint inventory throughout the assessment.

| ID    | Method | Endpoint | Auth | Parameters | Content-Type | Purpose            | Source  |
| ----- | ------ | -------- | ---- | ---------- | ------------ | ------------------ | ------- |
| E-001 | GET    | `/`      | No   | —          | HTML         | Homepage           | Browser |
| E-002 | GET    | `/...`   | No   | `id`       | JSON         | Data retrieval     | Burp    |
| E-003 | POST   | `/...`   | Yes  | JSON       | JSON         | Application action | Burp    |
| E-004 | GET    | `/...`   | Yes  | —          | JSON         | User data          | Burp    |

Replace placeholders with endpoints actually observed.

---

# 47. Parameter Inventory

| ID    | Endpoint | Parameter  | Location | Type    | Auth | Potential Testing |
| ----- | -------- | ---------- | -------- | ------- | ---- | ----------------- |
| P-001 | `/...`   | `id`       | Query    | Integer | No   | Access Control    |
| P-002 | `/...`   | `search`   | Query    | String  | No   | XSS/Injection     |
| P-003 | `/...`   | `email`    | JSON     | String  | Yes  | Authentication    |
| P-004 | `/...`   | `quantity` | JSON     | Integer | Yes  | Business Logic    |

---

# 48. Attack-Surface Classification

Classify discovered resources.

```text
                    Attack Surface
                          │
       ┌──────────────────┼──────────────────┐
       │                  │                  │
     Web UI              API             Authentication
       │                  │                  │
     Routes            Endpoints          Login
     Forms             Parameters         Register
     Search            Objects            Reset
       │                  │                  │
       └──────────────────┼──────────────────┘
                          │
       ┌──────────────────┼──────────────────┐
       │                  │                  │
     Client              HTTP             Files
       │                  │                  │
      JS               Headers          Robots
    Storage            Methods          Sitemap
    Routes             Cookies          Assets
```

---

# 49. Recon Findings Table

Record observations separately from confirmed vulnerabilities.

| ID    | Observation                        | Category       | Severity | Source  | Evidence   | Status    |
| ----- | ---------------------------------- | -------------- | -------- | ------- | ---------- | --------- |
| R-001 | Web service identified             | Infrastructure | Info     | Nmap    | Nmap       | Confirmed |
| R-002 | Technology identified              | Fingerprinting | Info     | WhatWeb | Screenshot | Confirmed |
| R-003 | API endpoint discovered            | API            | Info     | Burp    | Request    | Confirmed |
| R-004 | Authentication endpoint identified | Authentication | Info     | Burp    | Request    | Confirmed |
| R-005 | JavaScript bundle discovered       | Client-side    | Info     | Browser | Screenshot | Confirmed |
| R-006 | Parameter discovered               | Input Surface  | Info     | Burp    | Request    | Confirmed |
| R-007 | Public resource discovered         | Discovery      | Info     | curl    | Response   | Confirmed |

---

# 50. Recon-to-Vulnerability Mapping

Reconnaissance provides input to later phases.

| Recon Result         | Next Testing Area               |
| -------------------- | ------------------------------- |
| Login endpoint       | Authentication                  |
| Password reset       | Authentication                  |
| User ID              | Broken Access Control           |
| Basket ID            | Access Control / Business Logic |
| Search parameter     | XSS / Injection                 |
| JSON input           | Injection                       |
| Review functionality | XSS                             |
| API endpoint         | API Security                    |
| Object identifier    | IDOR/BOLA                       |
| File upload          | File Upload Security            |
| JavaScript route     | Client-Side Security            |
| Verbose response     | Information Disclosure          |
| HTTP headers         | HTTP Security                   |
| Error endpoint       | Error Handling                  |

---

# 51. Professional Recon Checklist

## Scope

* [ ] Scope documented
* [ ] Authorization confirmed
* [ ] Target documented
* [ ] Testing environment documented

## Passive Recon

* [ ] WHOIS performed where applicable
* [ ] DNS records identified
* [ ] Nameservers identified
* [ ] MX records identified
* [ ] TXT records identified
* [ ] Subdomains enumerated where applicable
* [ ] Historical URLs collected where applicable

## Infrastructure

* [ ] IP addresses identified
* [ ] Open ports identified
* [ ] Services identified
* [ ] Service versions identified
* [ ] Web ports checked

## Web Fingerprinting

* [ ] WhatWeb executed
* [ ] Server identified
* [ ] Framework identified
* [ ] WAF checked
* [ ] HTTP headers collected
* [ ] Cookies identified

## Content Discovery

* [ ] Directory enumeration performed
* [ ] File discovery performed
* [ ] robots.txt checked
* [ ] sitemap.xml checked
* [ ] security.txt checked
* [ ] manifest checked
* [ ] Source maps checked
* [ ] Interesting files reviewed

## Crawling

* [ ] Application crawled
* [ ] Routes identified
* [ ] Forms identified
* [ ] Parameters identified
* [ ] JavaScript identified
* [ ] API references identified

## API

* [ ] API endpoints documented
* [ ] HTTP methods documented
* [ ] Parameters documented
* [ ] Request bodies documented
* [ ] Response structures documented
* [ ] Authentication requirements documented
* [ ] Object identifiers documented

## Authentication

* [ ] Login mapped
* [ ] Registration mapped
* [ ] Logout mapped
* [ ] Password reset mapped
* [ ] Session mechanism identified
* [ ] Authentication tokens identified

## Burp

* [ ] Proxy configured
* [ ] HTTP History reviewed
* [ ] Site Map populated
* [ ] Interesting requests sent to Repeater
* [ ] API requests captured
* [ ] Evidence screenshots captured

## Documentation

* [ ] Endpoint inventory completed
* [ ] Parameter inventory completed
* [ ] Recon findings completed
* [ ] Attack surface mapped
* [ ] Evidence organized
* [ ] Next testing phase identified

---

# 52. Recon Evidence Quality Standard

Good VAPT evidence should answer:

```text
What did I discover?
        ↓
How did I discover it?
        ↓
What request produced it?
        ↓
What response was returned?
        ↓
Why is it relevant?
        ↓
What should be tested next?
```

Avoid uploading random screenshots.

Every important screenshot should have context.

Example:

```markdown
### Evidence – API Endpoint Discovery

The following request was observed in Burp Suite HTTP History while
using the application's product functionality.

![API Request](./evidence/05-burp/api-request.png)
```

---

# 53. Professional Recon Report Format

Each important observation can follow this format:

````markdown
## R-XXX – Observation Title

### Objective

What was being investigated?

### Method

Which tool or technique was used?

### Command

```bash
example-command
```

### Observation

What was discovered?

### Evidence

![Evidence](./evidence/example.png)

### Security Relevance

Why is this useful for the next testing phase?

### Next Step

What vulnerability category should be investigated next?

### Status

Confirmed Observation
````

---

# 54. Example End-to-End Recon Workflow

```bash
# Target
export TARGET="http://127.0.0.1:3000"

# 1. Connectivity
curl -I "$TARGET"

# 2. Technology
whatweb "$TARGET"

# 3. HTTP methods
curl -i -X OPTIONS "$TARGET"

# 4. Headers
curl -I "$TARGET"

# 5. Nmap
nmap -Pn -sC -sV -p 3000 127.0.0.1

# 6. Public resources
curl -i "$TARGET/robots.txt"
curl -i "$TARGET/sitemap.xml"
curl -i "$TARGET/.well-known/security.txt"

# 7. Directory discovery
ffuf \
-u "$TARGET/FUZZ" \
-w /usr/share/wordlists/dirb/common.txt \
-mc 200,204,301,302,307,401,403

# 8. Web crawling
katana \
-u "$TARGET" \
-js-crawl

# 9. Nuclei technology checks
nuclei \
-u "$TARGET" \
-tags tech
```

Then:

```text
Browser
   ↓
Burp Proxy
   ↓
HTTP History
   ↓
Site Map
   ↓
Application Crawling
   ↓
API Discovery
   ↓
JavaScript Analysis
   ↓
Parameter Identification
   ↓
Authentication Mapping
   ↓
Endpoint Inventory
   ↓
Evidence
```

For an internet-facing authorized domain, additionally:

```text
WHOIS
  ↓
DNS
  ↓
Subfinder
  ↓
Assetfinder
  ↓
Amass
  ↓
HTTPX
  ↓
Nmap
  ↓
WAF Detection
  ↓
Web Enumeration
```

---

# 55. Expected Recon Output

At the end of reconnaissance, the project should contain:

```text
Target Information
        +
Infrastructure Information
        +
Technology Fingerprint
        +
DNS/Subdomain Information
        +
Web Content
        +
Application Routes
        +
API Endpoints
        +
Parameters
        +
Authentication Surface
        +
JavaScript Intelligence
        +
HTTP Information
        +
Evidence
        =
Complete Attack-Surface Inventory
```

---

# 56. Reconnaissance Completion Criteria

Reconnaissance is complete when:

```text
☑ Scope documented
☑ Target identified
☑ Infrastructure mapped
☑ Ports identified
☑ Services identified
☑ Technologies identified
☑ WAF checked
☑ Web server identified
☑ Directories discovered
☑ Files discovered
☑ Application routes mapped
☑ JavaScript analyzed
☑ APIs identified
☑ Parameters identified
☑ Authentication mapped
☑ Cookies documented
☑ HTTP behavior documented
☑ Burp Site Map populated
☑ Endpoint inventory completed
☑ Parameter inventory completed
☑ Evidence captured
☑ Attack surface documented
```

---

# 📊 Final Recon Summary

| Category          | Result                                           |
| ----------------- | ------------------------------------------------ |
| Target            | OWASP Juice Shop                                 |
| Environment       | Authorized Laboratory                            |
| Target URL        | `http://127.0.0.1:3000`                          |
| Web Port          | 3000                                             |
| Frontend          | Angular                                          |
| Backend           | Node.js / Express                                |
| Network Recon     | Nmap                                             |
| Fingerprinting    | WhatWeb                                          |
| Web Proxy         | Burp Suite                                       |
| Content Discovery | FFUF / Gobuster                                  |
| Crawling          | Katana                                           |
| URL Discovery     | GAU / Waybackurls where applicable               |
| DNS               | dig / dnsenum where applicable                   |
| Subdomains        | Subfinder / Assetfinder / Amass where applicable |
| WAF               | wafw00f                                          |
| Automated Checks  | Nuclei                                           |
| API Discovery     | Burp / Browser / JavaScript                      |
| Parameters        | Inventoried                                      |
| Authentication    | Mapped                                           |
| Evidence          | `evidence/`                                      |
| Next Phase        | Information Disclosure                           |

---

# 📁 Project Structure

```text
01-Reconnaissance/
│
├── README.md
│
├── evidence/
│   ├── 01-target/
│   ├── 02-nmap/
│   ├── 03-technology/
│   ├── 04-http/
│   ├── 05-burp/
│   ├── 06-javascript/
│   └── 07-discovery/
│
├── endpoint-inventory.md
├── parameter-inventory.md
└── findings.md
```

---

# ➡️ Next Phase

After completing reconnaissance:

## [02 – Information Disclosure](../02-Information-Disclosure/)

The reconnaissance data will be used to investigate:

* Sensitive information exposure
* Verbose error messages
* Debug information
* Exposed files
* API information leakage
* Client-side information leakage
* Metadata
* Excessive API responses
* Credentials/secrets intentionally exposed by the training application

---

# 📚 Tools Used

This project may use:

* Nmap
* WhatWeb
* curl
* Burp Suite
* FFUF
* Gobuster
* Feroxbuster
* Katana
* Nuclei
* Subfinder
* Assetfinder
* Amass
* httpx
* dig
* dnsenum
* whois
* wafw00f
* GAU
* Waybackurls

Tool output is treated as **reconnaissance evidence**, not automatically as a confirmed vulnerability.

---

# 👩‍💻 Author

**Bhumika Mishra**

Cybersecurity / VAPT Portfolio Project

**Repository:** `OWASP-Juice-Shop-VAPT`

---

> **Disclaimer:** This documentation is intended for authorized security testing of OWASP Juice Shop and equivalent intentionally vulnerable laboratory environments. Techniques such as subdomain enumeration, directory discovery, port scanning, crawling and automated scanning should only be performed against systems explicitly authorized for testing.
