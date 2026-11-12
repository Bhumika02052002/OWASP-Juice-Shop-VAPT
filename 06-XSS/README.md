# 06 - Cross-Site Scripting (XSS)

## 1. Introduction

Cross-Site Scripting (XSS) is a web application security vulnerability that allows an attacker to inject malicious client-side code, usually JavaScript, into a webpage viewed by another user.

XSS occurs when an application accepts untrusted input and processes or displays it without appropriate validation, sanitization, or context-aware output encoding.

When a browser interprets injected content as executable code, an attacker may be able to execute JavaScript in the security context of the vulnerable application.

### Potential Impact

- Unauthorized JavaScript execution in a victim's browser.
- Modification of webpage content.
- Display of fake login forms and phishing content.
- Unauthorized actions using the victim's application privileges.
- Exposure of information accessible to browser-side scripts.
- Redirection to malicious websites.
- Compromise of application integrity and user trust.

**Project:** OWASP Juice Shop VAPT  
**Module:** 06 - Cross-Site Scripting (XSS)  
**Target:** OWASP Juice Shop (Authorized Local Lab)  
**Environment:** Kali Linux / VMware Workstation  
**Tools:** Burp Suite, Browser Developer Tools, Kali Linux

> All practical testing described in this document is intended for an authorized OWASP Juice Shop lab environment.

---

# 2. Objectives

The objectives of this XSS assessment are:

1. Understand different types of XSS vulnerabilities.
2. Identify user-controlled input fields.
3. Discover potentially vulnerable parameters and endpoints.
4. Intercept and analyze HTTP requests using Burp Suite.
5. Identify unsafe HTML and JavaScript rendering.
6. Test reflected, stored and DOM-based XSS.
7. Investigate API-based and HTTP-header-based injection.
8. Understand client-side and server-side XSS protection.
9. Analyze Content Security Policy (CSP).
10. Identify vulnerable source-code statements.
11. Understand the Juice Shop Find It and Fix It coding challenges.
12. Recommend secure coding practices.
13. Verify vulnerabilities through controlled proof-of-concept testing.

---

# 3. Types of Cross-Site Scripting

## 3.1 Reflected XSS (Non-Persistent XSS)

### Explanation

Reflected XSS occurs when user-controlled input is received through an HTTP request and immediately included in the server's response without appropriate output encoding.

The payload is generally supplied through URL parameters, search fields or other request parameters.

### Example

```text
https://example.com/search?q=<script>alert(1)</script>
```

If the application inserts the supplied input into an executable HTML context without proper protection, the browser may execute the JavaScript.

### Common Injection Locations

- Search functionality.
- URL query parameters.
- Error messages.
- Tracking pages.
- Filter parameters.
- Application-generated notifications.

### Testing Steps

1. Open the OWASP Juice Shop application.
2. Explore search, tracking and other input functionality.
3. Identify URL parameters.
4. Enter a unique test string, such as:

```text
XSS_TEST_12345
```

5. Submit the request.
6. Observe whether the input appears in the webpage.
7. Open Burp Suite.
8. Capture the corresponding HTTP request.
9. Send the request to Repeater.
10. Inspect the server response.
11. Check whether the input is HTML-encoded.
12. Identify the output context.
13. Test a harmless JavaScript proof of concept in the authorized lab.

### Example Payload

```html
<script>alert(1)</script>
```

Alternative:

```html
<img src=x onerror=alert(1)>
```

### Expected Result

If vulnerable, the supplied input is reflected into an executable browser context and JavaScript executes.

### Vulnerable Code Example

```javascript
app.get("/search", (req, res) => {

    const query = req.query.q;

    res.send(`<h2>Search: ${query}</h2>`);

});
```

**Reason:** User-controlled input is directly concatenated into an HTML response.

### Fixed Code Example

```javascript
app.get("/search", (req, res) => {

    const query = req.query.q;

    res.render("search", {
        query: query
    });

});
```

EJS template:

```ejs
<h2>Search: <%= query %></h2>
```

**Explanation:**

- Uses an auto-escaping template output mechanism.
- Encodes HTML-sensitive characters in the HTML text context.
- Prevents the input from being interpreted as executable HTML in this context.

### Evidence to Capture

- Original URL.
- Modified URL.
- Burp request.
- Burp response.
- Browser execution result.

