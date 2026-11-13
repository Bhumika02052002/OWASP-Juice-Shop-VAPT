# HTTP Security – OWASP Juice Shop

## 1. Introduction

HTTP security is an important part of web application security. HTTP headers allow browsers and web servers to exchange security-related instructions that help protect applications against common attacks.

Improper HTTP security configuration can expose applications to:

- Clickjacking
- Cross-Site Scripting (XSS) impact
- MIME-type sniffing
- Information disclosure
- Insecure communication
- Cross-origin resource sharing (CORS) weaknesses
- Insecure cookie handling
- Referrer information leakage
- Unnecessary exposure of server technologies

In this assessment, HTTP security configuration is examined in OWASP Juice Shop by inspecting HTTP request and response headers, testing browser security policies, and reviewing server-side configuration.

The objective is to identify missing, weak, or incorrectly configured HTTP security controls and recommend appropriate remediation.

---

## 2. OWASP Classification

| Classification | Details |
|---|---|
| Vulnerability area | HTTP Security Misconfiguration |
| Primary OWASP Top 10 (2025) | A02 – Security Misconfiguration |
| Related OWASP category | A04 – Cryptographic Failures |
| Related OWASP category | A01 – Broken Access Control, where access-control failures are involved |
| Related CWE | CWE-693 – Protection Mechanism Failure |
| Related CWE | CWE-16 – Configuration |
| Related CWE | CWE-319 – Cleartext Transmission of Sensitive Information |
| Related CWE | CWE-614 – Sensitive Cookie in HTTPS Session Without Secure Attribute |
| Related CWE | CWE-1021 – Improper Restriction of Rendered UI Layers or Frames |
| Testing environment | OWASP Juice Shop |
| Main testing tool | Burp Suite |
| Additional tools | Browser Developer Tools, curl, Kali Linux |

**Important:** Missing HTTP security headers are not automatically exploitable vulnerabilities. Assess each header in context and demonstrate the security impact before reporting a finding.

---

## 3. Objectives

1. Inspect HTTP request and response headers.
2. Identify missing security headers.
3. Examine Content Security Policy (CSP).
4. Check clickjacking protection.
5. Examine MIME-sniffing protection.
6. Review HTTPS and transport security.
7. Inspect cookie security attributes.
8. Analyze CORS configuration.
9. Identify unnecessary server information disclosure.
10. Review cache-control behavior for sensitive responses.
11. Inspect HTTP security configuration in source code.
12. Recommend and validate security improvements.

---

## 4. Important HTTP Security Headers

### 4.1 Content-Security-Policy (CSP)

**Description:**

Content Security Policy is a browser security mechanism that controls which sources are allowed to provide scripts, styles, images, fonts, and other resources.

A properly configured CSP can reduce the impact of certain XSS attacks.

**Testing steps:**

1. Open OWASP Juice Shop.
2. Open Burp Suite.
3. Browse the application.
4. Open Proxy → HTTP history.
5. Select the main HTML response.
6. Inspect the response headers.
7. Search for `Content-Security-Policy`.
8. Review the configured directives.

**Example secure header:**

```http
Content-Security-Policy: default-src 'self'; object-src 'none'; base-uri 'self'; frame-ancestors 'none'
```

**Important directives:**

| Directive | Purpose |
|---|---|
| default-src | Default resource loading policy |
| script-src | Controls JavaScript sources |
| style-src | Controls stylesheet sources |
| img-src | Controls image sources |
| connect-src | Controls fetch, XHR, and WebSocket destinations |
| object-src | Controls plugin/object content |
| base-uri | Restricts base URL changes |
| frame-ancestors | Controls which sites can embed the page |
| form-action | Restricts form submission destinations |

**What to look for:**

- Missing CSP
- `script-src 'unsafe-inline'`
- `script-src 'unsafe-eval'`
- Broad wildcard sources
- Untrusted external script sources
- Missing `object-src`
- Missing `base-uri`

These directives are not automatically vulnerabilities in every context. Assess the application's actual resource requirements.

**Mitigation:**

- Define a restrictive CSP.
- Prefer nonces or hashes for inline scripts.
- Avoid unnecessary wildcards.
- Avoid unsafe directives where possible.
- Test the policy before enforcing it.

Example report-only header:

```http
Content-Security-Policy-Report-Only: default-src 'self'; object-src 'none'
```

Report-only mode monitors violations without enforcing the policy.

---

### 4.2 Strict-Transport-Security (HSTS)

**Description:**

HTTP Strict Transport Security tells browsers to use HTTPS when communicating with a website.

It helps protect users against certain downgrade and SSL-stripping attacks.

**Testing steps:**

1. Open the application over HTTPS if HTTPS is configured.
2. Capture the HTTPS response using Burp Suite.
3. Inspect the response headers.
4. Search for `Strict-Transport-Security`.
5. Review the max-age value and optional directives.

**Example:**

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

**What to check:**

- Is HSTS present on HTTPS responses?
- Is `max-age` sufficiently long?
- Are subdomains intended to be included?
- Is preload appropriate for the domain?

**Mitigation:**

Configure HSTS only after confirming that the domain and relevant subdomains support HTTPS.

Do not assume that an HTTP response should contain an effective HSTS policy. Browsers ignore HSTS received over insecure HTTP.

**Important for local testing:** Juice Shop running at `http://localhost:3000` does not by itself demonstrate an HSTS vulnerability. You need an HTTPS deployment to assess HSTS properly.

---

### 4.3 X-Frame-Options

**Description:**

The X-Frame-Options header helps control whether a browser can render a page inside a frame or iframe.

It can help prevent clickjacking.

**Testing steps:**

1. Capture the main HTML response.
2. Inspect the response headers.
3. Search for `X-Frame-Options`.
4. Check whether it uses an appropriate value.
5. Review the CSP `frame-ancestors` directive as well.

**Example:**

```http
X-Frame-Options: DENY
```

Other commonly used value:

```http
X-Frame-Options: SAMEORIGIN
```

**Meaning:**

- `DENY`: Prevents framing.
- `SAMEORIGIN`: Allows framing only by the same origin.

**Mitigation:**

Use CSP `frame-ancestors` as the modern, flexible framing control. X-Frame-Options can also be retained for compatibility with older browsers.

Example:

```http
Content-Security-Policy: frame-ancestors 'none'
```

---

### 4.4 X-Content-Type-Options

**Description:**

This header instructs browsers not to guess the MIME type of a response.

MIME sniffing can sometimes cause browsers to interpret content differently from the server's declared content type.

**Testing steps:**

1. Capture an HTTP response.
2. Inspect the response headers.
3. Search for `X-Content-Type-Options`.
4. Verify whether the value is `nosniff`.

**Example:**

```http
X-Content-Type-Options: nosniff
```

**Mitigation:**

Configure the header consistently for appropriate application responses.

Also ensure that the server returns correct `Content-Type` values.

---

### 4.5 Referrer-Policy

**Description:**

The Referrer-Policy header controls how much referrer information browsers send when users navigate from one page to another.

A weak configuration may expose URL information to external websites.

**Testing steps:**

1. Capture the main HTML response.
2. Inspect the response headers.
3. Search for `Referrer-Policy`.
4. Review the configured policy.

**Example:**

```http
Referrer-Policy: strict-origin-when-cross-origin
```

Another privacy-focused option:

```http
Referrer-Policy: no-referrer
```

**Mitigation:**

Choose a policy that prevents unnecessary URL information disclosure while supporting application requirements.

Avoid placing passwords, tokens, or other secrets in URLs.

---

### 4.6 Permissions-Policy

**Description:**

Permissions-Policy controls whether a browser feature can be used by a page or embedded content.

Examples include:

- Camera
- Microphone
- Geolocation
- Payment
- USB

**Testing steps:**

1. Capture the main application response.
2. Search for `Permissions-Policy`.
3. Review which browser features are allowed.
4. Identify unnecessary permissions.

**Example:**

