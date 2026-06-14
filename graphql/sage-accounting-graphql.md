# Sage Accounting GraphQL Schema

## Overview

This GraphQL schema represents the Sage Accounting (formerly Sage Business Cloud Accounting / Sage One) REST API surface as a typed GraphQL interface. Sage Accounting is Sage's cloud-based accounting platform for UK and international small and medium businesses, covering invoicing, expenses, banking, VAT, payroll integration, and financial reporting.

The underlying REST API is available at `https://api.accounting.sage.com/v3.1` and uses OAuth 2.0 for authentication. This schema provides a conceptual GraphQL layer over that API, making the domain model explicit and queryable in a unified graph.

**Developer Reference:** https://developer.sage.com/accounting/reference/
**GitHub Organization:** https://github.com/Sage

---

## Schema Source

- **Provider:** Sage Accounting
- **API Version:** v3.1
- **Base URL:** https://api.accounting.sage.com/v3.1
- **Auth:** OAuth 2.0 (authorization code flow); access tokens expire in 5 minutes, refresh tokens in 31 days
- **Multi-Business:** Target a specific business using the `X-Business` request header

---

## Domain Coverage

The schema covers the following major functional areas of Sage Accounting:

### Contacts and Parties
Customers and suppliers are represented as `Contact` objects with nested `ContactPerson`, `ContactEmail`, `ContactPhone`, and `ContactAddress` types. Contacts can be marked as customers, suppliers, or both, and carry default tax treatment, payment terms, and credit limits.

### Sales Transactions
Sales invoices (`SalesInvoice`), credit notes (`CreditNote`), quotes (`Quote`), estimates (`Estimate`), and sales receipts (`SalesReceipt`) cover the full sales cycle. Each invoice carries one or more `SalesInvoiceLine` items linked to products, services, and tax rates.

### Purchase Transactions
Purchase invoices (`PurchaseInvoice`) and purchase credit notes (`PurchaseCreditNote`) with `PurchaseLine` items represent the AP side of the ledger.

### Banking
`BankAccount`, `BankTransaction`, `BankReceipt`, `BankPayment`, `BankTransfer`, and `BankReconciliation` cover the full banking workflow including statement imports and reconciliation.

### Payments and Refunds
`Payment`, `BankPayment`, and `Refund` types link transactions to bank accounts and contacts. `PaymentMethod` records the method used (BACS, card, cheque, etc.).

### Ledger and Journals
`Account` (chart of accounts / ledger accounts), `JournalEntry`, and `JournalLine` allow direct bookkeeping and GL integration.

### Tax
`TaxRate` and `TaxProfile` support the UK VAT schemes (Standard, Cash, Flat Rate) and HMRC Making Tax Digital (MTD) VAT submissions.

### Products and Services
`Product`, `Service`, `ProductGroup`, and `ServiceGroup` represent the catalogue items used on invoice lines. `StockItem` and `StockMovement` track inventory where applicable.

### Business Configuration
`BusinessSettings`, `FinancialYear`, `OpeningBalance`, `Currency`, `Country`, `Language`, `AddressRegion`, `BusinessActivity`, `BusinessType`, and `SalesChannel` capture the global business configuration.

### Exchange Rates
`BusinessExchangeRate` records the exchange rates used for multi-currency transactions.

### Platform and Security
`User`, `APIKey`, and `Webhook` cover user management, API credentials, and event subscriptions.

---

## Type Index

| Type | Description |
|------|-------------|
| `Contact` | Customer or supplier record |
| `ContactPerson` | Named individual at a contact |
| `ContactEmail` | Email address for a contact or person |
| `ContactPhone` | Phone number for a contact or person |
| `ContactAddress` | Postal address for a contact |
| `Bank` | Bank institution definition |
| `BankAccount` | Business bank account linked to a bank |
| `BankTransaction` | Individual bank statement line |
| `BankReconciliation` | Reconciliation session for a bank account |
| `BusinessExchangeRate` | Exchange rate between two currencies |
| `SalesInvoice` | Sales invoice raised against a contact |
| `SalesInvoiceLine` | Line item on a sales invoice |
| `PurchaseInvoice` | Supplier invoice received |
| `PurchaseLine` | Line item on a purchase invoice |
| `CreditNote` | Sales credit note issued to a customer |
| `PurchaseCreditNote` | Purchase credit note received from supplier |
| `Quote` | Sales quote sent to a prospect or customer |
| `Estimate` | Sales estimate (US/international markets) |
| `SalesReceipt` | Receipt for a direct cash sale |
| `Refund` | Refund issued against a credit note or overpayment |
| `PaymentMethod` | Configured payment method (BACS, card, etc.) |
| `Payment` | Payment allocation against an invoice |
| `BankPayment` | Payment recorded against a bank account |
| `BankReceipt` | Receipt recorded against a bank account |
| `BankTransfer` | Transfer between two bank accounts |
| `JournalEntry` | Manual journal entry posted to the GL |
| `JournalLine` | Individual line in a journal entry |
| `Account` | Ledger account in the chart of accounts |
| `Category` | Category grouping for ledger accounts |
| `TaxRate` | Tax rate (e.g. Standard 20%, Zero Rated) |
| `TaxProfile` | Tax scheme configuration for the business |
| `Product` | Physical product in the item catalogue |
| `Service` | Service item in the item catalogue |
| `ProductGroup` | Group or category for products |
| `ServiceGroup` | Group or category for services |
| `StockItem` | Inventory stock record for a product |
| `StockMovement` | Stock in/out movement against a stock item |
| `Ledger` | Named ledger (e.g. Sales, Purchases, Bank) |
| `FinancialYear` | Financial year configuration |
| `OpeningBalance` | Opening balances for accounts at period start |
| `User` | User with access to the business |
| `BusinessSettings` | Global settings for the business |
| `AddressRegion` | Region/county/state within a country |
| `Country` | ISO country |
| `Currency` | ISO currency |
| `Language` | Supported language/locale |
| `BusinessActivity` | Business activity classification |
| `BusinessType` | Legal business type (sole trader, limited, etc.) |
| `SalesChannel` | Sales channel used on invoices |
| `Webhook` | Webhook subscription for events |
| `APIKey` | API key / OAuth app credential |
| `Attachment` | File attachment on a transaction or contact |
| `Address` | Reusable inline address value object |
| `Money` | Monetary amount with currency |
| `PageInfo` | Pagination metadata |
| `Connection` | Generic paginated list wrapper |

---

## Query Examples

```graphql
# Fetch a page of contacts
query ListContacts {
  contacts(first: 20, customerSupplier: CUSTOMER) {
    pageInfo { hasNextPage endCursor }
    edges {
      node {
        id
        name
        email
        taxTreatment
        balance
      }
    }
  }
}

# Fetch a single sales invoice with lines
query GetInvoice($id: ID!) {
  salesInvoice(id: $id) {
    id
    reference
    date
    dueDate
    status
    totalAmount { amount currency { symbol } }
    contact { id name }
    lines {
      description
      quantity
      unitPrice { amount }
      taxRate { percentage }
      netAmount { amount }
    }
  }
}

# List bank accounts
query BankAccounts {
  bankAccounts {
    id
    name
    accountType
    currency { isoCode }
    balance
    bankAccount { sortCode accountNumber }
  }
}
```