---

## 3.2 Stored XSS (Persistent XSS)

### Explanation

Stored XSS occurs when an application stores attacker-controlled input in a database or another persistent storage location and later displays it to users without appropriate output encoding.

Unlike reflected XSS, the injected content remains stored and may execute whenever the affected page is viewed.

### Common Injection Locations

- Product reviews.
- Feedback forms.
- Comments.
- User profile information.
- Username fields.
- Support messages.
- Other user-generated content.

### Testing Steps

1. Open OWASP Juice Shop.
2. Log in using an authorized test account.
3. Identify a feature that stores user-generated content.
4. Select a feedback, review or profile-related feature.
5. Enter a unique test value.
6. Submit the information.
7. Refresh the page.
8. Navigate away and return.
9. Check whether the submitted content remains stored.
10. Inspect how the application displays the saved value.
11. Capture the request using Burp Suite.
12. Analyze the response and rendering behavior.
13. Perform a harmless XSS test in the authorized lab.
14. Verify whether the payload executes when the stored content is displayed.

### Example Payload

```html
<script>alert('Stored XSS')</script>
```

Alternative:

```html
<img src=x onerror=alert('Stored XSS')>
```

### Expected Result

If vulnerable, the injected content is stored and executes when the affected page renders it.

### Vulnerable Code Example

Backend:

```javascript
app.post("/feedback", async (req, res) => {

    const comment = req.body.comment;

    await Feedback.create({
        comment: comment
    });

    res.send("Feedback saved");

});
```

Frontend:

```javascript
commentContainer.innerHTML = savedComment;
```

**Reason:**

- User input is stored.
- Stored content is later inserted using an unsafe HTML rendering method.
- The browser may execute injected content.

### Fixed Code Example

Backend:

```javascript
app.post("/feedback", async (req, res) => {

    const comment = req.body.comment;

    await Feedback.create({
        comment: comment
    });

    res.send("Feedback saved");

});
```

Frontend:

```javascript
commentContainer.textContent = savedComment;
```

**Explanation:**

- User input is rendered as plain text.
- HTML is not interpreted by the browser.
- This prevents execution through this particular rendering path.

If the application requires rich HTML formatting, use a trusted HTML sanitizer with a restrictive allowlist and appropriate output handling.

### Evidence to Capture

- Original submission.
- Burp request.
- Stored content.
- Page after refresh.
- Browser execution result.

---

## 3.3 DOM-Based XSS

### Explanation

DOM-Based XSS occurs when client-side JavaScript processes untrusted input and inserts it into the Document Object Model (DOM) through an unsafe operation.

The vulnerability may occur entirely within the browser without the server directly returning the malicious payload in the response.

### Common Sources

```javascript
location.hash
location.search
document.URL
document.referrer
window.name
```

### Common Dangerous Sinks

```javascript
innerHTML
outerHTML
document.write()
eval()
```

These functions are not automatically vulnerable in every use. The vulnerability depends on whether untrusted data reaches an unsafe execution or rendering context.

### Testing Steps

1. Open OWASP Juice Shop.
2. Explore URL-based functionality.
3. Identify URL query parameters and fragments.
4. Enter a unique test string.
5. Open Developer Tools using F12.
6. Navigate to the Elements tab.
7. Search for the test string in the DOM.
8. Open the Sources tab.
9. Inspect JavaScript responsible for processing the input.
10. Identify the source from which the input originates.
11. Identify the function or DOM sink receiving the input.
12. Check whether sanitization or safe encoding is applied.
13. Test a harmless proof of concept in the authorized lab.
14. Verify whether JavaScript executes.

### Example Payload

```html
<img src=x onerror=alert('DOM XSS')>
```

### Vulnerable Code Example

```javascript
const userInput = location.hash.substring(1);

document.getElementById("output").innerHTML = userInput;
```

**Reason:**

- Input is obtained from the URL fragment.
- It is inserted directly into the DOM.
- The browser may interpret injected HTML.

### Fixed Code Example

```javascript
const userInput = location.hash.substring(1);

document.getElementById("output").textContent = userInput;
```

**Explanation:**

- Uses `textContent` instead of `innerHTML`.
- Displays input as text.
- Prevents HTML interpretation through this operation.

### Evidence to Capture