```http
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

**Mitigation:**

Disable browser features that the application does not require.

Do not block functionality that the application legitimately needs.

---

### 4.7 Cache-Control

**Description:**

Cache-Control determines whether and how HTTP responses can be stored by browsers and intermediate caches.

Sensitive information may be exposed if private responses are cached inappropriately.

**Testing steps:**

1. Identify authenticated pages or API responses containing sensitive information.
2. Capture the response.
3. Inspect the `Cache-Control` header.
4. Check whether private content is stored by shared caches.
5. Review whether the response needs to be cached.

**Example:**

```http
Cache-Control: no-store
```

For sensitive responses, this is often appropriate.

**Mitigation:**

Use cache directives based on the sensitivity of the response.

Do not blindly disable caching for every static asset.

---

### 4.8 CORS – Cross-Origin Resource Sharing

**Description:**

CORS is a browser mechanism that controls whether scripts from one origin can access resources from another origin.

Incorrect CORS configuration may allow unauthorized websites to read sensitive responses.

**Testing steps:**

1. Capture an API request in Burp Suite.
2. Send the request to Repeater.
3. Add an `Origin` header containing an unrelated test origin.
4. Send the request.
5. Inspect the response headers.
6. Check the Access-Control-Allow-Origin value.
7. Check whether credentials are allowed.
8. Verify whether the origin is reflected dynamically.

**Example request:**

```http
GET /rest/products/search?q=test HTTP/1.1
Host: localhost:3000
Origin: https://attacker.example
```

**Potentially dangerous response:**

```http
Access-Control-Allow-Origin: *
```

Or, in some circumstances:

```http
Access-Control-Allow-Origin: https://attacker.example
Access-Control-Allow-Credentials: true
```

**Important:**

A wildcard origin does not automatically mean sensitive data is exposed. Browsers do not allow credentialed CORS responses with `Access-Control-Allow-Origin: *`.

You must test the actual browser behavior, origin validation, authentication requirements, and response sensitivity.

**Mitigation:**

- Allow only trusted origins.
- Avoid reflecting arbitrary Origin values.
- Avoid enabling credentials unnecessarily.
- Restrict allowed methods and headers.
- Validate CORS behavior on sensitive endpoints.

---

### 4.9 Server Information Disclosure

**Description:**

Some HTTP responses expose server and technology information through headers.

Examples:

```http
Server: ExampleServer/1.2
X-Powered-By: Express
```

This information may help an attacker identify the technology stack.

**Testing steps:**

1. Capture HTTP responses.
2. Inspect all response headers.
3. Search for `Server` and `X-Powered-By`.
4. Record any unnecessary version information.

**Mitigation:**

- Remove unnecessary technology-identification headers.
- Avoid exposing detailed software versions.
- Keep server software updated.
- Do not rely on hiding version information as the only security control.

---

### 4.10 Secure Cookie Attributes

**Description:**

Cookies may contain session identifiers or other sensitive information.

Insecure cookie attributes may expose session information or increase the risk of session attacks.

**Important cookie attributes:**

| Attribute | Purpose |
|---|---|
| Secure | Sends the cookie only over secure HTTPS connections |
| HttpOnly | Prevents JavaScript from reading the cookie |
| SameSite | Controls cross-site cookie sending |
| Path | Restricts the URL path for which the cookie is sent |
| Domain | Defines the cookie's domain scope |

**Testing steps:**

1. Log in using a test account.
2. Capture the login response.
3. Inspect `Set-Cookie`.
4. Check for Secure, HttpOnly, and SameSite.
5. Repeat for relevant session cookies.

**Example:**

```http
Set-Cookie: session=example; Secure; HttpOnly; SameSite=Lax
```

**Mitigation:**

- Use Secure for cookies containing sensitive session information.
- Use HttpOnly for session cookies that do not need JavaScript access.
- Choose an appropriate SameSite policy.
- Restrict cookie Domain and Path where possible.

**Important:** Local HTTP testing can affect whether Secure cookies are sent. Do not conclude that a production HTTPS deployment is vulnerable based only on a local HTTP test.

---

## 5. Practical HTTP Security Testing Using Burp Suite

### Step 1: Start OWASP Juice Shop

Start the Docker container:

```bash
docker start juice-shop
```

Check whether it is running:

```bash
docker ps
```

Open:

```text
http://localhost:3000
```

If your container has a different name, identify it using:

```bash
docker ps -a
```

### Step 2: Open Burp Suite

1. Start Burp Suite.
2. Select Temporary Project.
3. Select Burp defaults.
4. Open Proxy.
5. Configure your browser to use Burp.
6. Open the Burp browser if available.

### Step 3: Capture HTTP Responses

1. Visit the Juice Shop homepage.
2. Browse different features.
3. Open Proxy → HTTP history.
4. Select the main HTML response.
5. Inspect the response headers.
6. Repeat for API responses and authenticated requests.

### Step 4: Inspect Security Headers

Look for:

```http
Content-Security-Policy
Strict-Transport-Security
X-Frame-Options
X-Content-Type-Options
Referrer-Policy
Permissions-Policy
Cache-Control
Access-Control-Allow-Origin
Set-Cookie
Server
X-Powered-By
```

Record which headers are present, their values, and which responses contain them.

### Step 5: Modify Requests

Send relevant requests to Repeater.

For CORS testing, add:

```http
Origin: https://attacker.example
```

For ordinary parameter-handling tests, modify one parameter at a time.

For cookie testing, inspect the original `Set-Cookie` response rather than merely changing the Cookie request header.

### Step 6: Analyze Results

For each test, document:

- Endpoint
- Request method
- HTTP status code
- Response headers
- Observed security configuration
- Expected secure configuration
- Actual impact
- Evidence screenshot

---

## 6. Command-Line Testing Using curl

You can inspect HTTP response headers from Kali Linux.

### Check all response headers

```bash
curl -I http://localhost:3000
```

### Display complete HTTP request and response details

```bash
curl -v http://localhost:3000
```

### Check a specific response

```bash
curl -i http://localhost:3000
```

### Test CORS behavior

```bash
curl -i \
-H "Origin: https://attacker.example" \
http://localhost:3000
```

### Save response headers

```bash
curl -s -D response-headers.txt -o /dev/null \
http://localhost:3000
```

View the results:

```bash
cat response-headers.txt
```

**Note:** curl helps inspect server responses but does not enforce browser security policies. Use a browser to validate actual CSP, CORS, cookie, and framing behavior.

---

## 7. Source Code Analysis – Jump to Code, Find It, Fix It

HTTP security headers are often configured in server-side middleware, framework settings, or reverse-proxy configuration.

### Step 1: Locate Juice Shop Source Code

Official repository:

https://github.com/juice-shop/juice-shop

If using Docker:

```bash
docker ps
```

Enter the container:

```bash
docker exec -it juice-shop sh
```

The exact container name may differ.

For source-code analysis, it may be easier to clone the official repository onto Kali:

```bash
git clone https://github.com/juice-shop/juice-shop.git
```

```bash
cd juice-shop
```

### Step 2: Jump to Code

When a challenge offers Jump to Code:

1. Open the relevant Juice Shop challenge.
2. Select Jump to Code.
3. Inspect the referenced source file.
4. Locate the middleware or configuration responsible for the HTTP response.
5. Identify the header or security policy being configured.
6. Understand how the response is generated.

### Step 3: Find It

Search for security-header configuration.

```bash
grep -Rni "Content-Security-Policy" .
```

```bash
grep -Rni "Strict-Transport-Security" .
```

```bash
grep -Rni "X-Frame-Options" .
```

```bash
grep -Rni "X-Content-Type-Options" .
```

```bash
grep -Rni "Referrer-Policy" .
```

Search for common Express security middleware:

```bash
grep -Rni "helmet" .
```

If ripgrep is installed:

```bash
rg -n "helmet|Content-Security-Policy|X-Frame-Options|Strict-Transport-Security" .
```

### Step 4: Fix It

If you are modifying an authorized test application:

1. Identify the missing or weak security configuration.
2. Determine which policy is appropriate.
3. Update the relevant middleware or server configuration.
4. Restart the application.
5. Capture the response again.
6. Compare the old and new headers.
7. Validate application functionality.

**Important:** Do not assume every header is configured in one source file. It may be set by a reverse proxy, web server, application middleware, or deployment platform.

---

## 8. Vulnerable vs. Secure Code Examples

The following are illustrative Express.js examples. They are not claimed to be the exact Juice Shop source code.

### Example 1: Missing Security Headers

**Vulnerable implementation:**

```javascript
const express = require("express");

