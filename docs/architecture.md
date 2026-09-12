# BokkPay — System Architecture

## 1. Overview

BokkPay is a community fundraising platform designed to simplify collective contributions and provide transparent campaign tracking.

The platform is designed around three main components:

* **Flutter** — Mobile client
* **Django REST Framework** — Backend API
* **Bitnob API** — Payment infrastructure

PostgreSQL is used as the primary database.

---

## 2. High-Level Architecture

```text
                         ┌─────────────────────┐
                         │       BokkPay       │
                         │    Mobile App       │
                         │      Flutter        │
                         └──────────┬──────────┘
                                    │
                              HTTPS / REST
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │    Django REST      │
                         │        API          │
                         └──────────┬──────────┘
                                    │
                 ┌──────────────────┼──────────────────┐
                 │                  │                  │
                 ▼                  ▼                  ▼
        ┌────────────────┐ ┌────────────────┐ ┌────────────────┐
        │   PostgreSQL   │ │ Payment Service│ │ Authentication │
        │    Database    │ │                │ │    & Users     │
        └────────────────┘ └───────┬────────┘ └────────────────┘
                                   │
                                   │ HTTPS
                                   ▼
                            ┌───────────────┐
                            │   Bitnob API  │
                            └───────┬───────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Bitcoin / Stablecoin│
                         │    Infrastructure   │
                         └─────────────────────┘
```

---

## 3. Mobile Application

The mobile application will be developed with Flutter.

The application will provide the user interface for:

* User registration and authentication
* Campaign creation
* Campaign browsing
* Campaign details
* Contribution initiation
* Payment status
* Campaign progress
* Organizer dashboard

The mobile application will communicate with the backend exclusively through the BokkPay REST API.

Sensitive credentials and payment secrets must never be stored in the mobile application.

---

## 4. Backend API

The backend will be developed using:

* Python
* Django
* Django REST Framework

The backend is responsible for:

* Authentication
* Authorization
* Campaign management
* Contribution management
* Payment management
* Database operations
* Bitnob API integration
* Webhook processing
* Business rules
* Security validation

The backend acts as the central layer between the mobile application, database, and payment infrastructure.

---

## 5. Database

PostgreSQL will be used as the primary database.

The main entities are expected to include:

```text
User
   │
   └── Campaign
          │
          ├── Contribution
          │       │
          │       └── Payment
          │
          └── Campaign metadata
```

The final database schema will be defined and implemented during the hackathon.

---

## 6. Payment Architecture

Payment operations must be handled by the backend.

The expected flow is:

```text
User
  │
  ▼
Flutter App
  │
  │ Create contribution
  ▼
Django API
  │
  │ Create payment request
  ▼
Payment Service
  │
  ▼
Bitnob API
  │
  ▼
Bitcoin / Stablecoin payment
```

The Flutter application must never directly expose Bitnob API credentials.

---

## 7. Payment Confirmation

Payment confirmation will rely on server-side verification and webhook processing.

Expected flow:

```text
                    Bitnob
                      │
                      │ Payment event
                      ▼
             Django Webhook Endpoint
                      │
                      ▼
             Validate the event
                      │
                      ▼
             Verify transaction
                      │
                      ▼
              Update Payment
                      │
                      ▼
            Update Contribution
                      │
                      ▼
             Update Campaign
                      │
                      ▼
              Notify the client
```

This approach allows BokkPay to maintain a reliable payment status instead of trusting information received directly from the mobile client.

---

## 8. Security Principles

Security is a core requirement of the platform.

### API credentials

Bitnob credentials must be stored on the backend and loaded through environment variables.

```text
BITNOB_API_KEY=...
BITNOB_SECRET=...
```

These values must never be committed to GitHub.

### Client security

The Flutter application must not contain private payment credentials.

### Server validation

Payment status and transaction information must be validated server-side.

### Webhooks

Webhook requests must be validated before modifying payment records.

### Environment variables

Sensitive configuration must be stored in `.env` files locally and excluded from version control.

---

## 9. API Communication

The expected communication model is:

```text
Flutter
   │
   │ HTTPS
   ▼
Django REST API
   │
   │ JSON
   ▼
Application services
   │
   ├── Database
   │
   └── Bitnob API
```

The API will use JSON for request and response payloads.

---

## 10. Planned Backend Modules

The backend is expected to contain modules similar to:

```text
backend/
│
├── accounts/
├── campaigns/
├── contributions/
├── payments/
└── bitnob/
```

Responsibilities:

### accounts

User registration, authentication, profiles and permissions.

### campaigns

Campaign creation, management, publication and progress.

### contributions

Contribution records and contributor information.

### payments

Payment lifecycle and transaction status.

### bitnob

Integration layer responsible for communication with the Bitnob API.

The exact structure may evolve during development.

---

## 11. Development Principle

BokkPay will follow a modular architecture so that payment infrastructure can be changed or extended without rewriting the entire application.

The payment integration will therefore be isolated behind a dedicated service layer.

This makes the system easier to:

* Test
* Maintain
* Extend
* Debug
* Scale

---

## 12. Hackathon Scope

The final implementation of BokkPay will be developed during **Code & Chain Dakar 2026**.

This document describes the planned architecture and does not represent a completed implementation.

The architecture may be adjusted during the hackathon based on:

* Bitnob API capabilities
* Technical constraints
* Team decisions
* Time available
* User experience requirements
