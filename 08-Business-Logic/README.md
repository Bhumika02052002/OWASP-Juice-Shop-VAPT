# 08 - Business Logic Vulnerabilities

## 1. Introduction

Business Logic Vulnerabilities occur when an application does not correctly enforce the rules, workflows, restrictions or conditions that control its business operations.

Unlike traditional vulnerabilities that may result from insecure coding functions, business logic vulnerabilities often occur because the application allows users to perform actions in ways that were never intended by the developers.

An attacker may manipulate application workflows, modify request parameters, skip required steps, or abuse legitimate functionality to achieve unauthorized results.

### Examples

- Purchasing products below their intended price.
- Applying discounts multiple times.
- Manipulating product quantities.
- Bypassing payment verification.
- Accessing functionality without completing required steps.
- Modifying another user's order.
- Abusing referral or coupon systems.
- Performing actions in an incorrect sequence.

### Potential Impact

- Financial loss.
- Unauthorized purchases.
- Unauthorized discounts.
- Inventory manipulation.
- Account or order manipulation.
- Business process abuse.
- Unauthorized access to restricted functionality.
- Loss of application integrity.

**Project:** OWASP Juice Shop VAPT  
**Module:** 08 - Business Logic Vulnerabilities  
**Target:** OWASP Juice Shop  
**Environment:** Kali Linux / VMware Workstation  
**Tools:** Burp Suite, Browser Developer Tools, OWASP Juice Shop

> All testing described in this documentation must be performed against an authorized local lab or another explicitly authorized application.

---

# 2. Objectives

The objectives of business logic testing are:

1. Understand the application's intended business workflows.
2. Identify important business rules.
3. Identify sensitive operations.
4. Inspect application requests using Burp Suite.
5. Test parameter manipulation.
6. Investigate price and quantity manipulation.
7. Test coupon and discount restrictions.
8. Investigate payment workflow weaknesses.
9. Test access control within business workflows.
10. Identify workflow and state validation issues.
11. Analyze source code.
12. Understand vulnerable and fixed implementations.
13. Recommend business logic security improvements.
14. Verify remediation through retesting.

---

# 3. Types of Business Logic Vulnerabilities

## 3.1 Price Manipulation

### Explanation

Price manipulation occurs when an application trusts prices supplied by the client instead of calculating or verifying the correct price on the server.

An attacker may attempt to modify the product price during a purchase request.

### Example

Suppose a product costs ₹5,000.

The legitimate checkout request contains:

```json
{
  "productId": 101,
  "quantity": 1,
  "price": 5000
}
```

A user attempts to modify the price:

```json
{
  "productId": 101,
  "quantity": 1,
  "price": 10
}
```

### Testing Steps

1. Open OWASP Juice Shop.
2. Navigate to the product catalog.
3. Select a product.
4. Add it to the shopping basket.
5. Open Burp Suite.
6. Navigate to Proxy → HTTP History.
7. Identify the request related to the basket or checkout.
8. Send the request to Repeater.
9. Inspect the product ID, quantity and price-related values.
10. Determine whether the application accepts client-supplied price values.
11. In the authorized lab, test whether modifying a price affects the server-calculated total.
12. Verify the actual result without completing an unauthorized purchase.

### Vulnerable Code Example

```javascript
app.post("/checkout", (req, res) => {

    const price = req.body.price;
    const quantity = req.body.quantity;

    const total = price * quantity;

    res.json({
        total: total
    });

});
```

### Why Vulnerable?

The server trusts the price supplied by the client.

### Fixed Code Example

```javascript
app.post("/checkout", async (req, res) => {

    const product = await Product.findById(req.body.productId);

    if (!product) {
        return res.status(404).json({
            message: "Product not found"
        });
    }

    const quantity = Number(req.body.quantity);

    if (!Number.isInteger(quantity) || quantity < 1) {
        return res.status(400).json({
            message: "Invalid quantity"
        });
    }

    const total = product.price * quantity;

    res.json({
        total: total
    });

});
```

### Why Fixed?

- Price is retrieved from trusted server-side product data.
- Quantity is validated.
- Client-supplied price is not trusted.
- The server calculates the total.

---

## 3.2 Quantity Manipulation

### Explanation

Quantity manipulation occurs when an application does not correctly validate the quantity of a product or service requested by a user.

An attacker may attempt to submit negative, zero, extremely large or otherwise invalid quantities.