const app = express();

app.get("/", (req, res) => {
    res.send("Welcome");
});

app.listen(3000);
```

**Problem:**

The application does not explicitly configure common browser security headers.

**Secure implementation:**

Install Helmet:

```bash
npm install helmet
```

Configure it:

```javascript
const express = require("express");
const helmet = require("helmet");

const app = express();

app.use(helmet());

app.get("/", (req, res) => {
    res.send("Welcome");
});

app.listen(3000);
```

**Improvement:**

Helmet configures several security-related HTTP headers. Review its defaults and configure policies according to application requirements.

### Example 2: Weak Content Security Policy

**Vulnerable configuration:**

```http
Content-Security-Policy: default-src * 'unsafe-inline' 'unsafe-eval'
```

**Problem:**

The policy permits overly broad resource loading and weakens script restrictions.

**Improved configuration:**

```javascript
const helmet = require("helmet");

app.use(
    helmet.contentSecurityPolicy({
        directives: {
            defaultSrc: ["'self'"],
            scriptSrc: ["'self'"],
            objectSrc: ["'none'"],
            baseUri: ["'self'"],
            frameAncestors: ["'none'"]
        }
    })
);
```

**Important:** A production CSP may need carefully configured nonces, hashes, and trusted resource sources. Test it thoroughly before enforcement.

### Example 3: Missing Clickjacking Protection

**Vulnerable behavior:**

The application does not restrict framing.

**Secure configuration:**

```javascript
app.use(
    helmet.frameguard({
        action: "deny"
    })
);
```

Equivalent CSP protection:

```http
Content-Security-Policy: frame-ancestors 'none'
```

### Example 4: Insecure Cookie Configuration

**Vulnerable implementation:**

```javascript
res.cookie("session", sessionId);
```

**Secure implementation:**

```javascript
res.cookie("session", sessionId, {
    secure: true,
    httpOnly: true,
    sameSite: "lax"
});
```

The Secure attribute requires HTTPS for normal browser transmission. Configure cookie behavior according to the application's authentication and cross-site requirements.

### Example 5: Insecure CORS Configuration

**Vulnerable implementation:**

```javascript
app.use((req, res, next) => {
    res.header("Access-Control-Allow-Origin", req.headers.origin);
    res.header("Access-Control-Allow-Credentials", "true");
    next();
});
```

**Problem:**

The application reflects arbitrary Origin values while allowing credentials.

**Secure implementation:**

```javascript
const cors = require("cors");

