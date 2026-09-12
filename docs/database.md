# BokkPay — Database Design

## 1. Overview

BokkPay uses PostgreSQL as its primary relational database.

The database is designed to support:

* User management
* Fundraising campaigns
* Contributions
* Payment transactions
* Payment status tracking
* Bitnob transaction references
* Auditability and data consistency

The database schema may evolve during the hackathon based on the final product requirements and Bitnob API capabilities.

---

## 2. Main Entities

The initial database model contains four main entities:

```text
User
 │
 └── Campaign
       │
       └── Contribution
              │
              └── Payment
```

---

## 3. User

The `User` entity represents an authenticated BokkPay user.

### Fields

| Field        | Type     | Description           |
| ------------ | -------- | --------------------- |
| id           | UUID     | Unique identifier     |
| email        | string   | User email address    |
| password     | string   | Hashed password       |
| first_name   | string   | User first name       |
| last_name    | string   | User last name        |
| phone_number | string   | Phone number          |
| created_at   | datetime | Account creation date |
| updated_at   | datetime | Last update date      |

### Responsibilities

A user can:

* Create campaigns
* Manage their campaigns
* View campaign contributions
* Access their organizer dashboard

A user may also contribute to campaigns.

---

## 4. Campaign

The `Campaign` entity represents a fundraising campaign.

### Fields

| Field            | Type     | Description                   |
| ---------------- | -------- | ----------------------------- |
| id               | UUID     | Unique identifier             |
| owner_id         | UUID     | Campaign owner                |
| title            | string   | Campaign title                |
| description      | text     | Campaign description          |
| goal_amount      | decimal  | Target amount                 |
| collected_amount | decimal  | Total confirmed contributions |
| currency         | string   | Campaign currency             |
| image            | string   | Campaign image reference      |
| deadline         | datetime | Campaign deadline             |
| status           | enum     | Campaign status               |
| created_at       | datetime | Creation date                 |
| updated_at       | datetime | Last update date              |

### Campaign statuses

```text
DRAFT
ACTIVE
COMPLETED
EXPIRED
CANCELLED
```

### Relationships

One user can create multiple campaigns.

```text
User 1 ─────────── N Campaign
```

---

## 5. Contribution

The `Contribution` entity represents a financial contribution made toward a campaign.

### Fields

| Field          | Type     | Description                   |
| -------------- | -------- | ----------------------------- |
| id             | UUID     | Unique identifier             |
| campaign_id    | UUID     | Target campaign               |
| contributor_id | UUID     | Contributor, if authenticated |
| amount         | decimal  | Contribution amount           |
| currency       | string   | Contribution currency         |
| status         | enum     | Contribution status           |
| created_at     | datetime | Creation date                 |
| updated_at     | datetime | Last update date              |

### Contribution statuses

```text
PENDING
CONFIRMED
FAILED
CANCELLED
```

### Anonymous contributions

The `contributor_id` field may be nullable.

This allows a contributor to make a contribution without creating a BokkPay account.

Example:

```text
Campaign
   │
   ├── Contribution → authenticated user
   │
   ├── Contribution → anonymous contributor
   │
   └── Contribution → authenticated user
```

---

## 6. Payment

The `Payment` entity represents the payment transaction associated with a contribution.

### Fields

| Field              | Type     | Description                     |
| ------------------ | -------- | ------------------------------- |
| id                 | UUID     | Unique identifier               |
| contribution_id    | UUID     | Related contribution            |
| provider           | string   | Payment provider                |
| provider_reference | string   | Provider transaction identifier |
| amount             | decimal  | Payment amount                  |
| currency           | string   | Payment currency                |
| status             | enum     | Payment status                  |
| payment_method     | string   | Payment method                  |
| created_at         | datetime | Payment creation date           |
| updated_at         | datetime | Last update date                |

### Payment statuses

```text
PENDING
PROCESSING
CONFIRMED
FAILED
EXPIRED
CANCELLED
```

The payment status must be updated based on verified payment events.

---

## 7. Relationships

The main relationships are:

```text
User
 │
 │ 1:N
 ▼
Campaign
 │
 │ 1:N
 ▼
Contribution
 │
 │ 1:1
 ▼
Payment
```