### Testing Steps

1. Open Juice Shop.
2. Select a product.
3. Add it to the basket.
4. Capture the related HTTP request.
5. Inspect the quantity parameter.
6. Send the request to Repeater.
7. Review how the application handles quantity values.
8. Test boundary values within the authorized lab.
9. Check whether negative or invalid values are rejected.
10. Verify that the server correctly calculates the basket total.

### Example Test Values

```text
1
0
-1
999999
```

### Vulnerable Code Example

```javascript
const quantity = req.body.quantity;

const total = product.price * quantity;
```

### Fixed Code Example

```javascript
const quantity = Number(req.body.quantity);

if (
    !Number.isInteger(quantity) ||
    quantity < 1 ||
    quantity > 100
) {
    return res.status(400).json({
        message: "Invalid quantity"
    });
}
```

The maximum quantity must be based on actual business requirements and inventory rules.

### Why Fixed?

- Rejects invalid quantities.
- Prevents negative quantities.
- Enforces reasonable business limits.
- Prevents unexpected calculations.

---

## 3.3 Coupon and Discount Abuse

### Explanation

Coupon abuse occurs when an application does not correctly enforce discount rules.

Examples include:

- Applying the same coupon multiple times.
- Reusing an expired coupon.
- Applying a single-use coupon repeatedly.
- Combining incompatible discounts.
- Applying a coupon to an ineligible product.
- Using a coupon belonging to another account.

### Testing Steps

1. Open Juice Shop.
2. Explore the shopping basket and checkout functionality.
3. Identify discount or coupon functionality, if available.
4. Capture the relevant request using Burp Suite.
5. Inspect coupon-related parameters.
6. Review whether discount calculations occur on the client or server.
7. Check whether the server validates coupon eligibility.
8. Test duplicate redemption using an authorized test account.
9. Check whether expired or invalid coupons are rejected.
10. Verify whether discounts are applied according to the defined rules.

### Vulnerable Code Example

```javascript
app.post("/apply-coupon", (req, res) => {

    const discount = req.body.discount;

    const total = req.body.total - discount;

    res.json({
        total: total
    });

});
```

### Why Vulnerable?

The application trusts a discount value supplied by the client.

### Fixed Code Example

```javascript
app.post("/apply-coupon", async (req, res) => {

    const coupon = await Coupon.findOne({
        code: req.body.code
    });

    if (!coupon || !coupon.active) {
        return res.status(400).json({
            message: "Invalid coupon"
        });
    }

    if (coupon.expiryDate < new Date()) {
        return res.status(400).json({
            message: "Coupon expired"
        });
    }

    // Verify eligibility and redemption limits
    // on the server before applying the discount.

    res.json({
        message: "Coupon validated"
    });

});
```

A production implementation must also enforce redemption limits atomically to prevent concurrent duplicate use.

### Why Fixed?

- Coupon validity is checked on the server.
- Expiration is verified.
- Eligibility and redemption restrictions can be enforced.
- The client cannot determine its own discount.

---

## 3.4 Payment Logic Manipulation

### Explanation

Payment logic manipulation occurs when an application does not correctly verify whether a transaction has actually been paid before completing an order.

For example, an attacker may attempt to:

- Skip the payment step.
- Modify the payment status.
- Submit an order without successful payment.
- Manipulate the amount being paid.
- Reuse a previous payment confirmation.

### Testing Steps

1. Open Juice Shop.
2. Add a product to the basket.
3. Proceed to checkout.
4. Capture the checkout requests in Burp Suite.
5. Identify payment-related endpoints.
6. Inspect order and payment status parameters.
7. Determine whether the server independently verifies payment.
8. Check whether the application allows order completion without valid payment confirmation.
9. Review whether payment amounts are calculated on the server.
10. Document any incorrect state transition.

### Vulnerable Code Example

```javascript
app.post("/complete-order", (req, res) => {

    const paymentStatus = req.body.paymentStatus;

    if (paymentStatus === "success") {

        return res.json({
            message: "Order completed"
        });

    }

});
```

### Why Vulnerable?

The server trusts a payment status supplied directly by the client.

### Fixed Code Example

