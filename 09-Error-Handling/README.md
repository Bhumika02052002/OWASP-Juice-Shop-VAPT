# Error Handling Vulnerabilities – OWASP Juice Shop

## 1. Introduction

Error handling is an important part of application security. Applications must handle unexpected inputs, failed operations, invalid requests, and internal exceptions safely.

Improper error handling occurs when an application reveals sensitive information, exposes internal implementation details, or behaves insecurely when an unexpected condition occurs.

In OWASP Juice Shop, error-handling behavior can be studied by sending invalid requests, modifying parameters, triggering unexpected application conditions, and examining server responses.

A poorly configured application may expose:

- Internal file paths
- Database error messages
- SQL query details
- Application stack traces
- Framework and server information
- Debug information
- Internal API endpoints
- Software versions
- Sensitive implementation details

The objective of this testing is to identify whether error responses disclose information that could help an attacker understand or exploit the application.

---

## 2. OWASP Classification

| Classification | Details |
|---|---|
| Vulnerability | Improper Error Handling |
| Primary OWASP Top 10 (2025) | A10 – Mishandling of Exceptional Conditions |
| Related OWASP category | A05 – Security Misconfiguration, where misconfiguration causes exposure |
| Related category | A07 – Authentication Failures, where errors enable account enumeration |
| Related CWE | CWE-209 – Generation of Error Message Containing Sensitive Information |
| Related CWE | CWE-755 – Improper Handling of Exceptional Conditions |
| Related CWE | CWE-703 – Improper Check or Handling of Exceptional Conditions |
| Testing environment | OWASP Juice Shop |
| Main testing tool | Burp Suite |

**Important:** Error handling is not always a vulnerability. An error response becomes a security finding when it exposes sensitive information, enables an attack, or demonstrates an insecure exceptional condition.

---

## 3. Objectives

The objectives of this assessment are:

1. Identify verbose error messages.
2. Detect stack trace disclosure.
3. Identify database error information leakage.
4. Check whether internal file paths are exposed.
5. Examine invalid input handling.
6. Test how the application handles unexpected HTTP requests.
7. Identify account enumeration through inconsistent error messages.
8. Inspect server responses using Burp Suite.
9. Locate relevant error-handling logic in the source code.
10. Recommend secure error-handling practices.

---

## 4. Types of Error Handling Vulnerabilities

### 4.1 Verbose Error Messages

**Description:**

Verbose error messages reveal more information than necessary when an application encounters an error.

For example, instead of displaying a generic error, an application may reveal:

- Database connection information
- Internal API routes
- Framework details
- File paths
- Application configuration details

**Testing steps:**

1. Open OWASP Juice Shop.
2. Browse different application features.
3. Identify input fields and API endpoints.
4. Submit invalid or unexpected values.
5. Open Burp Suite and enable Proxy interception or use HTTP history.
6. Inspect the HTTP response.
7. Check whether the response reveals internal implementation details.

**Example request:**

```http
GET /rest/products/search?q=%25invalid-input HTTP/1.1
Host: localhost:3000
```

The exact endpoint and response depend on the Juice Shop version.

**What to inspect:**

- HTTP status code
- Response body
- Error description
- Internal route names
- Server information
- Unexpected debugging information

**Expected secure behavior:**

The application should return a generic error message without revealing internal implementation details.

---

### 4.2 Stack Trace Disclosure

**Description:**

A stack trace provides technical details about the sequence of functions or methods executed before an error occurred.

If exposed to an unauthenticated user, it may reveal:

- Source-code file names
- Internal directory structures
- Function names
- Framework components
- Application architecture
- Internal execution flow

**Testing steps:**

1. Identify API endpoints that accept user input.
2. Send an invalid request through Burp Suite Repeater.
3. Change a parameter to an unexpected data type.
4. Remove a required parameter.
5. Send malformed JSON where applicable.
6. Inspect the response body.
7. Look for stack trace information.

**Example request:**

```http
POST /rest/user/login HTTP/1.1
Host: localhost:3000
Content-Type: application/json

{
  "email": {},
  "password": null
}
```

**Possible indicators:**

```text
TypeError
ReferenceError
at FunctionName (...)
node_modules/
internal server error
```

These are examples of possible indicators, not guaranteed Juice Shop responses.

**Security impact:**

An attacker may use exposed source paths, function names, and framework details to identify potential attack surfaces.

**Expected secure behavior:**

Return a generic error response to the client and store detailed diagnostic information in protected server-side logs.