- URL containing the test input.
- Developer Tools Elements tab.
- JavaScript source code.
- Vulnerable DOM sink.
- Browser execution result.

---

## 3.4 Blind XSS

### Explanation

Blind XSS occurs when injected content is stored or processed in an application location that the tester cannot directly observe.

The payload may execute later when an administrator or another user opens the affected page.

### Common Injection Locations

- Contact forms.
- Support tickets.
- Feedback systems.
- Administrative dashboards.
- Application logs.
- Internal reporting systems.

### Testing Steps

1. Identify forms that store information for later review.
2. Inspect the corresponding HTTP requests.
3. Determine where the submitted information is stored.
4. Submit a harmless unique test marker.
5. Investigate whether the information is displayed in another application interface.
6. If authorized, use a controlled callback-monitoring environment.
7. Monitor for the expected callback.
8. Record the execution context and evidence.
9. Remove test data after testing.

### Important

Do not send test payloads to external users or administrators without authorization. Use a controlled lab callback environment.

### Vulnerable Code Example

```javascript
const ticket = req.body.message;

await SupportTicket.create({
    message: ticket
});
```

Unsafe administrative rendering:

```javascript
adminPanel.innerHTML = ticket.message;
```

### Fixed Code Example

```javascript
adminPanel.textContent = ticket.message;
```

**Explanation:**

- Treats stored support messages as untrusted input.
- Renders messages as text.
- Prevents execution through the unsafe HTML rendering operation.

### Evidence to Capture

- Submission request.
- Stored record.
- Administrative rendering context.
- Controlled callback or execution evidence, if applicable.

---

## 3.5 API-Based XSS

### Explanation

API-based XSS occurs when an application accepts user-controlled data through an API, stores or returns that data, and later renders it unsafely in a browser.

An API response containing HTML is not automatically an XSS vulnerability. The vulnerability must be demonstrated in the browser rendering context.

### Testing Steps

1. Open Burp Suite.
2. Browse the Juice Shop application.
3. Identify API requests involving user-generated content.
4. Capture a legitimate request.
5. Send the request to Repeater.
6. Inspect the HTTP method.
7. Identify the endpoint and input properties.
8. Review the request body.
9. Test harmless input values.
10. Inspect the API response.
11. Determine whether the data is stored.
12. Open the relevant page that displays the API data.
13. Inspect how the frontend renders the returned content.
14. Verify whether unsafe HTML or JavaScript executes.

### Example Request Body

```json
{
    "comment": "<img src=x onerror=alert(1)>"
}
```

### Vulnerable Code Example

```javascript
const response = await fetch("/api/feedback");

const data = await response.json();

feedbackContainer.innerHTML = data.comment;
```

### Fixed Code Example

```javascript
const response = await fetch("/api/feedback");

const data = await response.json();

feedbackContainer.textContent = data.comment;
```

**Explanation:**

- The API data is treated as untrusted.
- The frontend displays the content as text.
- Prevents unsafe HTML interpretation.

### Evidence to Capture

- API request.
- Request body.
- API response.
- Frontend rendering.
- Verified browser behavior.

---

## 3.6 HTTP-Header-Based XSS

### Explanation

HTTP-Header-Based XSS occurs when an application processes attacker-controlled HTTP header values and later displays them in an unsafe HTML context.

Headers such as User-Agent and Referer may contain user-controlled values.

### Headers to Investigate

```http
User-Agent
Referer
X-Forwarded-For
```

These headers are examples of potentially user-controlled input and are not automatically vulnerable.

### Testing Steps

1. Open Burp Suite.
2. Capture a request to Juice Shop.
3. Inspect the HTTP request headers.
4. Identify headers that may be logged or displayed.
5. Modify a test header in the authorized lab.
6. Send the request.
7. Investigate whether the application stores the value.
8. Identify where the stored value is displayed.
9. Inspect the rendering context.
10. Verify whether the application safely encodes the value.

### Vulnerable Code Example

```javascript
const userAgent = req.headers["user-agent"];

await Log.create({
    userAgent: userAgent
});
```

Unsafe rendering:

```javascript
logContainer.innerHTML = log.userAgent;
```

### Fixed Code Example

```javascript
logContainer.textContent = log.userAgent;
```

**Explanation:**

- Treats HTTP header values as untrusted.
- Displays values as plain text.
- Prevents HTML interpretation at the output location.