### Detailed relationship

```text
User
 │
 ├───────────────┐
 │               │
 │ 1:N           │ 1:N
 ▼               ▼
Campaign      Contribution
 │               │
 │ 1:N           │ 1:1
 ▼               ▼
Contribution   Payment
```

A campaign can have many contributions.

A contribution should have one associated payment record.

---

## 8. Amount Management

Financial amounts must not be stored using floating-point database types.

The system should use decimal/numeric fields.

For example:

```text
goal_amount       = 500000.00
collected_amount  = 275000.00
contribution      = 25000.00
```

The currency must always be stored explicitly.

Example:

```text
amount: 25000.00
currency: XOF
```

This prevents ambiguity when supporting multiple currencies.

---

## 9. Campaign Progress

Campaign progress is calculated from confirmed contributions.

Example:

```text
Goal:
500,000 XOF

Confirmed contributions:
300,000 XOF

Progress:
60%
```

Conceptually:

```text
progress = confirmed_amount / goal_amount × 100
```

Only confirmed contributions should increase the campaign's collected amount.

Pending or failed payments must not be counted.

---

## 10. Payment Consistency

The following rule is critical:

```text
Payment CONFIRMED
        │
        ▼
Contribution CONFIRMED
        │
        ▼
Campaign collected_amount updated
```

A client application must never be allowed to directly mark a contribution as confirmed.

Confirmation must happen through trusted backend logic.

---

## 11. Bitnob References

The `Payment` entity must store the external payment reference returned by Bitnob.

Example:

```text
provider:
bitnob

provider_reference:
<transaction-reference>
```

The exact fields and identifiers will depend on the Bitnob API available during the hackathon.

These references allow BokkPay to correlate an internal payment with an external Bitnob transaction.

---

## 12. Idempotency

Payment processing must be idempotent.

If the same payment event is received multiple times, BokkPay must not create duplicate contributions or count the same payment twice.

Example:

```text
Bitnob event
     │
     ▼
Check provider_reference
     │
     ├── Already processed → Ignore safely
     │
     └── New event → Process
```

This is especially important for webhook processing.

---

## 13. Database Constraints

The database should enforce important integrity rules.

Examples:

* Campaign owner must reference an existing user.
* Contribution must reference an existing campaign.
* Payment must reference an existing contribution.
* Monetary amounts must be positive.
* Provider references should be unique where appropriate.
* Required fields must not be null.
* Invalid statuses must be rejected.

---

## 14. Indexing

Indexes should be considered for frequently queried fields.

Potential indexes include:

```text
Campaign.owner_id
Campaign.status
Campaign.deadline

Contribution.campaign_id
Contribution.status

Payment.contribution_id
Payment.provider_reference
Payment.status
```

The final indexes will be determined after analyzing the implemented queries.

---

## 15. Data Integrity

Financial data must be handled carefully.

The application should follow these principles:

1. Never trust amounts received directly from the client.
2. Validate all payment information on the backend.
3. Never manually mark payments as confirmed from the mobile application.
4. Keep external provider references.
5. Prevent duplicate payment processing.
6. Use database transactions for critical financial updates.

---

## 16. Example Payment Lifecycle

```text
Contribution created
        │
        ▼
Payment created
        │
        ▼
PENDING
        │
        ▼
Bitnob payment initiated
        │
        ▼
PROCESSING
        │
        ├───────────────┐
        │               │
        ▼               ▼
CONFIRMED            FAILED
        │
        ▼
Contribution CONFIRMED
        │
        ▼
Campaign total updated
```

---

## 17. Future Extensions

The initial database design may later support:

* Campaign categories
* Campaign images
* Multiple beneficiaries
* Refunds
* Notifications
* Transaction history
* Recurring contributions
* Multi-currency campaigns
* Bitcoin payments
* Stablecoin payments
* Payment receipts
* Administrative moderation

These features are outside the initial MVP unless required during the hackathon.

---

## 18. Implementation Principle

The database model should remain simple enough to implement within the hackathon while providing a reliable foundation for payment processing.

The final Django models must reflect the actual Bitnob API capabilities and the final product requirements decided by the team during Code & Chain Dakar 2026.