---

### 4.3 Database Error Disclosure

**Description:**

Database error disclosure occurs when an application reveals database-related error details to the user.

Possible information includes:

- SQL syntax errors
- Table names
- Column names
- Database engine information
- Query structure
- Database configuration details

**Testing steps:**

1. Identify search, login, registration, or other database-backed features.
2. Capture a normal request using Burp Suite.
3. Send the request to Repeater.
4. Change input values to unexpected values.
5. Test special characters and malformed input.
6. Observe whether the response reveals database errors.

**Example test values:**

```text
'
"
''
"
%27
test'
```

Use these values only in the authorized local Juice Shop environment.

**Example of a potentially sensitive error:**

```text
SQLITE_ERROR: no such column: example
```

This is an illustrative example, not a claim that Juice Shop will return this exact response.

**Security impact:**

Database errors may disclose database structure and help attackers refine SQL injection attempts.

**Expected secure behavior:**

- Use parameterized queries.
- Handle database exceptions safely.
- Avoid returning raw database error messages.
- Log detailed errors securely on the server.

---

### 4.4 Internal File Path Disclosure

**Description:**

Internal file path disclosure occurs when an application reveals server-side directory structures or file locations.

Example:

```text
/home/application/server/routes/user.js
```

**Testing steps:**

1. Browse application functionality.
2. Trigger invalid requests.
3. Inspect error responses in Burp Suite.
4. Search for directory structures and source file names.
5. Check whether absolute paths are disclosed.

**What to look for:**

```text
/home/
/var/www/
/app/
/server/
/node_modules/
```

**Impact:**

File path information can help attackers understand application structure and identify interesting components for further investigation.

**Mitigation:**

- Disable detailed error output in production.
- Use centralized exception handling.
- Return generic client-facing messages.
- Protect internal logs.

---

### 4.5 Debug Information Disclosure

**Description:**

Debug information is useful during development but should not be exposed to normal users in production.

Information may include:

- Debug mode status
- Environment variables
- Configuration details
- Internal application state
- Framework diagnostics
- Detailed exception output

**Testing steps:**

1. Send invalid requests to the application.
2. Inspect the response headers and body.
3. Check whether debug information is returned.
4. Inspect error pages.
5. Check whether environment or configuration information is exposed.

**Possible indicators:**

```text
DEBUG = true
development mode
environment configuration
debugger information
```

These are example indicators, not confirmed Juice Shop output.

**Mitigation:**

- Disable production debugging.
- Avoid exposing environment variables.
- Use secure configuration management.
- Restrict diagnostic endpoints.
- Keep detailed diagnostics in protected logs.

---

### 4.6 Inconsistent Error Responses and Account Enumeration

**Description:**

Account enumeration occurs when an application reveals whether a user account exists through different error messages or response behavior.

For example:

```text
Account exists: Invalid password
Account does not exist: User not found
```

An attacker could use this difference to identify registered accounts.

**Testing steps:**

1. Open the login or password recovery functionality.
2. Capture a normal request in Burp Suite.
3. Submit an existing test account with an incorrect password.
4. Submit a clearly nonexistent test account.
5. Compare the response status, message, structure, and timing.
6. Determine whether the responses reveal account existence.

**What to compare:**

- HTTP status codes
- Response messages
- Response structure
- Response length
- Noticeable timing differences

**Mitigation:**

Use consistent authentication error messages and avoid revealing whether an account exists.

---

### 4.7 Improper Handling of Invalid Input

**Description:**

Applications must validate and safely handle unexpected input.

Improper input handling can cause:

- Unhandled exceptions
- Application errors
- Unexpected application behavior
- Information disclosure
- Denial of service in some circumstances

**Testing steps:**

Test the application using:

| Input type | Example |
|---|---|
| Empty value | `""` |
| Null value | `null` |
| Number instead of string | `12345` |
| Object instead of string | `{}` |
| Array instead of string | `[]` |
| Very long string | 5,000-character test string |
| Special characters | `!@#$%^&*()` |
| Unicode input | `测试安全` |
| Malformed JSON | `{"email":` |

1. Capture the normal request.
2. Send it to Repeater.
3. Modify one parameter at a time.
4. Send the request.
5. Inspect the response.
6. Record any exception, information disclosure, or unexpected behavior.

**Mitigation:**

- Validate data types.
- Apply input length restrictions.
- Handle null and missing values.
- Use safe parsing.
- Implement centralized exception handling.

