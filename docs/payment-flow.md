# BokkPay — Payment Flow

## 1. Overview

Payment processing is one of the most important components of BokkPay.

The payment architecture must ensure that:

* The client never handles private Bitnob credentials.
* Payment status is controlled by the backend.
* Contributions are only confirmed after payment verification.
* Duplicate payment events do not create duplicate contributions.
* The campaign total is updated only after a confirmed payment.

---

## 2. Payment Architecture

The general flow is:

```text
Flutter Mobile App
        │
        │ 1. Create contribution
        ▼
BokkPay Django API
        │
        │ 2. Create payment
        ▼
BokkPay Payment Service
        │
        │ 3. Request payment
        ▼
Bitnob API
        │
        │ 4. Payment processing
        ▼
Bitcoin / Stablecoin Infrastructure
        │
        │ 5. Payment event
        ▼
Bitnob Webhook
        │
        │ 6. Notify BokkPay
        ▼
BokkPay Webhook Endpoint
        │
        │ 7. Validate & process
        ▼
PostgreSQL
        │
        ├── Payment → CONFIRMED
        ├── Contribution → CONFIRMED
        └── Campaign total → UPDATED
```

---

## 3. Step 1 — User Selects an Amount

The contributor opens a campaign.

Example:

```text
Campaign: Help Mamadou

Goal:
500,000 XOF

Collected:
275,000 XOF

Contribution:
25,000 XOF
```

The user selects:

```text
25,000 XOF
```

The Flutter application sends the contribution request to the BokkPay API.

---

## 4. Step 2 — Create Contribution

Flutter sends:

```http
POST /api/campaigns/{campaign_id}/contributions/
```

Example:

```json
{
  "amount": "25000.00",
  "currency": "XOF"
}
```

The backend validates:

* Campaign exists.
* Campaign is active.
* Campaign has not expired.
* Amount is valid.
* Currency is supported.
* Amount is greater than zero.

The backend then creates:

```text
Contribution
status = PENDING
```

---

## 5. Step 3 — Create Payment

The backend creates the associated payment.

Initial state:

```text
Payment
status = PENDING
```

The backend then communicates with Bitnob.

The Flutter application does not communicate directly with Bitnob using private credentials.

---

## 6. Step 4 — Bitnob Processing

The BokkPay backend sends the required payment information to the Bitnob API.

The exact request structure will depend on the Bitnob API capabilities available during the hackathon.

Bitnob processes the payment through its supported payment infrastructure.

The BokkPay backend stores the external provider reference returned by Bitnob.

Example:

```text
provider:
bitnob

provider_reference:
external-transaction-id
```

The payment may now be:

```text
PROCESSING
```

---

## 7. Step 5 — Payment Event

After the payment changes state, Bitnob may notify BokkPay through a webhook.

Example endpoint:

```http
POST /api/payments/webhook/bitnob/
```

The webhook contains information allowing BokkPay to identify and process the payment event.

The exact payload and verification mechanism must follow the official Bitnob API documentation.

---

## 8. Step 6 — Webhook Validation

The BokkPay backend must never blindly trust incoming webhook data.

The webhook handler should:

1. Receive the event.
2. Validate the request.
3. Verify the event authenticity according to Bitnob's documented mechanism.
4. Identify the associated payment.
5. Check the current payment status.
6. Prevent duplicate processing.
7. Verify the relevant transaction information.
8. Update the payment.

Conceptually:

```text
Webhook received
       │
       ▼
Is request valid?
       │
   ┌───┴────┐
   │        │
  NO       YES
   │        │
 Reject     ▼
        Find payment
             │
             ▼
       Already processed?
          │       │
         YES      NO
          │       │
        Ignore    ▼
              Process event
```

---

## 9. Step 7 — Confirm Payment

If the payment is successfully verified:

```text
Payment
   │
   ▼
CONFIRMED
```

Then:

```text
Contribution
   │
   ▼
CONFIRMED
```

Only after this confirmation should the campaign total be updated.

---

## 10. Step 8 — Update Campaign

Suppose the campaign currently has:

```text
Collected:
275,000 XOF
```

The confirmed contribution is:

```text
25,000 XOF
```

The new total becomes:

```text
300,000 XOF
```

The campaign progress becomes:

```text
300,000 / 500,000 × 100

= 60%
```

The campaign should only be updated after the contribution is confirmed.

---

## 11. Complete Successful Flow