### Evidence to Capture

- Modified HTTP header.
- Captured request.
- Server-side storage or processing.
- Rendering location.
- Verified result.

---

## 3.7 HTML Injection

### Explanation

HTML Injection occurs when an application allows user-controlled HTML to be inserted into a webpage.

HTML Injection does not necessarily execute JavaScript, but it may change webpage content or create fake interface elements.

If the injected HTML allows JavaScript execution, it may become an XSS vulnerability.

### Testing Steps

1. Identify an input field.
2. Enter a unique text marker.
3. Test harmless HTML elements.
4. Submit the input.
5. Inspect the rendered webpage.
6. Check whether the browser interprets the supplied HTML.
7. Investigate whether event handlers or executable contexts are available.
8. Document the actual impact.

### Example Payload

```html
<h2>HTML Injection Test</h2>
```

### Vulnerable Code Example

```javascript
output.innerHTML = userInput;
```

### Fixed Code Example

```javascript
output.textContent = userInput;
```

**Explanation:** User input is displayed as text instead of being interpreted as HTML.

---

## 3.8 Content Security Policy (CSP) Analysis

### Explanation

Content Security Policy is a browser security mechanism that controls which resources and scripts a webpage is allowed to load or execute.

CSP provides an additional layer of defense against certain XSS attacks.

### Testing Steps

1. Open OWASP Juice Shop.
2. Press F12 to open Developer Tools.
3. Select the Network tab.
4. Reload the application.
5. Select the main document request.
6. Inspect the response headers.
7. Find the Content-Security-Policy header.
8. Review the directives.
9. Check the browser Console for CSP violations.
10. Investigate whether the policy permits unsafe script execution.

### Important CSP Directives

| Directive | Purpose |
|---|---|
| default-src | Default resource-loading policy |
| script-src | Controls permitted JavaScript sources |
| style-src | Controls stylesheet sources |
| img-src | Controls image sources |
| object-src | Controls plugin/object resources |
| frame-src | Controls iframe sources |
| base-uri | Restricts document base URLs |
| frame-ancestors | Controls which sites can embed the page |

### Weak CSP Example

```http
Content-Security-Policy: default-src *; script-src 'unsafe-inline' 'unsafe-eval'
```

### Improved CSP Example

```http
Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-RANDOM_NONCE'; object-src 'none'; base-uri 'self'; frame-ancestors 'self'
```

**Explanation:**

- Restricts script sources.
- Uses a nonce-based script policy.
- Prevents unauthorized inline scripts without the matching nonce.
- Blocks plugin/object resources.
- Restricts document base URLs.

The nonce must be unpredictable and generated separately for each response. Do not use a fixed nonce in production.

CSP is defense in depth and does not replace fixing the underlying XSS vulnerability.

---

# 4. OWASP Juice Shop XSS Challenges

The official Juice Shop challenge system includes XSS-related challenges and associated coding exercises.

The main challenge areas include:

| Challenge | Learning objective |
|---|---|
| DOM XSS | Identify unsafe client-side DOM handling |
| Bonus Payload | Understand additional DOM XSS payload contexts |
| Reflected XSS | Understand reflected input |
| API-only XSS | Investigate XSS through API functionality |
| Client-side XSS Protection | Understand client-side security controls |
| Server-side XSS Protection | Understand backend security controls |
| HTTP-Header XSS | Investigate header-based input handling |
| CSP Bypass | Understand CSP restrictions |
| Video XSS | Investigate media-related input handling |

**Note:** Challenge availability and names may depend on the Juice Shop version. Not every category represents a separate built-in coding challenge.

---

# 5. How to Use Jump to Code in OWASP Juice Shop

One of the most useful features of Juice Shop for secure coding practice is its coding challenge system.

It helps identify vulnerable source code and understand the corresponding secure implementation.

## 5.1 Open the Score Board

1. Start OWASP Juice Shop.
2. Open the application in your browser.
3. Navigate to the Score Board.
4. Locate the XSS-related challenges.
5. Complete the relevant challenge in the authorized lab.
6. Look for an associated coding challenge.

## 5.2 Find It - Identify Vulnerable Code

The Find It phase helps identify the actual vulnerable statement in the source code.

Steps:

1. Open the relevant coding challenge.
2. Read the challenge description.
3. Examine the displayed source-code snippet.
4. Understand how user-controlled input is processed.
5. Identify the unsafe source-to-sink data flow.
6. Select the vulnerable statement or statements.
7. Submit your selection.
8. Review the challenge feedback.

### What to Look For

| Code pattern | Possible security issue |
|---|---|
| innerHTML | Unsafe HTML insertion |
| outerHTML | Unsafe DOM modification |
| document.write() | Dynamic HTML insertion |
| eval() | Dynamic code execution |
| location.hash | Potential untrusted DOM source |
| location.search | Potential untrusted DOM source |
| bypassSecurityTrustHtml() | Bypasses Angular HTML sanitization |
| Unescaped template output | Potential HTML injection |

A function is not automatically vulnerable just because it appears in this list. Trace the input and understand the context.

## 5.3 Fix It - Select Secure Code

After identifying the vulnerable code:

1. Open the Fix It phase.
2. Review the proposed code corrections.
3. Understand what makes the original implementation unsafe.
4. Compare the available solutions.
5. Identify the correction that properly protects the affected output context.
6. Select the secure solution.
7. Submit your selection.
8. Read the explanation.
9. Record the vulnerable and corrected code in your notes.

## 5.4 Important Coding Challenge Notes

- Find It identifies vulnerable source-code statements.
- Fix It teaches secure coding alternatives.
- A challenge's solution should be understood, not just memorized.
- Some XSS challenges do not have an associated Find It/Fix It exercise.
- Do not assume a source-code example in this README is the exact code from Juice Shop.
- Always record the actual filename and line number when reviewing your installed source version.

---

# 6. Vulnerable Code vs Fixed Code - Secure Coding Reference

The following examples demonstrate common XSS coding mistakes and their corresponding secure alternatives.

They are illustrative examples for learning and source-code analysis, not verified snippets from the Juice Shop repository.

## 6.1 DOM-Based XSS

### Vulnerable Part

```javascript
const input = location.hash.substring(1);

document.getElementById("result").innerHTML = input;
```

### Why Vulnerable?

- Accepts input from the URL fragment.
- Inserts the value directly into HTML.
- Browser may interpret injected HTML and event handlers.

### Fixed Part

```javascript
const input = location.hash.substring(1);

document.getElementById("result").textContent = input;
```

### Why Fixed?

- Uses textContent instead of innerHTML.
- Displays untrusted input as text.
- Prevents HTML interpretation through this operation.

## 6.2 Reflected XSS

### Vulnerable Part

```javascript
const search = req.query.search;

res.send(`<h2>Results for ${search}</h2>`);
```

### Why Vulnerable?

The application inserts user-controlled input directly into an HTML response.

### Fixed Part

```javascript
const search = req.query.search;

res.render("search", {
    search: search
});
```

Template:

```ejs
<h2>Results for <%= search %></h2>
```

### Why Fixed?

The EJS escaped-output tag encodes HTML-sensitive characters for the HTML text context.

## 6.3 Stored XSS

### Vulnerable Part

```javascript
const comment = req.body.comment;

await Comment.create({
    comment: comment
});
```

Unsafe frontend:

```javascript
commentContainer.innerHTML = savedComment;
```

### Why Vulnerable?

Stored untrusted content is interpreted as HTML when displayed.

### Fixed Part

```javascript
commentContainer.textContent = savedComment;
```

### Why Fixed?

The stored value is rendered as plain text instead of executable HTML.

## 6.4 API-Based XSS

### Vulnerable Part

```javascript
const response = await fetch("/api/comments");

const data = await response.json();

container.innerHTML = data.comment;
```

### Fixed Part

```javascript
const response = await fetch("/api/comments");

const data = await response.json();

container.textContent = data.comment;
```

### Why Fixed?

API responses must also be treated as untrusted data. The frontend safely renders the returned value as text.

## 6.5 Client-Side Validation Bypass

### Vulnerable Part

```javascript
const input = document.getElementById("comment").value;

if (input.includes("<script>")) {
    alert("Invalid input");
    return;
}

output.innerHTML = input;
```

### Why Vulnerable?

- Uses blacklist-based filtering.
- Checks only one specific string.
- Does not account for other HTML execution contexts.
- Uses an unsafe HTML sink.