---

## 5. Practical Testing Using Burp Suite

### Step 1: Start OWASP Juice Shop

If you installed Juice Shop using Docker, start the container:

```bash
docker start juice-shop
```

Check the container:

```bash
docker ps
```

Open:

```text
http://localhost:3000
```

If your container has a different name, use the name shown by:

```bash
docker ps -a
```

### Step 2: Open Burp Suite

1. Start Burp Suite.
2. Select Temporary Project.
3. Select Burp defaults.
4. Open Proxy.
5. Open Proxy settings.
6. Confirm the browser proxy configuration.
7. Use Burp's built-in browser or configure your browser to use Burp.

### Step 3: Capture Requests

1. Open Juice Shop through the configured browser.
2. Navigate to login, registration, search, and other application features.
3. Perform normal actions.
4. Open Proxy → HTTP history.
5. Identify API requests.
6. Select an interesting request.
7. Send it to Repeater.

### Step 4: Modify Requests

Test one change at a time:

- Remove a required parameter.
- Replace a string with a number.
- Replace a string with an object.
- Send null values.
- Send malformed JSON.
- Use unexpected special characters.
- Test invalid content types where appropriate.

Example:

```http
POST /rest/user/login HTTP/1.1
Host: localhost:3000
Content-Type: application/json

{
  "email": 12345,
  "password": null
}
```

### Step 5: Analyze Responses

Inspect:

- HTTP status code
- Response headers
- Response body
- Error messages
- Stack traces
- File paths
- Database information
- Framework information

### Step 6: Record Evidence

Save:

- Original request
- Modified request
- Server response
- Screenshot of exposed information
- Explanation of the security impact

Do not report a vulnerability solely because the application returned HTTP 400 or HTTP 500. Demonstrate the actual security impact.

---

## 6. Parameter Discovery – What Should You Test?

Error-handling testing is not limited to one parameter. Identify input points throughout the application.

| Feature | Parameters to inspect |
|---|---|
| Login | email, password |
| Registration | email, password, passwordRepeat |
| Search | q |
| Contact form | comment, rating |
| Feedback | comment, rating |
| Address | country, city, street, postalCode |
| Basket | quantity, product ID |
| Profile | username, email |
| API requests | JSON fields, query parameters, path parameters |
| File upload | filename, content type, file metadata |

The actual parameter names depend on the endpoint and Juice Shop version.

**Parameter discovery methodology:**

1. Browse the application.
2. Capture requests in Burp Suite HTTP history.
3. Identify query parameters, URL path parameters, JSON fields, and form fields.
4. Send interesting requests to Repeater.
5. Change one parameter at a time.
6. Compare normal and abnormal responses.
7. Record the results.

You can also inspect the application's API documentation or source code to discover additional input points.

---

## 7. Source Code Analysis – Jump to Code, Find It, Fix It

Source-code analysis helps determine why an application returns an unsafe error response.

### Step 1: Locate Juice Shop Source Code

If you are using a Docker container, identify its name:

```bash
docker ps
```

You can inspect the application files inside the container:

```bash
docker exec -it juice-shop sh
```

The exact container name may differ.

For easier source-code searching, you can also inspect the official Juice Shop repository:

https://github.com/juice-shop/juice-shop

### Step 2: Jump to Code

When a Juice Shop challenge provides a **Jump to Code** option:

1. Open the relevant challenge.
2. Select Jump to Code.
3. Inspect the related source file.
4. Identify the function responsible for processing the request.
5. Look for error-handling logic.
6. Trace how exceptions are passed to the response.

### Step 3: Find It

Search the source code for relevant patterns.

Useful search commands:

```bash
grep -Rni "catch" .
```

```bash
grep -Rni "next(error)" .
```

```bash
grep -Rni "stack" .
```

```bash
grep -Rni "console.error" .
```

```bash
grep -Rni "res.status" .
```

If `ripgrep` is installed:

```bash
rg -n "catch|next\(error\)|stack|console\.error|res\.status" .
```

**What to look for:**

- Exception handlers
- Error middleware
- Direct error responses
- Stack trace output
- Database exception handling
- Debugging statements
- Error messages returned to clients

### Step 4: Fix It

When the challenge provides a Fix It option:

1. Read the vulnerable implementation.
2. Identify the information being exposed.
3. Determine the expected secure behavior.
4. Apply the correction.
5. Retest the same request.
6. Confirm that sensitive information is no longer exposed.