```text
┌───────────────┐
│    Flutter    │
└───────┬───────┘
        │
        │ Create contribution
        ▼
┌────────────────────┐
│    Django API      │
└─────────┬──────────┘
          │
          │ Validate
          ▼
┌────────────────────┐
│   Contribution     │
│      PENDING       │
└─────────┬──────────┘
          │
          │ Create payment
          ▼
┌────────────────────┐
│   Payment Service   │
└─────────┬──────────┘
          │
          │ API request
          ▼
┌────────────────────┐
│      Bitnob        │
└─────────┬──────────┘
          │
          │ Payment event
          ▼
┌────────────────────┐
│  BokkPay Webhook   │
└─────────┬──────────┘
          │
          │ Validate
          ▼
┌────────────────────┐
│      Payment       │
│     CONFIRMED      │
└─────────┬──────────┘
          │
          ▼
┌────────────────────┐
│    Contribution    │
│     CONFIRMED      │
└─────────┬──────────┘
          │
          ▼
┌────────────────────┐
│      Campaign      │
│     total +amount  │
└────────────────────┘
```

---

## 12. Failed Payment

If Bitnob reports that a payment failed:

```text
Payment
   │
   ▼
FAILED
   │
   ▼
Contribution
   │
   ▼
FAILED
```

The campaign total must not change.

Example:

```text
Before:
275,000 XOF

Failed contribution:
25,000 XOF

After:
275,000 XOF
```

---

## 13. Cancelled Payment

If a payment is cancelled before confirmation:

```text
Payment
   │
   ▼
CANCELLED
```

The related contribution should not increase the campaign total.

---

## 14. Duplicate Webhooks

The system must handle duplicate webhook events safely.

Example:

```text
Webhook #1
    │
    ▼
Payment confirmed

Webhook #2
    │
    ▼
Same provider reference
    │
    ▼
Already processed
    │
    ▼
Ignore safely
```

The campaign must never receive the same contribution twice.

---

## 15. Database Transaction

Payment confirmation should be handled atomically.

Conceptually:

```text
BEGIN TRANSACTION

Update Payment
       │
       ▼
Update Contribution
       │
       ▼
Update Campaign total

COMMIT
```

If an unexpected error occurs:

```text
ROLLBACK
```

This prevents the database from reaching an inconsistent state.

---

## 16. Idempotency

The payment system must be idempotent.

The backend should use the external provider reference to identify previously processed transactions.

Example:

```text
provider_reference = "abc123"
```

Before processing an event:

```text
Does "abc123" already exist?
```

If yes:

```text
Do not process again.
```

If no:

```text
Process the payment.
```

---

## 17. Security

The following rules are mandatory:

### Never expose private credentials

Bitnob credentials must only exist on the backend.

### Never trust the client

Flutter must not be able to send:

```text
status = CONFIRMED
```

and make the backend accept it.

### Validate amounts

The backend must determine and validate the actual contribution amount.

### Validate webhook events

Webhook requests must be authenticated or verified according to Bitnob's official documentation.

### Protect secrets

Secrets must be stored in environment variables.

Example:

```text
BITNOB_API_KEY=...
BITNOB_SECRET=...
```

These values must never be committed to GitHub.

---

## 18. User Experience

After the payment is initiated, the Flutter application can display:

```text
Payment processing...

Please wait while we confirm your contribution.
```

After confirmation:

```text
Contribution successful!

25,000 XOF contributed to Help Mamadou.
```

After failure:

```text
Payment failed.

Your contribution was not confirmed.
Please try again.
```

The application should retrieve the authoritative payment status from the BokkPay backend.

---

## 19. Payment Status Lifecycle

```text
PENDING
   │
   ▼
PROCESSING
   │
   ├───────────────┐
   │               │
   ▼               ▼
CONFIRMED        FAILED
   │
   │
   └──► Campaign updated
```

Additional terminal states such as `EXPIRED` or `CANCELLED` may be used depending on the final payment implementation.

---

## 20. Important Implementation Rule

The exact Bitnob payment flow must be implemented according to the official Bitnob API documentation available during the hackathon.

The team must not assume that an endpoint, field, webhook event, currency or payment method exists without verifying it in the official documentation.

This document defines the architecture and expected behavior, not a guarantee of the final Bitnob API implementation.

---

## 21. Final Principle

The fundamental rule of BokkPay payment processing is:

```text
Client request
      ↓
Backend validation
      ↓
Bitnob payment
      ↓
Verified webhook/event
      ↓
Payment confirmation
      ↓
Contribution confirmation
      ↓
Campaign update
```

The backend is the source of truth for financial state.
