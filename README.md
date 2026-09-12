# 🏆 BokkPay

> **L'argent du groupe, simplement.**

BokkPay is a community fundraising platform designed to make collective contributions simpler, more transparent, and easier to track.

## 🎯 The Problem

Across Africa, people regularly organize collective fundraising for:

* 💍 Weddings
* 🎓 Education
* ❤️ Family support
* 🏥 Medical assistance
* 🎉 Events
* 🤝 Community projects
* 👥 Associations and group activities

However, contributions are often managed manually through messaging apps, spreadsheets, notebooks, or multiple payment channels.

This makes it difficult to know:

* Who has contributed?
* How much has been collected?
* How much remains?
* Which payments have been confirmed?
* Where is the money coming from?

## 💡 Our Vision

BokkPay aims to provide a simple digital experience for creating, sharing, and tracking community fundraising campaigns.

A campaign organizer should be able to:

1. Create a fundraising campaign.
2. Set a fundraising goal.
3. Share the campaign using a link or QR code.
4. Receive contributions.
5. Track campaign progress.
6. Access contribution records.
7. Provide contributors with a digital receipt.

## ₿ Bitcoin & Payment Infrastructure

BokkPay is being designed for the **Code & Chain Dakar 2026 Hackathon**.

The project will explore Bitcoin and modern payment infrastructure through the **Bitnob API**.

The final implementation will be developed during the hackathon in accordance with the event rules.

## 🚀 Planned Features

* 🔐 User authentication
* 💰 Fundraising campaign creation
* 🎯 Fundraising goals
* 🔗 Campaign sharing
* 📱 QR code contribution
* ₿ Bitcoin payment integration
* 💵 Stablecoin payment support where available
* 🔔 Transaction status tracking
* 🧾 Digital contribution receipts
* 📊 Campaign dashboard
* 📈 Fundraising progress tracking

## 🏗️ Planned Architecture

```text
                    BOKKPAY
                       │
              ┌────────┴────────┐
              │                 │
           Flutter           Django
           Mobile              API
              │                 │
              │          Django REST Framework
              │                 │
              │          ┌──────┴──────┐
              │          │             │
              │       PostgreSQL    Bitnob API
              │                        │
              └────────────────────────┤
                                       │
                              Bitcoin / Stablecoins
```

## 🛠️ Planned Tech Stack

| Technology            | Purpose                |
| --------------------- | ---------------------- |
| Flutter               | Mobile application     |
| Dart                  | Mobile development     |
| Python                | Backend development    |
| Django                | Backend framework      |
| Django REST Framework | REST API               |
| PostgreSQL            | Database               |
| Bitnob API            | Payment infrastructure |
| Git & GitHub          | Version control        |

## 🔐 Security Principles

Security will be a priority throughout development.

* API credentials will never be exposed in the frontend.
* Secrets will be stored in environment variables.
* `.env` files will not be committed.
* Payment status will be validated server-side.
* Bitnob webhook authenticity will be verified.
* Sensitive transaction information will be handled carefully.

## 📁 Planned Project Structure

```text
bokkpay/
│
├── backend/
│
├── mobile/
│
├── docs/
│
├── .gitignore
├── LICENSE
└── README.md
```

## 🎯 Hackathon

**Code & Chain Dakar 2026**

📍 Dakar, Senegal
📅 October 17–18, 2026

Built for the next generation of African Bitcoin developers.

## 👥 Team

Team members will be added during the hackathon.

## 📜 License

MIT License
