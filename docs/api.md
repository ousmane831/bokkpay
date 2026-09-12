# BokkPay — API Specification

## 1. Overview

The BokkPay backend exposes a REST API consumed by the Flutter mobile application.

The API is responsible for:

* Authentication
* User management
* Campaign management
* Contributions
* Payments
* Payment status
* Bitnob integration

The API uses JSON for requests and responses.

---

## 2. Base URL

During local development:

```text
http://127.0.0.1:8000/api/
```

The production URL will be defined before deployment.

---

## 3. Authentication

Authentication endpoints:

```text
POST /api/auth/register/
POST /api/auth/login/
POST /api/auth/logout/
GET  /api/auth/me/
```

### Register

```http
POST /api/auth/register/
```

Example request:

```json
{
  "first_name": "Ousmane",
  "last_name": "Diop",
  "email": "ousmane@example.com",
  "password": "********",
  "phone_number": "+221770000000"
}
```

Example response:

```json
{
  "id": "uuid",
  "first_name": "Ousmane",
  "last_name": "Diop",
  "email": "ousmane@example.com"
}
```

---

## 4. Campaign API

### List campaigns

```http
GET /api/campaigns/
```

Returns publicly available campaigns.

Example:

```json
{
  "count": 2,
  "results": [
    {
      "id": "uuid",
      "title": "Help Mamadou",
      "goal_amount": "500000.00",
      "collected_amount": "275000.00",
      "currency": "XOF",
      "status": "ACTIVE"
    }
  ]
}
```

---

### Campaign details

```http
GET /api/campaigns/{campaign_id}/
```

Returns the details of a campaign.

---

### Create campaign

```http
POST /api/campaigns/
```

Authentication required.

Example request:

```json
{
  "title": "Help Mamadou",
  "description": "Community fundraising campaign.",
  "goal_amount": "500000.00",
  "currency": "XOF",
  "deadline": "2026-10-30T23:59:59Z"
}
```

---

### Update campaign

```http
PATCH /api/campaigns/{campaign_id}/
```

Only the campaign owner should be allowed to update their campaign.

---

### Delete campaign

```http
DELETE /api/campaigns/{campaign_id}/
```

Only authorized users should be allowed to delete their campaigns.

---

## 5. Contribution API

### Create contribution

```http
POST /api/campaigns/{campaign_id}/contributions/
```

Example request:

```json
{
  "amount": "25000.00",
  "currency": "XOF"
}
```

The contributor may be authenticated or anonymous depending on the final product design.

The backend validates the amount before creating the contribution.

---

### Contribution details

```http
GET /api/contributions/{contribution_id}/
```

Returns the contribution and its current status.

Example:

```json
{
  "id": "uuid",
  "campaign_id": "uuid",
  "amount": "25000.00",
  "currency": "XOF",
  "status": "PENDING"
}
```

---

## 6. Payment API

### Initiate payment

```http
POST /api/payments/
```

Example request:

```json
{
  "contribution_id": "uuid"
}
```

The backend creates or initiates the payment through the configured payment provider.

The mobile application must not communicate directly with Bitnob using private credentials.

---

### Payment status

```http
GET /api/payments/{payment_id}/
```

Example response:

```json
{
  "id": "uuid",
  "status": "PROCESSING",
  "amount": "25000.00",
  "currency": "XOF"
}
```

---

## 7. Bitnob Webhook

The backend will expose a webhook endpoint for payment events.

```http
POST /api/payments/webhook/bitnob/
```

This endpoint is called by Bitnob when a relevant payment event occurs.

Expected flow:

```text
Bitnob
   │
   │ POST webhook
   ▼
BokkPay API
   │
   ├── Validate request
   ├── Identify payment
   ├── Check transaction
   ├── Prevent duplicate processing
   └── Update payment status
```

The exact webhook payload and authentication mechanism must follow the official Bitnob API documentation available during the hackathon.

---

## 8. Organizer Dashboard API

The organizer needs access to information about their campaigns.

Potential endpoint:

```http
GET /api/dashboard/
```

Example response:

```json
{
  "campaigns": 3,
  "total_collected": "1250000.00",
  "total_contributions": 47
}
```

The exact dashboard structure may evolve during implementation.

---

## 9. HTTP Status Codes

The API should use standard HTTP status codes.

| Status | Meaning                                  |
| ------ | ---------------------------------------- |
| 200    | Successful request                       |
| 201    | Resource created                         |
| 204    | Successful request with no response body |
| 400    | Invalid request                          |
| 401    | Authentication required                  |
| 403    | Permission denied                        |
| 404    | Resource not found                       |
| 409    | Conflict                                 |
| 422    | Validation error                         |
| 429    | Too many requests                        |
| 500    | Internal server error                    |

---

## 10. Error Format

API errors should use a consistent JSON structure.

Example:

```json
{
  "error": {
    "code": "INVALID_AMOUNT",
    "message": "The contribution amount is invalid."
  }
}
```

This makes it easier for the Flutter application to display useful error messages.

---

## 11. Authentication Headers

Authenticated requests will use an authorization mechanism defined by the backend.

Example:

```http
Authorization: Bearer <access_token>
```

Tokens must never be hard-coded into the application source code.

---

## 12. API Security

The API must:

* Validate authentication tokens.
* Validate user permissions.
* Validate request data.
* Validate monetary amounts.
* Prevent unauthorized campaign modification.
* Prevent unauthorized payment modification.
* Protect webhook endpoints.
* Never expose Bitnob private credentials.
* Apply appropriate rate limiting to sensitive endpoints.

---

## 13. Campaign Access Rules

### Public users

Public users can:

* View active campaigns.
* View campaign details.
* Start a contribution.

### Authenticated users

Authenticated users can:

* Create campaigns.
* Manage their campaigns.
* View their contributions.
* Access their dashboard.

### Campaign owners

Campaign owners can:

* Update their own campaigns.
* View contribution information for their campaigns.
* Access campaign statistics.

A user must not be able to modify another user's campaign.

---

## 14. Payment Access Rules

Payment records contain sensitive financial information.

The backend must ensure that users can only access payment information they are authorized to view.

Payment confirmation must never be controlled by the Flutter client.

The server remains the source of truth for payment status.

---

## 15. API Versioning

The API should be designed to support future versioning.

Possible structure:

```text
/api/v1/
```

For example:

```text
/api/v1/auth/
 /api/v1/campaigns/
 /api/v1/contributions/
 /api/v1/payments/
```

The final versioning strategy will be decided during implementation.

---

## 16. API Documentation

The backend should provide machine-readable API documentation during development.

Possible technologies include:

* OpenAPI
* Swagger
* Redoc

The final documentation tool will be selected during implementation.

---

## 17. API Design Principle

The API should remain:

* Simple
* Consistent
* Secure
* Predictable
* Easy to consume from Flutter

Payment-related operations must always be handled server-side.

The exact endpoints and request/response structures may change during the hackathon according to the final BokkPay product design and Bitnob API capabilities.