### Fixed Part

```javascript
const input = document.getElementById("comment").value;

output.textContent = input;
```

### Why Fixed?

- Does not depend on a blacklist.
- Uses a safe text-rendering method.
- Prevents HTML execution through this rendering path.

Server-side validation must also be implemented.

## 6.6 Server-Side XSS

### Vulnerable Part

```javascript
app.get("/profile", (req, res) => {

    const username = req.query.username;

    res.send(`<h2>Welcome ${username}</h2>`);

});
```

### Fixed Part

```javascript
app.get("/profile", (req, res) => {

    const username = req.query.username;

    res.render("profile", {
        username: username
    });

});
```

EJS:

```ejs
<h2>Welcome <%= username %></h2>
```

### Why Fixed?

The template uses HTML escaping instead of direct string concatenation.

## 6.7 HTTP-Header-Based XSS

### Vulnerable Part

```javascript
const userAgent = req.headers["user-agent"];

await Log.create({
    userAgent: userAgent
});
```

Unsafe rendering:

```javascript
logContainer.innerHTML = log.userAgent;
```

### Fixed Part

```javascript
logContainer.textContent = log.userAgent;
```

### Why Fixed?

HTTP header values are treated as untrusted input and rendered as text.

## 6.8 Video / Media-Related XSS

### Vulnerable Part

```javascript
const title = req.query.title;

videoContainer.innerHTML = `<h2>${title}</h2>`;
```

### Fixed Part

```javascript
const title = req.query.title;

const heading = document.createElement("h2");

heading.textContent = title;

videoContainer.appendChild(heading);
```

### Why Fixed?

The application creates an HTML element and inserts the title as text instead of concatenating it into HTML.

## 6.9 CSP Misconfiguration

### Vulnerable Configuration

```http
Content-Security-Policy: default-src *; script-src 'unsafe-inline' 'unsafe-eval'
```

### Improved Configuration

```http
Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-RANDOM_NONCE'; object-src 'none'; base-uri 'self'; frame-ancestors 'self'
```

### Why Improved?

- Restricts script sources.
- Uses a per-response nonce.
- Removes unnecessary unsafe script permissions.
- Adds additional browser security restrictions.

CSP must be tested with the application's actual resource requirements.

---

# 7. Important Source Code Functions and Keywords

During manual source-code review, search for the following terms.

| Keyword | What to investigate |
|---|---|
| innerHTML | User input inserted into HTML |
| outerHTML | Dynamic HTML replacement |
| document.write | Dynamic document content |
| eval | Dynamic JavaScript execution |
| Function | Dynamic function construction |
| location.hash | URL fragment as input |
| location.search | Query string as input |
| document.URL | Current URL as input |
| document.referrer | Referrer information |
| bypassSecurityTrustHtml | Angular security bypass |
| req.query | Query-string parameters |
| req.body | Request body parameters |
| req.headers | HTTP headers |
| res.send | Server response construction |
| res.render | Template rendering |
| textContent | Safe plain-text DOM rendering |

### Source-to-Sink Analysis

A useful approach is to trace:

**Source → Processing → Sink**

Example:

```text
location.hash
      |
      v
JavaScript variable
      |
      v
No safe encoding
      |
      v
innerHTML
      |
      v
Potential DOM XSS
```

A secure implementation should ensure untrusted input is handled appropriately before reaching the output.

---

# 8. Common XSS Parameters and Locations to Investigate

There is no universal XSS parameter name. Identify the actual parameters from application functionality and intercepted requests.

| Parameter / Location | What to Check |
|---|---|
| q | Search input |
| search | Search functionality |
| id | User-controlled identifiers |
| comment | Comments and feedback |
| review | Product reviews |
| username | Profile information |
| message | User messages |
| title | User-controlled titles |
| URL query | Reflected input |
| URL fragment | DOM-based input |
| User-Agent | Header-based input |
| Referer | Header-based input |

These are examples for investigation, not confirmed vulnerable Juice Shop parameters.

---

# 9. Burp Suite Testing Methodology

## Step 1: Configure Burp Suite

1. Open Burp Suite.
2. Start a temporary project.
3. Open Proxy.
4. Configure the browser to use Burp.
5. Enable interception.

## Step 2: Identify Input Points