const allowedOrigins = [
    "https://trusted.example"
];

app.use(cors({
    origin: function (origin, callback) {
        if (!origin || allowedOrigins.includes(origin)) {
            return callback(null, true);
        }

        return callback(new Error("Origin not allowed"));
    },
    credentials: true
}));
```

Configure trusted origins explicitly and handle rejected origins safely.

---

## 9. Impact of Weak HTTP Security

| Issue | Potential impact |
|---|---|
| Missing or weak CSP | May increase the impact of XSS |
| Missing framing protection | May enable clickjacking |
| Missing nosniff | May allow unsafe MIME-type interpretation |
| Missing HSTS | May increase exposure to downgrade attacks |
| Weak CORS policy | May permit unauthorized cross-origin data access |
| Insecure cookies | May increase session exposure |
| Weak Referrer-Policy | May expose URL information |
| Server header disclosure | May assist technology reconnaissance |
| Weak cache controls | May expose sensitive cached information |

Severity must be determined based on the actual deployment, affected data, browser behavior, and exploitability.

---

## 10. Mitigation and Prevention Checklist

### Security Headers

- Configure an appropriate Content-Security-Policy.
- Configure X-Content-Type-Options: nosniff.
- Configure appropriate framing restrictions.
- Configure a suitable Referrer-Policy.
- Configure Permissions-Policy based on required browser features.

### HTTPS and Transport Security

- Enforce HTTPS for production applications.
- Configure HSTS after validating HTTPS across the relevant domain.
- Use modern TLS configurations.
- Avoid transmitting sensitive information over unencrypted connections.

### Cookies

- Set Secure on sensitive cookies.
- Set HttpOnly where JavaScript access is unnecessary.
- Configure an appropriate SameSite policy.
- Restrict cookie scope where possible.

### CORS

- Allow only trusted origins.
- Avoid reflecting arbitrary origins.
- Restrict methods and headers.
- Enable credentials only when required.

### Server Configuration

- Remove unnecessary technology-identification headers.
- Keep server software updated.
- Avoid exposing debugging information.
- Configure cache behavior according to response sensitivity.

---

## 11. Evidence and Screenshot Structure

Save your testing evidence in this structure:

```text
10-HTTP-Security/
│
├── README.md
│
├── Evidence/

