# DATABASE_DICTIONARY.md

# iGmali Database Dictionary

Project:
iGmali

Current Database Schema Version:
1

Document Version:
1.0

Last Updated:
2026-06

License

This document is provided to help developers understand the public database structure used by iGmali.

The document describes the database schema and business meaning of stored values.

It does not describe the internal implementation of the application.

---

## Overview

Database Engine

* SQLite

Schema Version

* Stored in `AppMeta`
* Current Version: **1**

Storage

* Local device only
* Offline-first architecture

Database Mode

* WAL (Write-Ahead Logging)

Purpose

The database stores all business data, application metadata, licensing information, and supporting lookup tables.

---

# Table Categories

The database consists of four logical groups.

## Business Tables

* Deals
* Transactions
* DealPricing
* DealPricingLines
* ServicePeriodTransactions
* SettlementTransactions

---

## Lookup Tables

* Parties
* Items
* Banks

---

## System Tables

* AppMeta

---

## Licensing Tables

* LicenseCustomers
* LicenseActivations

---

# Entity Relationship Overview

The following diagram illustrates the logical relationships between the main database tables.

```text
                               +----------------+
                               |    AppMeta     |
                               +----------------+
                                        |
                                        |
                                Application Settings
                                        |
                                        |
────────────────────────────────────────────────────────────────────────────

                          +----------------------+
                          |        Deals         |
                          +----------------------+
                          | Id                   |
                          | DealType             |
                          | Title                |
                          +----------+-----------+
                                     |
                                     |
                    1                |               N
                                     |
                                     v

                         +----------------------+
                         |    Transactions      |
                         +----------------------+
                         | Id                   |
                         | DealId              |
                         | PartyId             |
                         | ItemId              |
                         | BankId              |
                         | Type                |
                         | Direction           |
                         | Status              |
                         | Amount              |
                         +--+--------+-----+----+
                            |        |     |
                            |        |     |
                            |        |     |
                            |        |     |
                            |        |     |
                            |        |     |
                            |        |     |
                            |        |     |
                            |        |     |
                            |        |     |
                            |        |     |
                            |        |     |
                            |        |     |
                            |        |     |
                            |        |     |
                            |        |     |
                            |        |     |
                            |        |     |
                            |        |     |
                            |        |     |
                            |        |     |
                            v        v     v

                  +-----------+  +---------+  +--------+
                  | Parties   |  | Items   |  | Banks  |
                  +-----------+  +---------+  +--------+

────────────────────────────────────────────────────────────────────────────

          Deals
             |
             | 1
             |
             | N
             v

      +----------------------+
      |     DealPricing      |
      +----------------------+
                  |
                  | 1
                  |
                  | N
                  v

      +----------------------+
      |   DealPricingLines   |
      +----------------------+
                  |
                  |
                  v
               Items

────────────────────────────────────────────────────────────────────────────

      Transactions
             |
             | 1
             |
             | N
             v

 +--------------------------------+
 | ServicePeriodTransactions      |
 +--------------------------------+

Represents the logical relationship between:

    Service Start Transaction
                │
                │
                ▼
      Service End Transaction

────────────────────────────────────────────────────────────────────────────

      Settlement Deal
             |
             |
             v

 +--------------------------------+
 |   SettlementTransactions       |
 +--------------------------------+
             |
             |
             v

     Original Transactions

A settlement deal groups one or more existing transactions.

────────────────────────────────────────────────────────────────────────────

                Licensing Subsystem

 +----------------------+      +----------------------+
 | LicenseCustomers     |      | LicenseActivations   |
 +----------------------+      +----------------------+

These tables are independent from business data.

They belong exclusively to the licensing subsystem.

────────────────────────────────────────────────────────────────────────────

                     Attachments

 +----------------------+
 |     Attachments      |
 +----------------------+

Stores files attached to transactions.

(Currently reserved for future expansion.)
```

## Notes

* Most relationships are maintained by the application layer rather than SQLite foreign keys.
* `Transactions` is the central business table.
* `Deals` act as business containers that group related transactions.
* `Parties`, `Items`, and `Banks` are lookup tables.
* `ServicePeriodTransactions` and `SettlementTransactions` represent logical relationships rather than standalone business entities.
* `LicenseCustomers`, `LicenseActivations`, and `AppMeta` are system-level tables and are independent of normal business operations.


---

# Tables

---

# Parties

Purpose

Stores customers, suppliers and any external party involved in transactions.

Primary Key

```text
Id
```

Columns

| Column   | Type    | Description           |
| -------- | ------- | --------------------- |
| Id       | Integer | Primary key           |
| Name     | Text    | Party name            |
| Phone    | Text    | Optional phone number |
| Notes    | Text    | Optional notes        |
| IsActive | Boolean | Active flag           |

Referenced By

* Transactions

---

# Items

Purpose

Stores products, services and rental/service definitions.

Primary Key

```text
Id
```

Columns

| Column           | Description            |
| ---------------- | ---------------------- |
| Id               | Primary key            |
| Name             | Display name           |
| Type             | Item type              |
| DefaultUnitPrice | Optional default price |
| IsActive         | Active flag            |

Referenced By

* Transactions
* DealPricingLines

---

# Banks

Purpose

Stores banks used only for cheque transactions.

Primary Key

```text
Id
```

Referenced By

* Transactions

Developer Note

Banks are supporting data only.

They are intentionally hidden from the application's main menu.

---

# Deals

Purpose

