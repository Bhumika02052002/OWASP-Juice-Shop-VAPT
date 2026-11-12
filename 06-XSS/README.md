# 06 - Cross-Site Scripting (XSS)

## 1. Introduction

Cross-Site Scripting (XSS) is a web application vulnerability that allows an attacker to inject malicious client-side code, usually JavaScript, into a web page viewed by another user.

XSS occurs when an application accepts untrusted input and displays it in a browser without proper validation, sanitization, or context-aware output encoding.

An attacker may exploit XSS to:
- Execute unauthorized JavaScript in a victim's browser.
- Modify webpage content.
- Display fake login forms.
- Perform actions using the victim's authenticated session, depending on available protections.
- Redirect users to malicious websites.
- Access sensitive information exposed to browser-side scripts.

**Lab Target:** OWASP Juice Shop  
**Category:** Cross-Site Scripting  
**Tools:** Burp Suite, Browser Developer Tools, Kali Linux

> All testing in this documentation is performed against an authorized local OWASP Juice Shop lab.

---

## 2. Types of XSS Vulnerabilities

### 2.1 Reflected XSS (Non-Persistent XSS)

Reflected XSS occurs when malicious input supplied through a request is immediately included in the server's response without proper encoding.

The payload is generally delivered through a URL parameter or request parameter.

**Example:**

```text
https://example.com/search?q=<script>alert(1)</script>
```

If the application processes the input unsafely and returns executable HTML, the JavaScript may execute in the victim's browser.

**Characteristics:**
- Payload is not permanently stored.
- Usually requires a victim to visit a crafted URL or submit a crafted request.
- Commonly occurs in search results, error messages and tracking pages.

### 2.2 Stored XSS (Persistent XSS)

Stored XSS occurs when an application saves malicious input in its database or another persistent storage location and later displays it to users without proper output encoding.

**Common injection locations:**
- User feedback.
- Product reviews.
- Username or profile fields.
- Comments.
- Other user-generated content.

**Characteristics:**
- Payload remains stored.
- Other users may trigger the payload by viewing the affected page.
- Can affect multiple users.
- Impact may be greater than reflected XSS.

### 2.3 DOM-Based XSS

DOM-Based XSS occurs when client-side JavaScript processes untrusted input and inserts it into the Document Object Model (DOM) using unsafe methods.

Unlike traditional reflected XSS, the vulnerability may occur entirely within browser-side JavaScript.

**Common dangerous JavaScript sinks:**

```javascript
element.innerHTML = userInput;
document.write(userInput);
element.outerHTML = userInput;
```

**Potentially dangerous sources:**

```javascript
location.search
location.hash
document.URL
window.name
```

These sources are not automatically vulnerable. The vulnerability depends on whether untrusted data reaches an unsafe sink without appropriate handling.

**Characteristics:**
- Vulnerability exists in client-side code.
- May not require a malicious server response.
- Browser Developer Tools help identify the vulnerable JavaScript behavior.

### 2.4 Blind XSS

Blind XSS occurs when an injected payload is stored and executed in another application context that the tester cannot directly observe.

For example, an application may store submitted information and later display it in an administrator dashboard.

**Common locations:**
- Support forms.
- Contact forms.
- Feedback systems.
- Administrative dashboards.
- Application logs.

**Characteristics:**
- Execution may occur later.
- The tester may not see the result immediately.
- Requires controlled callback monitoring in an authorized environment.

### 2.5 Context-Based XSS

XSS can also be classified according to where the application inserts untrusted input.

| Context | Description |
|---|---|
| HTML context | Input is inserted into HTML content. |
| HTML attribute context | Input is inserted inside an HTML attribute. |
| JavaScript context | Input is inserted into JavaScript code. |
| URL context | Input is inserted into a URL or navigation attribute. |
| CSS context | Input is inserted into a CSS property or style block. |

The correct testing and remediation approach depends on the output context.

---

# 3. XSS Vulnerability Discovery Methodology

## Step 1: Identify Input Fields

Open the OWASP Juice Shop application.

Explore different pages and identify all possible user-controlled input fields.

Check the following locations:

- Search bar.
- Login and registration forms.
- Username and profile fields.
- Feedback forms.
- Product review sections.
- Contact or support forms.
- Tracking-related functionality.
- Other fields that accept user input.

**Objective:** Identify locations where user-controlled data enters the application.

## Step 2: Perform Input Reflection Testing

Start with harmless test strings.

Example:

```text
XSS_TEST_12345
```

Enter the string into an input field and submit the request.

Observe:

1. Does the input appear on the webpage?
2. Does it appear in the page source?
3. Is it reflected in the URL?
4. Is it stored and displayed after refreshing?
5. Does it appear inside an HTML element or attribute?

If the input appears in the response, inspect how the application handles it.

**Important:** Reflection alone does not confirm XSS.

## Step 3: Capture Requests Using Burp Suite

1. Open Burp Suite.
2. Navigate to Proxy → Intercept.
3. Enable interception.
4. Open Juice Shop through the configured browser.
5. Submit a test value through an identified input field.
6. Capture the HTTP request.
7. Send the request to Repeater.

Inspect the following:

| Component | What to investigate |
|---|---|
| HTTP Method | GET, POST, PUT or PATCH |
| URL | Identify the affected endpoint. |
| Query Parameters | Identify user-controlled URL values. |
| Request Body | Check form or JSON input. |
| Headers | Identify user-controlled HTTP headers. |
| Response Body | Check how the application returns the input. |
| Content-Type | Identify the response format. |
| Encoding | Check whether special characters are encoded. |

## Step 4: Test Basic HTML Injection

Before testing JavaScript execution, determine whether the application interprets input as HTML.

Use a harmless HTML test:

```html
<h2>XSS_TEST_12345</h2>
```

Submit the input.

Observe:

- Does the browser display the text as a heading?
- Does the application display the HTML as literal text?
- Does the HTML appear in the DOM?
- Is the input encoded or sanitized?

If the application interprets the HTML, investigate the output context further.

## Step 5: Test JavaScript Execution

For an authorized local lab, use a harmless proof-of-concept payload:

```html
<script>alert(1)</script>
```

If script elements are filtered, test another harmless browser-execution context:

```html
<iframe src="javascript:alert('XSS')"></iframe>
```

Observe whether:

- The payload is rejected.
- The payload is displayed as text.
- The payload is modified or encoded.
- JavaScript executes.
- Browser security policies block execution.

A payload being accepted does not automatically prove XSS. Actual execution in the intended browser context must be verified.

---

# 4. Testing Different XSS Types in Juice Shop

## 4.1 DOM-Based XSS Testing

**Objective:** Identify unsafe handling of user-controlled input in browser-side JavaScript.

### Steps

1. Open the Juice Shop homepage.
2. Locate the search functionality.
3. Enter a harmless unique test string.
4. Observe the URL and browser behavior.
5. Open Developer Tools using F12.
6. Navigate to the Elements panel.
7. Search for your test string in the DOM.
8. Check whether the input is inserted into HTML.
9. Inspect the relevant JavaScript using the Sources panel.
10. Identify whether an unsafe DOM sink processes the input.
11. In the authorized lab, test a harmless XSS proof of concept.

**Example payload:**

```html
<iframe src="javascript:alert('XSS')"></iframe>
```

**Expected result:** If the application is vulnerable, the injected code executes in the browser.

**Evidence to capture:**
- Search input.
- Submitted URL or request.
- Browser DOM showing the injected content.
- JavaScript execution result.

## 4.2 Reflected XSS Testing

**Objective:** Determine whether user-controlled request data is returned unsafely in a webpage.

### Steps

1. Explore the Juice Shop application.
2. Complete the checkout process using a test account if required to access tracking functionality.
3. Locate the tracking-related page or URL.
4. Identify URL parameters.
5. Enter a unique test value in the relevant parameter.
6. Observe whether the value appears in the webpage.
7. Capture the HTTP request in Burp Suite.
8. Inspect the response body.
9. Check whether the input is HTML-encoded.
10. Perform a harmless JavaScript execution test in the authorized lab.

**Example payload:**

```html
<iframe src="javascript:alert('XSS')"></iframe>
```

**Expected result:** The injected payload is reflected and executes when the affected page is loaded.

**Evidence to capture:**
- Original URL.
- Modified URL.
- Burp request and response.
- Browser execution result.