```javascript
app.post("/complete-order", async (req, res) => {

    const order = await Order.findById(req.body.orderId);

    if (!order) {
        return res.status(404).json({
            message: "Order not found"
        });
    }

    // Verify payment through a trusted server-side
    // payment provider or verified transaction record.

    const payment = await Payment.findOne({
        orderId: order.id,
        status: "verified"
    });

    if (!payment) {
        return res.status(400).json({
            message: "Payment not verified"
        });
    }

    order.status = "completed";

    await order.save();

    res.json({
        message: "Order completed"
    });

});
```

### Why Fixed?

- Payment status is obtained from a trusted server-side record.
- Client input cannot independently mark an order as paid.
- Order completion depends on verified payment.

A real payment integration must also verify transaction identity, amount, currency, order association and duplicate processing.

---

## 3.5 Workflow Bypass

### Explanation

Workflow bypass occurs when an application allows a user to skip required steps in a business process.

For example, an application may require:

1. Login.
2. Add product.
3. Confirm delivery address.
4. Complete payment.
5. Confirm order.

If the application does not enforce the required sequence on the server, an attacker may attempt to access a later step directly.

### Testing Steps

1. Explore the complete application workflow.
2. Identify the required steps.
3. Capture requests for each step.
4. Record the endpoint sequence.
5. Check whether later endpoints can be accessed directly.
6. Determine whether the server verifies completion of previous steps.
7. Test whether incomplete workflows are rejected.
8. Document any unauthorized state transition.

### Vulnerable Code Example

```javascript
app.post("/confirm-order", (req, res) => {

    // Directly confirms the order.

    res.json({
        message: "Order confirmed"
    });

});
```

### Fixed Code Example

```javascript
app.post("/confirm-order", async (req, res) => {

    const order = await Order.findById(req.body.orderId);

    if (!order) {
        return res.status(404).json({
            message: "Order not found"
        });
    }

    if (order.status !== "payment_verified") {
        return res.status(400).json({
            message: "Previous steps incomplete"
        });
    }

    order.status = "confirmed";

    await order.save();

    res.json({
        message: "Order confirmed"
    });

});
```

### Why Fixed?

- The server verifies the current workflow state.
- Required previous steps must be completed.
- Users cannot simply skip the business process.

---

## 3.6 Unauthorized Order Modification

### Explanation

This vulnerability occurs when users can modify orders that do not belong to them.

An application may use an order ID supplied by the client without verifying ownership.

### Testing Steps

1. Create two authorized test accounts.
2. Create an order using the first account.
3. Log in using the second account.
4. Capture the order-related HTTP requests.
5. Identify the order ID parameter.
6. Review whether the server checks ownership.
7. Test access to an order belonging to the other test account.
8. Verify whether the server rejects unauthorized access.
9. Document the result.

### Vulnerable Code Example

```javascript
app.get("/orders/:id", async (req, res) => {

    const order = await Order.findById(req.params.id);

    res.json(order);

});
```

### Why Vulnerable?

The application retrieves an order without checking whether it belongs to the authenticated user.

### Fixed Code Example

```javascript
app.get("/orders/:id", async (req, res) => {

    const order = await Order.findOne({
        _id: req.params.id,
        userId: req.user.id
    });

    if (!order) {
        return res.status(404).json({
            message: "Order not found"
        });
    }

    res.json(order);

});
```

### Why Fixed?

- Checks the authenticated user's identity.
- Enforces order ownership.
- Prevents unauthorized access to another user's order.

---

## 3.7 Race Condition

### Explanation

A race condition occurs when multiple requests access or modify shared application data at nearly the same time and the application does not correctly handle concurrent operations.

Business logic race conditions may affect:

- Coupon redemption.
- Inventory availability.
- Account balances.
- Transaction processing.
- Limited-time offers.
- Single-use operations.

### Testing Steps

1. Identify an operation with a limited resource or single-use restriction.
2. Understand the expected business rule.
3. Capture the normal request.
4. Use Burp Suite Repeater to investigate request handling.
5. In an authorized lab, test controlled concurrent requests.
6. Check whether the same resource can be consumed more than once.
7. Compare the resulting database or application state.
8. Document any violation of the expected business rule.

### Vulnerable Code Example

```javascript
if (coupon.remainingUses > 0) {

    await applyCoupon(coupon);

    coupon.remainingUses--;

    await coupon.save();

}
```

### Why Vulnerable?

Two requests may read the same remaining-use value before either request updates it.

### Fixed Code Example

Use an atomic database update:

```javascript
const result = await Coupon.updateOne(
    {
        _id: couponId,
        remainingUses: { $gt: 0 }
    },
    {
        $inc: {
            remainingUses: -1
        }
    }
);

if (result.modifiedCount !== 1) {

    return res.status(400).json({
        message: "Coupon unavailable"
    });

}
```

**Why Fixed?**

- Uses an atomic conditional update.
- Prevents multiple requests from consuming the same remaining use.
- Ensures the business rule is enforced at the database level.

For multi-step transactions, use appropriate database transactions and idempotency controls.

---

## 3.8 Negative Value Manipulation

### Explanation

Negative value manipulation occurs when an application does not properly validate numerical inputs.

For example, an application may accept negative quantities, negative prices or negative transaction amounts.

### Testing Steps

1. Identify numerical input fields.
2. Capture the corresponding request.
3. Inspect the numeric parameters.
4. Test valid boundary values.
5. Test negative values in the authorized lab.
6. Observe the resulting calculations.
7. Check whether the server rejects invalid values.
8. Verify that account balances and totals remain consistent.

### Vulnerable Code Example

```javascript
const amount = req.body.amount;

account.balance -= amount;
```

### Why Vulnerable?

There is no validation that the amount is a positive number within permitted business limits.

### Fixed Code Example

```javascript
const amount = Number(req.body.amount);

if (
    !Number.isFinite(amount) ||
    amount <= 0
) {
    return res.status(400).json({
        message: "Invalid amount"
    });
}
```

For financial calculations, use integer minor units or an appropriate decimal arithmetic library rather than floating-point calculations.

---

# 4. Business Logic Testing Methodology Using Burp Suite

## Step 1: Understand the Application

Before testing, understand how the application is intended to work.

Explore:

- User registration.
- Login and logout.
- Product catalog.
- Shopping basket.
- Checkout.
- Coupon functionality.
- Payment process.
- Order history.
- User profile.
- Administrative functionality.

Create a simple workflow diagram.

Example:

```text
Login
  |
  v
Browse Products
  |
  v
Add to Basket
  |
  v
Review Basket
  |
  v
Checkout
  |
  v
Payment
  |
  v
Order Confirmation
```

## Step 2: Capture Normal Requests

1. Open Burp Suite.
2. Navigate to Proxy.
3. Open HTTP History.
4. Use the Juice Shop application.
5. Perform normal business operations.
6. Capture the corresponding requests.
7. Identify the parameters.
8. Save important requests for later testing.

## Step 3: Identify Business Parameters

Investigate the following types of parameters:

| Parameter | Testing objective |
|---|---|
| productId | Check product identification and authorization |
| quantity | Check quantity validation |
| price | Check server-side price calculation |
| total | Check total calculation |
| discount | Check discount validation |
| coupon | Check coupon eligibility |
| orderId | Check order ownership |
| userId | Check user authorization |
| paymentStatus | Check payment verification |
| amount | Check numerical validation |
| status | Check workflow state validation |

These are common examples. Actual parameter names must be identified from the application's requests.

## Step 4: Send Requests to Repeater

1. Right-click a captured request.
2. Select Send to Repeater.
3. Open the Repeater tab.
4. Review the original request.
5. Identify parameters related to the business operation.
6. Modify one test value at a time.
7. Send the request.
8. Compare the server response with the original request.
9. Verify the resulting application state.

## Step 5: Compare Expected and Actual Behavior

For every test, ask:

- What should happen according to the business rule?
- What actually happened?
- Did the server validate the operation?
- Was the request authorized?
- Did the application update its state correctly?
- Could the operation be repeated?
- Could the operation be performed out of sequence?

## Step 6: Document the Result

Record:

- Feature tested.
- Original request.
- Modified request.
- Parameter tested.
- Expected behavior.
- Actual behavior.
- Evidence.
- Impact.
- Recommended remediation.

---

# 5. Business Logic Source Code Analysis

Business logic vulnerabilities frequently exist in the application code responsible for enforcing business rules.

During source-code analysis, investigate:

| Code area | What to investigate |
|---|---|
| Checkout functions | Server-side total calculation |
| Product functions | Product price and availability |
| Coupon functions | Discount validation |
| Payment functions | Payment verification |
| Order functions | Order ownership |
| User functions | Authorization |
| Inventory functions | Stock validation |
| Workflow functions | Required state transitions |
| Database operations | Atomic updates and transaction handling |