Represents one business agreement.

A deal groups one or more transactions.

Primary Key

```text
Id
```

Important Fields

| Field    | Description           |
| -------- | --------------------- |
| DealType | General or Settlement |
| Title    | User title            |
| Notes    | General notes         |

Referenced By

* Transactions
* DealPricing
* SettlementTransactions

---

# Transactions

Purpose

Main business table.

Every movement inside the application is represented as a transaction.

Primary Key

```text
Id
```

Logical References

| Field   | References |
| ------- | ---------- |
| DealId  | Deals      |
| PartyId | Parties    |
| ItemId  | Items      |
| BankId  | Banks      |

Important Fields

| Field       | Description         |
| ----------- | ------------------- |
| Type        | Transaction type    |
| Direction   | Incoming / Outgoing |
| Status      | Current state       |
| CloseReason | Reason for closing  |
| Amount      | Current amount      |
| RootAmount  | Original amount     |
| RecordDate  | Creation date       |
| DueDate     | Due date            |
| StatusDate  | Operational date    |
| Notes       | Notes               |

Developer Notes

Transactions should never be interpreted using Amount alone.

Business logic also depends on:

* Status
* Direction
* Type
* CloseReason

---

# DealPricing

Purpose

Stores pricing templates attached to a Deal.

Referenced By

* DealPricingLines

---

# DealPricingLines

Purpose

Stores detailed pricing rows.

Logical References

* DealPricing
* Items

---

# ServicePeriodTransactions

Purpose

Links rental / period-service transactions together.

Purpose

Represents relationships between:

* Service Start
* Service End

Developer Notes

Applications should treat both transactions as one logical service period.

---

# SettlementTransactions

Purpose

Stores transactions participating in a settlement deal.

Fields

| Field            | Description         |
| ---------------- | ------------------- |
| SettlementDealId | Settlement deal     |
| TransactionId    | Related transaction |

---

# Attachments

Purpose

Stores files attached to transactions.

Current Usage

Reserved for future expansion.

---

# LicenseCustomers

Purpose

Stores licensing-related customer records.

Used internally by the licensing subsystem.

Not related to business Parties.

---

# LicenseActivations

Purpose

Stores activation records used by the licensing subsystem.

Used by the licensing subsystem only.

---

# AppMeta

Purpose

Stores application metadata.

Examples

* Database version
* Update policy information
* Internal settings

Developer Note

This table replaces a traditional Settings table.

Values are stored as key/value pairs.

---

# Enumerations

## DealType

| Value | Meaning    |
| ----: | ---------- |
|     1 | Settlement |
|     2 | General    |

---

## TransactionType

Represents the nature of a transaction.

Examples include:

* Money
* Cheque
* Product
* Service
* Service Start
* Service End

---

## TransactionDirection

| Value | Meaning  |
| ----: | -------- |
|     1 | Incoming |
|     2 | Outgoing |

---

## TransactionStatus

Represents the current operational state of a transaction.

Typical states include:

* Pending
* Completed
* Returned Cheque
* Rescheduled
* Written Off
* Settled

---

## CloseReason

Explains why a transaction became closed.

Examples include:

* Completed
* Returned
* Written Off
* Settled

---

## ItemType

Defines the business meaning of an Item.

Examples include:

* Product
* Service
* Rental / Period Service

---

# Important Field Semantics

## RecordDate

Represents when the transaction was originally recorded.

It never changes after creation.

---

## DueDate

Represents the expected execution or obligation date.

Pending transactions use this date to indicate when work is expected.

---

## StatusDate

The operational date used by the application.

Rules

Pending

```text
StatusDate = DueDate
```

Completed

```text
StatusDate = Completion Date
```

Rescheduled

```text
StatusDate = New DueDate
```

Developer Note

Home screen buckets are based primarily on StatusDate.

Do not replace StatusDate logic with DueDate without understanding the application workflow.

---

## Amount

Current effective value of the transaction.

This value may change after partial settlement.

---

## RootAmount

Stores the original transaction amount.

Used to preserve historical context after operations such as:

* Partial payment
* Partial fulfillment
* Settlement

---

## Rental / Period Services

Rental services are represented by linked transactions.

The relationship is maintained through ServicePeriodTransactions.

Developers should avoid editing one side independently without updating the related transaction.

---

## Settlement Deals

Settlement transactions are grouped under a dedicated Deal.

The SettlementTransactions table identifies which original transactions participate in the settlement.

---

# Logical Relationships

```text
Party
   │
   └──────────────┐
                  │
                Transaction
                  │
                  │
Deal ─────────────┘

Deal
   │
   └── DealPricing
          │
          └── DealPricingLines
                 │
                 └── Item

Transaction
      │
      └── ServicePeriodTransactions

Settlement Deal
      │
      └── SettlementTransactions
```

---

# Database Design Notes

The database intentionally avoids enforcing most foreign keys at the SQLite level.

Relationships are maintained by the application layer.

Reasons include:

* simpler imports,
* easier backup/restore,
* flexibility during future schema evolution.

---

# Developer Guidelines

When reading the database:

* Prefer logical relationships over physical foreign keys.
* Use enums instead of hardcoded numeric values.
* Never assume StatusDate equals RecordDate.
* RootAmount preserves the original business context.
* AppMeta stores system configuration, not business data.
* Preserve compatibility with future schema versions.

---

# Compatibility

Current Schema Version

```text
1
```

Future schema changes should include migration logic while preserving user data whenever possible.