1. Open Juice Shop.
2. Explore all accessible functionality.
3. Identify user-controlled fields.
4. Enter a unique test string.
5. Submit the request.

## Step 3: Inspect the Request

Check:

- HTTP method.
- Request URL.
- Query parameters.
- Request body.
- Cookies.
- Authorization headers.
- Content-Type.
- User-controlled headers.

## Step 4: Send to Repeater

1. Right-click the captured request.
2. Select Send to Repeater.
3. Open Repeater.
4. Inspect the request.
5. Modify test values.
6. Send the request.
7. Analyze the response.

## Step 5: Verify in the Browser

1. Return to the application.
2. Observe how the input is displayed.
3. Inspect the DOM.
4. Check the browser Console.
5. Verify whether JavaScript executes.
6. Record the result.

**Important:** A successful request in Burp Repeater alone does not prove XSS. Confirm the behavior in the actual browser context.

---

# 10. XSS Payload Reference

Use only harmless payloads in your authorized local lab.

| Testing purpose | Example payload |
|---|---|
| Basic JavaScript test | `<script>alert(1)</script>` |
| HTML injection | `<h1>XSS_TEST</h1>` |
| Image event handler | `<img src=x onerror=alert(1)>` |
| SVG event handler | `<svg onload=alert(1)>` |
| Unique input marker | `XSS_TEST_12345` |
| Plain text rendering | `<script>alert(1)</script>` |

### Payload Testing Guidelines

1. Start with a harmless text marker.
2. Determine the output context.
3. Check whether HTML encoding is applied.
4. Test the appropriate context in the local lab.
5. Verify browser execution.
6. Document the result.
7. Remove stored payloads after testing.

Do not assume a payload will work in every browser or application context.

---

# 11. XSS Mitigation and Remediation

## 11.1 Context-Aware Output Encoding

Encode untrusted data according to the output context.

Examples:

- HTML text context: HTML entity encoding.
- HTML attribute context: Attribute encoding.
- JavaScript context: Avoid inserting untrusted input into executable JavaScript.
- URL context: Validate and encode URL components.
- CSS context: Avoid accepting arbitrary CSS values.

## 11.2 Safe DOM Manipulation

Avoid unsafe DOM operations with untrusted data.

Prefer:

```javascript
element.textContent = userInput;
```

Instead of:

```javascript
element.innerHTML = userInput;
```

## 11.3 Input Validation

- Validate expected input formats.
- Use allowlists where appropriate.
- Restrict unexpected input.
- Validate on the server.
- Do not rely exclusively on frontend validation.

## 11.4 HTML Sanitization

When rich HTML is a business requirement:

- Use a maintained HTML sanitization library.
- Configure a restrictive allowlist.
- Remove dangerous elements and attributes.
- Review sanitizer configuration regularly.
- Avoid custom regex-based HTML sanitization.

## 11.5 Secure Template Rendering

Use template engines that automatically escape output.

Avoid unescaped template output unless the data has been safely sanitized for the exact context.

## 11.6 Content Security Policy

Implement a restrictive CSP.

Prefer nonce-based or hash-based script policies.

Avoid unnecessary use of:

```text
unsafe-inline
unsafe-eval
```

## 11.7 Secure Cookies

Use appropriate cookie attributes:

- HttpOnly
- Secure
- SameSite

These attributes provide additional protection but do not fix the underlying XSS vulnerability.

## 11.8 Regular Security Testing

- Review JavaScript source code.
- Test API responses.
- Inspect user-generated content.
- Perform automated and manual testing.
- Retest after remediation.

---

# 12. Evidence Collection

Store screenshots inside the `evidence/` directory.

Suggested GitHub folder structure:

```text
06-XSS/
│
├── README.md
│
└── evidence/

```

### Evidence Documentation Requirements

For every confirmed finding, record:

1. Vulnerability name.
2. Affected functionality.
3. Affected URL or API endpoint.
4. HTTP method.
5. Vulnerable parameter.
6. Original request.
7. Modified request.
8. Proof-of-concept payload.
9. Browser execution result.
10. Security impact.
11. Vulnerable source-code location, if identified.
12. Recommended mitigation.
13. Retest result.

Redact passwords, authentication cookies, session identifiers and sensitive tokens before uploading screenshots publicly.

---

# 13. Vulnerability Reporting Format

Use the following structure when documenting individual XSS findings.