## 4.3 Stored XSS Testing

**Objective:** Determine whether malicious input is saved and executed when another user views the affected content.

### Steps

1. Log in to Juice Shop using a test account.
2. Identify a feature that stores user-generated content.
3. Explore feedback, reviews or profile-related functionality.
4. Enter a unique harmless test value.
5. Submit the information.
6. Refresh the page.
7. Navigate away and return.
8. Check whether the submitted value remains stored.
9. Inspect how the stored value is displayed.
10. In the authorized lab, test a harmless XSS payload.
11. Verify whether execution occurs when the stored content is viewed.

**Example payload:**

```html
<iframe src="javascript:alert('XSS')"></iframe>
```

**Expected result:** If vulnerable, the payload is stored and executes when the affected content is rendered.

**Evidence to capture:**
- Original submission.
- Burp request.
- Successful storage.
- Page after refresh.
- Browser execution result.

**Important:** Test with a separate controlled test account where possible. Do not expose other users to stored test payloads.

## 4.4 API-Only XSS Testing

**Objective:** Identify persistent XSS vulnerabilities through direct API interaction without using the normal frontend form.

### Steps

1. Open Burp Suite.
2. Browse the application and identify the relevant API endpoint.
3. Capture a legitimate request that stores user-controlled data.
4. Send the request to Repeater.
5. Inspect the request method, headers and body.
6. Identify the input property that accepts user-controlled data.
7. Review client-side validation and server-side processing.
8. In the authorized lab, test a harmless XSS payload through the API.
9. Check the server response.
10. Open the relevant page that renders the stored information.
11. Verify whether the browser executes the payload.

**Example payload:**

```html
<iframe src="javascript:alert('XSS')"></iframe>
```

**Expected result:** The payload is persisted through the API and executes when rendered by the frontend.

## 4.5 Client-Side XSS Protection Testing

**Objective:** Understand whether browser-side validation can be bypassed and whether the server independently validates and sanitizes input.

### Steps

1. Identify a form that validates user input.
2. Enter a harmless test value.
3. Observe the client-side validation.
4. Capture the legitimate request using Burp Suite.
5. Send the request to Repeater.
6. Compare frontend validation with server-side behavior.
7. Test the request with controlled modified input.
8. Observe whether the server accepts or rejects the input.
9. Verify whether any accepted content is safely encoded when displayed.

**Expected result:** A secure application must not rely exclusively on client-side validation.

## 4.6 Server-Side XSS Protection Testing

**Objective:** Evaluate how the backend validates and sanitizes user-controlled input.

### Steps

1. Identify a feature that stores user-generated content.
2. Capture the corresponding request.
3. Send it to Burp Repeater.
4. Inspect the request body and parameters.
5. Test harmless HTML and special characters.
6. Observe whether the backend rejects, sanitizes or accepts the input.
7. Retrieve the saved content.
8. Check how the frontend renders the returned data.
9. Verify whether executable content can reach the browser.

**Expected result:** Server-side input validation and context-aware output encoding should prevent executable content from being rendered.

## 4.7 HTTP-Header XSS Testing

**Objective:** Identify whether user-controlled HTTP header values are stored or rendered unsafely.

### Steps

1. Open Burp Suite.
2. Capture requests sent to Juice Shop.
3. Identify headers that may contain user-controlled values.
4. Examine headers such as User-Agent and Referer.
5. Inspect whether the application records these values.
6. Determine whether they are displayed in application logs or administrative pages.
7. In the authorized lab, test a harmless HTML or JavaScript payload in the relevant header.
8. Check the page where the header value is rendered.
9. Verify whether the browser executes the content.

**Expected result:** User-controlled header values should be safely encoded before being displayed.

## 4.8 Content Security Policy (CSP) Testing

**Objective:** Understand how Content Security Policy helps restrict script execution and investigate whether the policy is configured correctly.

### Steps

1. Open Juice Shop.
2. Open Developer Tools.
3. Navigate to the Network tab.
4. Reload the application.
5. Select the main document request.
6. Inspect response headers.
7. Search for:

```http
Content-Security-Policy
```

8. Review important directives:

| Directive | Purpose |
|---|---|
| default-src | Default resource loading policy. |
| script-src | Controls permitted JavaScript sources. |
| object-src | Controls plugin/object resource sources. |
| frame-src | Controls permitted iframe sources. |
| base-uri | Restricts document base URLs. |
| frame-ancestors | Controls which sites may embed the page. |

9. Check whether unsafe directives such as `unsafe-inline` or `unsafe-eval` are present.
10. Review browser console messages for CSP violations.
11. Evaluate whether the policy meaningfully restricts script execution.

**Important:** CSP is an additional security layer, not a replacement for secure input handling and output encoding.

## 4.9 Video XSS Testing

**Objective:** Investigate unsafe handling of user-controlled content in video or media-related functionality.

### Steps

1. Identify the relevant video or media functionality in the lab.
2. Inspect the related HTML and JavaScript.
3. Identify how video content and associated metadata are processed.
4. Capture relevant HTTP requests.
5. Inspect parameters and content handling.
6. Review whether untrusted input can enter HTML or script contexts.
7. Use harmless test values to identify unsafe processing.
8. Verify browser behavior and collect evidence.

**Expected result:** Media-related content should be handled safely without allowing user input to execute JavaScript.

---

# 5. Common XSS Payloads for Authorized Testing

The following payloads are for controlled local testing.

| Purpose | Test payload |
|---|---|
| Basic script execution | `<script>alert(1)</script>` |
| HTML injection | `<h1>XSS_TEST</h1>` |
| Image event handler | `<img src=x onerror=alert(1)>` |
| SVG event handler | `<svg onload=alert(1)>` |
| Iframe JavaScript | `<iframe src="javascript:alert('XSS')"></iframe>` |
| HTML comment test | `<!-- XSS_TEST -->` |

**Testing notes:**

- Start with a harmless text marker.
- Identify the output context before selecting a payload.
- Check whether HTML encoding occurs.
- Do not assume one payload works in every context.
- Avoid destructive JavaScript or real credential collection.
- Clean up stored test content after testing.

---

# 6. Parameters and Locations to Check for XSS

There is no universal XSS parameter name. Identify actual input points by exploring the application and intercepting requests.

| Location | What to investigate |
|---|---|
| `q` | Search query input. |
| `search` | Search functionality. |
| `id` | User-controlled identifiers that may reach unsafe rendering. |
| `comment` | User comments or feedback. |
| `review` | Product review content. |
| `username` | Profile-related input. |
| `message` | User-submitted messages. |
| `title` | User-controlled titles. |
| URL query string | Reflected and DOM-based injection. |
| URL fragment | Client-side DOM processing. |
| User-Agent | Potential HTTP-header injection. |
| Referer | Potential HTTP-header injection. |

These are examples, not confirmed vulnerable Juice Shop parameters.

---

# 7. Tools Used

| Tool | Purpose |
|---|---|
| OWASP Juice Shop | Vulnerable web application used as the testing target. |
| Burp Suite | HTTP interception, request analysis and modification. |
| Browser Developer Tools | Inspect DOM, JavaScript, network requests and console output. |
| Kali Linux | Security testing environment. |
| Firefox / Chrome | Browser-based vulnerability verification. |
| Nuclei | Optional template-based security testing where applicable. |

---

# 8. Impact of XSS

Depending on the application and execution context, XSS may result in:

1. Unauthorized JavaScript execution.
2. Modification of displayed webpage content.
3. Phishing through fake forms.
4. Unauthorized actions within the victim's application session.
5. Exposure of information accessible to browser-side JavaScript.
6. Redirection to malicious websites.
7. Compromise of application trust and user confidentiality.

The actual impact depends on the vulnerable page, user privileges, browser security controls and available application functionality.

---

# 9. XSS Mitigation and Remediation

### 9.1 Context-Aware Output Encoding

Encode untrusted data before displaying it.

For example:
- HTML context: HTML entity encoding.
- HTML attributes: Attribute encoding.
- JavaScript context: Avoid inserting untrusted input into executable JavaScript.
- URL context: Validate and encode URL components.

### 9.2 Input Validation

- Validate data according to expected formats.
- Restrict unexpected input.
- Use allowlists where practical.
- Never rely only on client-side validation.