```

**Evidence guidelines:**

1. Capture the original HTTP response.
2. Highlight missing or weak headers.
3. Capture modified requests used during testing.
4. Capture the relevant server response.
5. Include the endpoint and HTTP status code.
6. Capture the response after remediation, if implemented.
7. Do not mark a missing header as exploitable without assessing its impact.

---

## 12. Vulnerability Report Template

### Finding: Missing or Misconfigured HTTP Security Header

| Field | Details |
|---|---|
| Finding ID | HTTP-001 |
| Vulnerability | Add the specific HTTP security issue |
| Affected Application | OWASP Juice Shop |
| Affected Endpoint | Add tested endpoint |
| OWASP Mapping | A02:2025 – Security Misconfiguration |
| CWE | Select the CWE relevant to the actual finding |
| Severity | Based on demonstrated impact |
| Status | Open / Fixed / Retested |

**Description:**

The application was observed to have a missing or incorrectly configured HTTP security control. The response headers did not provide the expected protection for the tested application behavior.

**Steps to Reproduce:**

1. Open OWASP Juice Shop.
2. Configure Burp Suite.
3. Browse the target endpoint.
4. Capture the HTTP response.
5. Inspect the response headers.
6. Identify the missing or weak security configuration.
7. Document the actual security impact.

**Evidence:**

Add the relevant screenshots from the Evidence folder.

**Impact:**

Explain how the observed configuration could affect the security of the application.

**Recommendation:**

Configure the appropriate HTTP security header and validate it across relevant application responses.

**Retest Result:**

Document whether the issue was successfully corrected.

---

## 13. HTTP Security Testing Checklist

- [ ] Started OWASP Juice Shop.
- [ ] Configured Burp Suite.
- [ ] Captured normal HTTP responses.
- [ ] Inspected Content-Security-Policy.
- [ ] Inspected Strict-Transport-Security.
- [ ] Inspected X-Frame-Options.
- [ ] Inspected X-Content-Type-Options.
- [ ] Inspected Referrer-Policy.
- [ ] Inspected Permissions-Policy.
- [ ] Inspected Cache-Control.
- [ ] Tested CORS behavior.
- [ ] Inspected Set-Cookie attributes.
- [ ] Checked Server and X-Powered-By headers.
- [ ] Used curl to inspect response headers.
- [ ] Reviewed relevant source-code configuration.
- [ ] Documented confirmed security findings.
- [ ] Captured screenshots.
- [ ] Recommended remediation.
- [ ] Retested corrected configurations.

---

## 14. References

1. OWASP Top 10 – 2025  
   https://owasp.org/Top10/2025/

2. OWASP Secure Headers Project  
   https://owasp.org/www-project-secure-headers/

3. OWASP HTTP Headers Cheat Sheet  
   https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Headers_Cheat_Sheet.html

4. OWASP Transport Layer Security Cheat Sheet  
   https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Security_Cheat_Sheet.html

5. OWASP Cross-Origin Resource Sharing Cheat Sheet  
   https://cheatsheetseries.owasp.org/cheatsheets/Cross-Origin_Resource_Sharing_Cheat_Sheet.html

6. MDN HTTP Headers Documentation  
   https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers

7. Helmet.js Documentation  
   https://helmetjs.github.io/

8. PortSwigger Web Security Academy  
   https://portswigger.net/web-security

9. OWASP Juice Shop  
   https://github.com/juice-shop/juice-shop

---

## 15. Conclusion

HTTP security configuration is an essential part of protecting modern web applications.

During this assessment, the focus is on inspecting HTTP response headers, analyzing browser security policies, checking CORS behavior, reviewing cookie attributes, and identifying insecure transport configurations.

Burp Suite helps capture and inspect HTTP traffic, curl provides command-line header analysis, and source-code review helps identify the root cause of insecure configuration.

Correctly configured HTTP security controls can reduce the risk of clickjacking, information disclosure, insecure cross-origin access, and other browser-related security weaknesses.

**Primary OWASP Mapping:** A02:2025 – Security Misconfiguration

**Project:** OWASP Juice Shop – Web Application Security Testing

**Tools:** Burp Suite, Kali Linux, Docker, curl, Browser Developer Tools