### Finding: Cross-Site Scripting

**Vulnerability:** Reflected / Stored / DOM-Based XSS

**Affected URL:** Actual affected URL

**Affected Endpoint:** Actual endpoint

**HTTP Method:** GET / POST / PUT / PATCH

**Parameter:** Actual vulnerable parameter

**Severity:** Based on demonstrated impact and application context

**Description:**

Explain how untrusted input is processed and reaches an executable browser context.

**Steps to Reproduce:**

1. Open the affected application page.
2. Submit the test input.
3. Capture the request.
4. Verify the response or DOM behavior.
5. Confirm JavaScript execution.

**Proof of Concept:**

Include the harmless payload used in the authorized lab.

**Impact:**

Explain what an attacker could achieve based on the demonstrated behavior.

**Root Cause:**

Explain the unsafe rendering, missing encoding or improper sanitization.

**Vulnerable Code:**

Include the actual source code, filename and line number where available.

**Recommended Fix:**

Provide corrected code and explain the security improvement.

**Retest Result:**

Document whether the original payload is safely handled after remediation.

---

# 14. XSS Testing Checklist

### Discovery

- [ ] Identify all user input fields.
- [ ] Identify URL parameters.
- [ ] Identify API endpoints.
- [ ] Identify stored user-generated content.
- [ ] Inspect HTTP headers.

### Testing

- [ ] Test reflected XSS.
- [ ] Test stored XSS.
- [ ] Test DOM-based XSS.
- [ ] Investigate blind XSS where applicable.
- [ ] Test API-based input handling.
- [ ] Investigate header-based input handling.
- [ ] Test HTML injection.
- [ ] Inspect CSP configuration.

### Burp Suite

- [ ] Capture legitimate requests.
- [ ] Inspect request parameters.
- [ ] Analyze responses.
- [ ] Modify test inputs.
- [ ] Verify browser behavior.

### Source Code Analysis

- [ ] Locate the vulnerable source.
- [ ] Identify the source-to-sink data flow.
- [ ] Identify the unsafe function.
- [ ] Complete Find It where available.
- [ ] Complete Fix It where available.
- [ ] Record vulnerable and fixed code.

### Remediation

- [ ] Apply context-aware output encoding.
- [ ] Use safe DOM APIs.
- [ ] Validate input server-side.
- [ ] Sanitize HTML where required.
- [ ] Implement CSP.
- [ ] Retest the original proof of concept.
- [ ] Capture remediation evidence.

---

# 15. Conclusion

The OWASP Juice Shop XSS assessment focuses on identifying unsafe handling of user-controlled input across frontend and backend components.

The testing process covers reflected, stored and DOM-based XSS, along with API-related, HTTP-header-related and advanced XSS testing scenarios.

The assessment also includes source-code analysis through the Juice Shop coding challenges.

By completing Find It and Fix It exercises, reviewing vulnerable code, understanding secure alternatives and retesting the application, the objective is to develop practical knowledge of:

- Web application penetration testing.
- HTTP request and response analysis.
- JavaScript and DOM security.
- API security testing.
- Source-code review.
- Secure coding.
- Vulnerability remediation.
- Professional VAPT reporting.

**Project:** OWASP Juice Shop VAPT  
**Module:** 06 - Cross-Site Scripting (XSS)  
**Testing Environment:** Kali Linux + VMware + OWASP Juice Shop  
**Testing Tools:** Burp Suite + Browser Developer Tools

---

# 16. References

1. OWASP Juice Shop:
   https://owasp.org/www-project-juice-shop/

2. OWASP Juice Shop Official Companion Guide:
   https://help.owasp-juice.shop/

3. OWASP Cross-Site Scripting Prevention Cheat Sheet:
   https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html

4. OWASP DOM-Based XSS Prevention Cheat Sheet:
   https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html

5. OWASP Content Security Policy Cheat Sheet:
   https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html

6. OWASP Web Security Testing Guide:
   https://owasp.org/www-project-web-security-testing-guide/

---

**Final Note:** This README documents the XSS assessment methodology and secure coding examples. Mark a vulnerability as confirmed only after successfully verifying the behavior in the authorized lab. The vulnerable and fixed code examples are educational illustrations and must not be represented as exact Juice Shop source-code findings unless independently verified.