**Note:** The example code below demonstrates secure coding patterns. It is not claimed to be the exact vulnerable source code in every Juice Shop version. Use the actual challenge source when documenting a real challenge.

---

## 8. Vulnerable vs. Secure Code Examples

### Example 1: Exposing Stack Traces

**Vulnerable implementation:**

```javascript
app.use((err, req, res, next) => {
    res.status(500).json({
        message: err.message,
        stack: err.stack
    });
});
```

**Problem:**

The application returns the complete exception details and stack trace to the client.

**Secure implementation:**

```javascript
app.use((err, req, res, next) => {
    console.error(err);

    res.status(500).json({
        error: "Internal Server Error",
        message: "Something went wrong. Please try again later."
    });
});
```

**Improvement:**

- Detailed information stays in server-side logs.
- The client receives a generic error.
- Internal source paths and stack traces are not exposed.

### Example 2: Exposing Database Errors

**Vulnerable implementation:**

```javascript
database.query(sql, (err, result) => {
    if (err) {
        return res.status(500).send(err.message);
    }

    res.json(result);
});
```

**Problem:**

The raw database error may reveal internal database information.

**Secure implementation:**

```javascript
database.query(sql, (err, result) => {
    if (err) {
        console.error("Database operation failed:", err);

        return res.status(500).json({
            error: "Internal Server Error",
            message: "Unable to process your request."
        });
    }

    res.json(result);
});
```

**Additional recommendation:**

Use parameterized queries instead of constructing SQL queries through string concatenation.

### Example 3: Improper Input Handling

**Vulnerable implementation:**

```javascript
app.post("/search", (req, res) => {
    const query = req.body.q.toLowerCase();

    res.send(query);
});
```

**Problem:**

If `q` is missing or is not a string, the application may throw an exception.

**Secure implementation:**

```javascript
app.post("/search", (req, res) => {
    const query = req.body?.q;

    if (typeof query !== "string" || query.length > 200) {
        return res.status(400).json({
            error: "Bad Request",
            message: "Invalid search input."
        });
    }

    res.json({
        search: query.toLowerCase()
    });
});
```

**Improvement:**

- Checks whether the parameter exists.
- Validates the data type.
- Applies a maximum length.
- Returns a controlled response.

### Example 4: Inconsistent Authentication Errors

**Vulnerable behavior:**

```javascript
if (!user) {
    return res.status(404).json({
        message: "User not found"
    });
}

if (!passwordMatches) {
    return res.status(401).json({
        message: "Incorrect password"
    });
}
```

**Problem:**

Different messages reveal whether an account exists.

**Secure behavior:**

```javascript
if (!user || !passwordMatches) {
    return res.status(401).json({
        message: "Invalid email or password"
    });
}
```

**Improvement:**

Use consistent client-facing authentication errors. Also consider rate limiting and monitoring repeated login attempts.

---

## 9. Impact of Improper Error Handling

| Impact | Explanation |
|---|---|
| Information Disclosure | Internal application information may be exposed. |
| Reconnaissance | Attackers may learn about the application's structure. |
| Database Information Leakage | Database errors may reveal schema or query details. |
| Attack Surface Discovery | Stack traces may reveal internal functions and components. |
| Account Enumeration | Different responses may reveal registered accounts. |
| Application Instability | Unhandled exceptions may interrupt normal operations. |
| Further Exploitation | Disclosed details may help attackers investigate other weaknesses. |

The actual severity depends on the sensitivity of the information disclosed and whether it enables further exploitation.

---

## 10. Mitigation and Prevention

### 10.1 Generic Client-Facing Error Messages

Avoid exposing internal technical details to users.

Example:

```json
{
  "error": "Internal Server Error",
  "message": "Something went wrong. Please try again later."
}
```

### 10.2 Centralized Exception Handling

Implement a centralized error-handling mechanism to process application exceptions consistently.

### 10.3 Secure Server-Side Logging

- Record detailed exception information on the server.
- Restrict log access.
- Avoid logging passwords, authentication tokens, or sensitive personal information.
- Protect logs against unauthorized access.

### 10.4 Disable Debug Mode in Production

Ensure production environments do not expose:

- Stack traces
- Debug pages
- Environment variables
- Internal configuration
- Detailed framework errors

### 10.5 Input Validation

Validate:

- Data types
- Input lengths
- Required fields
- Allowed characters
- JSON structures
- File metadata

### 10.6 Secure Database Error Handling