## 5.1 Common Source Code Patterns

Search for these patterns in the source code:

```text
req.body
req.query
req.params
price
quantity
discount
coupon
paymentStatus
orderId
userId
balance
remainingUses
```

These terms help identify important business logic. Their presence alone does not prove a vulnerability.

## 5.2 Source-to-Sink Analysis

Example:

```text
User Input
    |
    v
HTTP Request
    |
    v
Backend Controller
    |
    v
Business Rule Validation
    |
    v
Database Operation
    |
    v
Application State
```

Investigate whether the application validates the input and enforces the intended business rule before changing application state.

---

# 6. Jump to Code - Find It and Fix It

## 6.1 Find It

The Find It phase helps identify vulnerable statements in source code.

Steps:

1. Open the Juice Shop Score Board.
2. Identify available coding challenges.
3. Open the relevant coding challenge.
4. Read the description.
5. Review the source-code snippet.
6. Identify where business rules are missing or incorrectly enforced.
7. Select the relevant vulnerable statement.
8. Submit the selection.
9. Review the feedback.

## 6.2 Fix It

The Fix It phase teaches how to correct the vulnerable implementation.

Steps:

1. Open the Fix It stage.
2. Read the proposed code alternatives.
3. Understand the underlying business rule.
4. Compare the original and corrected code.
5. Select the secure implementation.
6. Submit your answer.
7. Read the explanation.
8. Record the vulnerable and fixed implementation.

## 6.3 What to Look For

| Vulnerable implementation | Security concern |
|---|---|
| Trusting req.body.price | Client-side price manipulation |
| Trusting req.body.discount | Unauthorized discounts |
| Missing quantity validation | Invalid quantity values |
| Missing ownership checks | Unauthorized order access |
| Trusting paymentStatus | Payment bypass |
| Missing workflow validation | Workflow bypass |
| Non-atomic database updates | Race conditions |
| Missing authorization checks | Unauthorized business actions |

**Important:** Not every business logic issue has a dedicated Juice Shop Find It/Fix It challenge. Use manual source-code analysis for additional testing.

---

# 7. Vulnerable Code vs Fixed Code Summary

| Vulnerability | Vulnerable implementation | Secure implementation |
|---|---|---|
| Price manipulation | Trust client price | Retrieve price from server |
| Quantity manipulation | Accept unrestricted quantity | Validate integer and limits |
| Coupon abuse | Trust client discount | Validate coupon server-side |
| Payment bypass | Trust client payment status | Verify payment server-side |
| Workflow bypass | Allow direct state changes | Enforce state transitions |
| Order modification | Retrieve order by ID only | Verify ownership |
| Race condition | Read then update separately | Atomic database operations |
| Negative values | No numerical validation | Validate positive amounts |

These examples are educational patterns. Always adapt fixes to the actual application architecture and business requirements.

---

# 8. Business Logic Security Best Practices

## 8.1 Server-Side Validation

Never trust values supplied by the client.

Validate:

- Price.
- Quantity.
- Discount.
- Order status.
- Payment status.
- Account balance.
- Product availability.

## 8.2 Authorization

Verify that users are allowed to perform the requested action.

Check:

- User identity.
- Resource ownership.
- Role permissions.
- Transaction ownership.
- Administrative privileges.

## 8.3 Workflow Enforcement

Implement server-side state machines for important workflows.

For example:

```text
PENDING
   |
   v
PAYMENT_VERIFIED
   |
   v
CONFIRMED
   |
   v
COMPLETED
```

Reject invalid state transitions.

## 8.4 Atomic Database Operations

Use database transactions and atomic updates for operations involving:

- Inventory.
- Account balances.
- Coupon limits.
- Financial transactions.
- Single-use resources.

## 8.5 Idempotency

Use idempotency controls for sensitive operations such as payment processing to prevent duplicate execution when requests are retried.

## 8.6 Audit Logging

Record important business operations.

Examples:

- Price changes.
- Discount application.
- Payment confirmation.
- Order modification.
- Account changes.
- Administrative actions.

Avoid storing sensitive authentication secrets in logs.

---

# 9. Evidence Collection

Create an evidence directory for your assessment.

Suggested folder structure:

```text
08-Business-Logic/
│
├── README.md
│
└── evidence/
```

### Evidence Requirements

For every confirmed finding, capture:

1. Original application behavior.
2. Normal HTTP request.
3. Modified HTTP request.
4. Server response.
5. Actual application state.
6. Vulnerable business rule.
7. Vulnerable source-code location, if identified.
8. Corrected implementation.
9. Retest result.

Use test accounts and avoid real financial transactions.

---

# 10. Business Logic Vulnerability Reporting Template

## Finding: Business Logic Vulnerability

**Vulnerability Name:** Actual vulnerability identified.

**Affected Functionality:** Checkout / Payment / Coupon / Order / Other.

**Affected Endpoint:** Actual endpoint.

**HTTP Method:** Actual method.

**Affected Parameter:** Actual parameter.

**Severity:** Based on demonstrated impact.

### Description

Describe the business rule that is not being correctly enforced.

### Expected Behavior

Explain what the application should allow or reject.

### Actual Behavior

Explain what happened during testing.

### Steps to Reproduce

1. Log in using an authorized test account.
2. Navigate to the affected functionality.
3. Capture the original HTTP request.
4. Identify the relevant business parameter.
5. Perform the controlled test.
6. Verify the resulting application state.

### Proof of Concept

Include the modified request or test scenario.

### Impact

Explain the realistic business impact.

### Root Cause

Identify the missing or incorrect business validation.

### Vulnerable Code

Include the actual source code and line number, if identified.

### Recommended Fix

Provide corrected code and explain the security improvement.

### Retest

Verify that the original test is rejected or handled correctly after remediation.

---

# 11. Business Logic Testing Checklist

### Application Understanding

- [ ] Understand application functionality.
- [ ] Identify business workflows.
- [ ] Identify sensitive operations.
- [ ] Document expected business rules.

### Request Analysis

- [ ] Capture normal requests.
- [ ] Inspect HTTP methods.
- [ ] Identify relevant parameters.
- [ ] Analyze API endpoints.
- [ ] Send requests to Burp Repeater.

### Vulnerability Testing

- [ ] Price manipulation.
- [ ] Quantity manipulation.
- [ ] Coupon and discount abuse.
- [ ] Payment workflow testing.
- [ ] Workflow bypass.
- [ ] Unauthorized order modification.
- [ ] Race condition testing.
- [ ] Negative value manipulation.

### Source Code Analysis

- [ ] Review backend controllers.
- [ ] Identify missing validation.
- [ ] Identify missing authorization.
- [ ] Inspect database operations.
- [ ] Trace business rule enforcement.
- [ ] Complete Find It where available.
- [ ] Complete Fix It where available.

### Remediation

- [ ] Implement server-side validation.
- [ ] Enforce authorization.
- [ ] Validate workflow state.
- [ ] Use atomic database operations.
- [ ] Implement idempotency controls.
- [ ] Retest the original issue.
- [ ] Capture final evidence.

---

# 12. Conclusion

Business Logic Vulnerability Assessment is an important part of web application penetration testing because it evaluates whether an application correctly enforces its intended business rules.

The assessment focuses on identifying weaknesses in application workflows, pricing calculations, quantity validation, discount mechanisms, payment verification, order ownership and database operations.

Using Burp Suite, HTTP request analysis and source-code review helps identify situations where an application trusts client-controlled values or fails to enforce important business conditions.

The objective is not only to discover vulnerabilities but also to understand their root causes, recommend secure implementations and verify remediation.

**Project:** OWASP Juice Shop VAPT  
**Module:** 08 - Business Logic Vulnerabilities  
**Environment:** Kali Linux + VMware + OWASP Juice Shop  
**Tools:** Burp Suite + Browser Developer Tools

---

# 13. References

1. OWASP Juice Shop:
   https://owasp.org/www-project-juice-shop/

2. OWASP Juice Shop Companion Guide:
   https://help.owasp-juice.shop/

3. OWASP Web Security Testing Guide:
   https://owasp.org/www-project-web-security-testing-guide/

4. OWASP Business Logic Testing:
   https://owasp.org/www-project-web-security-testing-guide/

5. OWASP Application Security Verification Standard:
   https://owasp.org/www-project-application-security-verification-standard/

6. OWASP API Security Top 10:
   https://owasp.org/www-project-api-security/

---

**Final Note:** This README documents business logic testing techniques and illustrative secure coding examples. Do not report a vulnerability as confirmed unless the behavior has been reproduced and its impact verified in the authorized test environment.