### 9.3 HTML Sanitization

If an application intentionally allows HTML input, use a trusted HTML sanitization library with an appropriate allowlist.

### 9.4 Secure DOM Manipulation

Avoid unsafe DOM methods when processing untrusted data.

Prefer:

```javascript
element.textContent = userInput;
```

Instead of:

```javascript
element.innerHTML = userInput;
```

### 9.5 Content Security Policy

Implement a restrictive CSP.

Prefer nonce-based or hash-based script policies and avoid unnecessary use of `unsafe-inline` and `unsafe-eval`.

### 9.6 Secure Cookie Configuration

Use appropriate cookie attributes:

- HttpOnly
- Secure
- SameSite

These attributes provide additional protection but do not fix the underlying XSS vulnerability.

### 9.7 Security Testing

- Perform regular XSS assessments.
- Test all input fields and output contexts.
- Review frontend JavaScript.
- Validate server-side input handling.
- Retest after remediation.

---

# 10. XSS Testing Checklist

- [ ] Identify all user-controlled input fields.
- [ ] Test input reflection.
- [ ] Identify HTML injection possibilities.
- [ ] Inspect HTTP requests using Burp Suite.
- [ ] Check query parameters.
- [ ] Check POST request bodies.
- [ ] Check API endpoints.
- [ ] Test reflected XSS.
- [ ] Test stored XSS.
- [ ] Test DOM-based XSS.
- [ ] Investigate blind XSS where applicable.
- [ ] Inspect HTTP headers.
- [ ] Review client-side validation.
- [ ] Review server-side sanitization.
- [ ] Inspect CSP response headers.
- [ ] Verify JavaScript execution in the browser.
- [ ] Capture screenshots.
- [ ] Clean up test payloads.
- [ ] Document findings and remediation.

---

# 11. Evidence Collection

Save screenshots in the `evidence/` folder.

Suggested folder structure:

```text
06-XSS/
│
├── README.md
│
└── evidence/
    ├── 01-xss-input-fields.png
    ├── 02-html-injection.png
    ├── 03-reflected-xss-request.png
    ├── 04-reflected-xss-execution.png
    ├── 05-dom-xss-input.png
    ├── 06-dom-xss-execution.png
    ├── 07-stored-xss-submission.png
    ├── 08-stored-xss-execution.png
    ├── 09-api-xss-request.png
    ├── 10-http-header-testing.png
    ├── 11-csp-response-headers.png
    └── 12-xss-remediation.png
```

### Evidence Documentation

For each confirmed vulnerability, record:

- Vulnerability name.
- Affected functionality.
- Affected URL or API endpoint.
- HTTP method.
- Vulnerable input parameter.
- Request and response.
- Proof-of-concept payload.
- Screenshot of the execution result.
- Security impact.
- Recommended mitigation.

Do not mark a vulnerability as confirmed based only on an input being accepted or reflected.

---

# 12. Conclusion

During the OWASP Juice Shop XSS assessment, different input locations, request parameters, client-side processing mechanisms and server-side protections are examined.

The assessment covers reflected, stored and DOM-based XSS along with advanced testing scenarios involving APIs, HTTP headers and Content Security Policy.

The purpose is to understand how untrusted data can reach an executable browser context, how to demonstrate the vulnerability safely, and how to recommend appropriate remediation.

**Project:** OWASP Juice Shop VAPT  
**Module:** 06 - Cross-Site Scripting (XSS)  
**Environment:** Kali Linux + VMware + OWASP Juice Shop  
**Testing Tool:** Burp Suite

---

# 13. References

1. OWASP Cross-Site Scripting Prevention Cheat Sheet:
   https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html

2. OWASP DOM-Based XSS Prevention Cheat Sheet:
   https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html

3. OWASP Juice Shop:
   https://owasp.org/www-project-juice-shop/

4. OWASP Juice Shop XSS Challenges:
   https://help.owasp-juice.shop/part2/xss.html

5. OWASP Web Security Testing Guide:
   https://owasp.org/www-project-web-security-testing-guide/

---

**Note:** This README documents the testing methodology. Individual XSS vulnerabilities should only be marked as confirmed after successful verification in the authorized lab.