- Use parameterized queries.
- Avoid exposing database error messages.
- Handle database exceptions safely.
- Log diagnostic details securely.

### 10.7 Consistent Authentication Responses

Avoid error messages that reveal whether a username or email address is registered.

### 10.8 Security Testing

Perform negative testing and verify that unexpected inputs do not expose sensitive information or cause unsafe application behavior.

---

## 11. Evidence and Screenshot Structure

Save screenshots of your testing results in the following structure:

```text
09-Error-Handling/
│
├── README.md
│
├── Evidence/


```

**Screenshot guidelines:**

- Capture the original request.
- Capture the modified request.
- Capture the server response.
- Highlight the exposed sensitive information.
- Include the endpoint and HTTP status code.
- Capture the secure response after remediation, if you have implemented a fix.

Only include evidence for vulnerabilities you actually reproduced. Do not create empty or misleading evidence screenshots.

---

## 12. Vulnerability Report Template

Use the following format to document each confirmed finding.

### Finding: Improper Error Handling

| Field | Details |
|---|---|
| Finding ID | EH-001 |
| Vulnerability | Improper Error Handling |
| Affected Application | OWASP Juice Shop |
| Affected Endpoint | Add tested endpoint |
| OWASP Mapping | A10:2025 – Mishandling of Exceptional Conditions |
| CWE | CWE-209, if sensitive error information is disclosed |
| Severity | Based on actual impact |
| Status | Open / Fixed / Retested |

**Description:**

The application returned detailed error information after receiving an unexpected request. The response exposed internal implementation details that should not be accessible to normal users.

**Steps to Reproduce:**

1. Open OWASP Juice Shop.
2. Capture the relevant request using Burp Suite.
3. Send the request to Repeater.
4. Modify the input parameter.
5. Send the modified request.
6. Observe the server response.
7. Record the sensitive information exposed.

**Evidence:**

Add the relevant screenshots from the Evidence directory.

**Impact:**

An attacker may use the disclosed information to understand internal application behavior and identify additional attack surfaces.

**Recommendation:**

Implement centralized exception handling, return generic error messages, and store detailed diagnostic information in protected server-side logs.

**Retest Result:**

Document whether the information disclosure was eliminated after remediation.

---

## 13. Testing Checklist

- [ ] Identified application API endpoints.
- [ ] Captured normal requests.
- [ ] Tested missing parameters.
- [ ] Tested null values.
- [ ] Tested unexpected data types.
- [ ] Tested malformed JSON.
- [ ] Tested special characters.
- [ ] Checked verbose error messages.
- [ ] Checked stack trace disclosure.
- [ ] Checked database error disclosure.
- [ ] Checked internal file path disclosure.
- [ ] Checked debug information exposure.
- [ ] Compared authentication error responses.
- [ ] Analyzed requests using Burp Suite Repeater.
- [ ] Inspected relevant source code.
- [ ] Documented confirmed findings.
- [ ] Captured evidence screenshots.
- [ ] Recommended remediation.
- [ ] Performed retesting where possible.

---

## 14. References

1. OWASP Top 10 – 2025  
   https://owasp.org/Top10/2025/

2. OWASP Error Handling Cheat Sheet  
   https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html

3. CWE-209 – Information Exposure Through Error Messages  
   https://cwe.mitre.org/data/definitions/209.html

4. CWE-755 – Improper Handling of Exceptional Conditions  
   https://cwe.mitre.org/data/definitions/755.html

5. CWE-703 – Improper Check or Handling of Exceptional Conditions  
   https://cwe.mitre.org/data/definitions/703.html

6. OWASP Juice Shop  
   https://github.com/juice-shop/juice-shop

7. PortSwigger Web Security Academy  
   https://portswigger.net/web-security

---

## 15. Conclusion

Error handling is an important aspect of secure application development.

During this assessment, the focus is on identifying whether OWASP Juice Shop exposes sensitive information through error messages, stack traces, database errors, internal file paths, or inconsistent application responses.

Burp Suite helps capture and manipulate HTTP requests, while source-code analysis helps identify the root cause of insecure error handling.

Proper exception handling, input validation, secure logging, and generic client-facing error messages help reduce information disclosure and improve application security.

**Primary OWASP Mapping:** A10:2025 – Mishandling of Exceptional Conditions

**Project:** OWASP Juice Shop – Web Application Security Testing

**Tools:** Burp Suite, Kali Linux, Docker, OWASP Juice Shop
