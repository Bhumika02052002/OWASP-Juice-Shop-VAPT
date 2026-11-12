## 6 Step-by-Step CSRF Vulnerability Discovery

### Step 1: Identify Potentially Vulnerable Functionality

1. Open the OWASP Juice Shop application.
2. Log in using your authorized test account.
3. Explore features that modify user or application data.
4. Identify actions such as:
   - Change username
   - Update email address
   - Change password
   - Update account details
   - Add or modify saved information
   - Perform other state-changing operations
5. Note the original values before testing.

**Objective:** Identify functionality that performs sensitive actions using the authenticated user's session.

### Step 2: Intercept the HTTP Request Using Burp Suite

1. Open Burp Suite.
2. Go to **Proxy → Intercept**.
3. Enable interception.
4. Open the Juice Shop application through the configured browser.
5. Perform an action such as changing your username.
6. Capture the corresponding HTTP request.
7. Send the request to **Repeater** for further analysis.

Record the following information:

- Request method (GET, POST, PUT, PATCH or DELETE).
- Request URL and endpoint.
- Query parameters.
- Form parameters or JSON body.
- Authentication cookies.
- Authorization headers.
- CSRF token, if present.
- Origin and Referer headers.
- Content-Type header.

### Step 3: Identify CSRF-Related Parameters

Inspect the captured HTTP request and look for:

| Parameter / Header | What to Check |
|---|---|
| `csrf` | Check whether a CSRF token is supplied. |
| `_csrf` | Check for application-specific CSRF protection. |
| `csrf_token` | Check whether the token is validated by the server. |
| `Origin` | Check whether the server validates the request origin. |
| `Referer` | Check whether the request source is verified. |
| `Cookie` | Check whether authentication depends on automatically attached cookies. |
| `Content-Type` | Check which request content types the endpoint accepts. |
| `Authorization` | Determine whether the application uses bearer tokens or another authorization mechanism. |

**Important:** These are common names and headers to investigate, not mandatory parameters. Actual parameter names depend on the application.

### Step 4: Analyze Authentication Mechanisms

Determine how the application authenticates users.

Check whether authentication relies on:

- Session cookies.
- JWT stored in cookies.
- JWT stored in browser local storage and sent through an Authorization header.
- Other authentication mechanisms.

**Why this matters:**

Traditional CSRF commonly affects applications that automatically attach authentication cookies to cross-site requests.

If authentication requires a bearer token manually added to an Authorization header from JavaScript, a normal cross-site HTML form generally cannot attach that token.

Therefore, do not classify an application as CSRF-vulnerable merely because a CSRF token is absent.

### Step 5: Check for Missing or Improper CSRF Protection

Review the captured state-changing request.

Check the following:

1. Is a CSRF token required?
2. Does the server reject requests with a missing token?
3. Does the server reject an invalid token?
4. Is the token unpredictable and validated server-side?
5. Does the application validate the Origin or Referer header?
6. Are authentication cookies configured with appropriate SameSite attributes?
7. Can a state-changing action be triggered using GET?
8. Does the browser attach the authenticated session to a cross-site request?

A missing token is an indicator for further investigation, not sufficient proof of a vulnerability.

### Step 6: Create a CSRF Proof of Concept (PoC)

For an authorized lab test:

1. Copy the relevant state-changing request.
2. Identify the required endpoint and parameters.
3. Determine whether the endpoint accepts browser-compatible form submissions.
4. Create a local HTML file containing a form.
5. Replace the placeholders with the actual endpoint and parameter names from the captured request.
6. Open the PoC in a separate browser tab while the test account is authenticated.
7. Submit the form.
8. Check whether the application processes the request.

**Example HTML PoC template:**

```html
<!DOCTYPE html>
<html>
<head>
    <title>CSRF PoC</title>
</head>
<body>

<h2>CSRF Testing</h2>

<form action="https://YOUR-LAB-HOST/YOUR-ENDPOINT"
      method="POST">

    <input type="hidden"
           name="YOUR_PARAMETER"
           value="TEST_VALUE">

    <button type="submit">
        Submit CSRF Test
    </button>

</form>

</body>
</html>
```

This is a generic template. It must be adapted to the actual request captured from the authorized application. It will not work against every endpoint.

### Step 7: Verify the Vulnerability

After submitting the PoC:

1. Return to the Juice Shop application.
2. Check whether the intended account information changed.
3. Compare the result with the original value.
4. Verify that the request was initiated from the separate PoC page.
5. Confirm whether the browser included the authenticated session.
6. Check whether the server accepted the request without valid CSRF protection.
7. Restore the original account information.

**A CSRF vulnerability is confirmed only when an authenticated browser can perform an unintended state-changing action through a cross-site request without effective CSRF protection.**

### Step 8: Document the Findings

Record the following information in the VAPT report:

| Field | Description |
|---|---|
| Vulnerability | Cross-Site Request Forgery (CSRF) |
| Affected Functionality | Username / Profile Update |
| HTTP Method | Actual captured method |
| Endpoint | Actual captured endpoint |
| Parameter | Actual affected parameter |
| Authentication | Cookie / Token / Other |
| CSRF Protection | Present / Missing / Improper |
| Proof of Concept | HTML PoC and captured request |
| Evidence | Screenshots of request and resulting change |
| Result | Confirmed / Not Confirmed / Needs Further Validation |

---

## 6. CSRF Testing Checklist

- [ ] Identify state-changing functionality.
- [ ] Capture the legitimate request using Burp Suite.
- [ ] Identify the HTTP method and endpoint.
- [ ] Inspect request parameters and headers.
- [ ] Identify the authentication mechanism.
- [ ] Check for CSRF tokens.
- [ ] Test missing and invalid CSRF tokens.
- [ ] Inspect Origin and Referer validation.
- [ ] Check SameSite cookie configuration.
- [ ] Check whether GET requests modify application state.
- [ ] Create an authorized HTML PoC where applicable.
- [ ] Test using a separate browser context.
- [ ] Verify whether the server accepts the cross-site request.
- [ ] Capture screenshots as evidence.
- [ ] Restore the original account information.
- [ ] Document the result and recommended mitigation.

## 7. Important Testing Notes

**Do not confuse these situations:**

- Missing CSRF token does not automatically mean CSRF vulnerability.
- A successful Burp Repeater request does not prove CSRF.
- A username change alone does not prove that the request was cross-site.
- JWT authentication through an Authorization header may change the CSRF attack surface.
- SameSite cookies provide protection in supported browser contexts but should not be the only security control.

The final finding must be supported by an actual browser-based proof of concept and verified server-side state change.
