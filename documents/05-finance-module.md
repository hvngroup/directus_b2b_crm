# HVN Enterprise Platform - Finance & Billing Module

## 1. Module Overview

### 1.1 Purpose

The Finance & Billing module manages financial operations including contracts, invoicing, payments, expenses, and cash flow tracking.

### 1.2 Key Capabilities

- Contract lifecycle management
- Invoice tracking (consolidated from WHMCS)
- Payment recording and reconciliation
- Expense management and approval
- Cash flow tracking
- AR aging and collections
- Financial reporting

### 1.3 Important Note: WHMCS as Billing Source

**WHMCS remains the source of truth for billing.** Directus consolidates data from multiple WHMCS instances for unified reporting and AR management.

---

## 2. Architecture

### 2.1 Revenue Flow

```
CONTRACT CREATED ──▶ INVOICE GENERATED ──▶ PAYMENT RECEIVED ──▶ REVENUE RECOGNIZED
       │                    │                     │                    │
       ▼                    ▼                     ▼                    ▼
  [Directus]           [WHMCS]              [Bank/WHMCS]          [Directus]
  Contract record      Invoice created      Payment recorded      Cash Flow IN
                       Sync to Directus     Sync to Directus      KPI Updated
```

### 2.2 WHMCS Multi-Brand Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MULTI-BRAND WHMCS SETUP                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌───────────────┐   ┌───────────────┐   ┌───────────────┐        │
│  │   WHMCS #1    │   │   WHMCS #2    │   │   WHMCS #N    │        │
│  │   (hvn.vn)    │   │  (brand-b)    │   │  (brand-n)    │        │
│  └───────┬───────┘   └───────┬───────┘   └───────┬───────┘        │
│          │                   │                   │                 │
│          └───────────────────┼───────────────────┘                 │
│                              │                                     │
│                              ▼                                     │
│              ┌───────────────────────────────┐                     │
│              │        DIRECTUS FINANCE       │                     │
│              │      (Consolidated View)      │                     │
│              │                               │                     │
│              │  • Unified AR Aging           │                     │
│              │  • Cross-brand reports        │                     │
│              │  • Consolidated cash flow     │                     │
│              └───────────────────────────────┘                     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3. Data Model

### 3.1 Collections Overview

| Collection | Purpose | Source |
|------------|---------|--------|
| `contracts` | Service contracts | Directus (manual/from Opp) |
| `invoices` | Invoice records | Synced from WHMCS |
| `payments` | Payment records | Synced from WHMCS + Manual |
| `expenses` | Company expenses | Directus (manual) |
| `cash_flow` | Cash movement | Auto-generated |
| `whmcs_instances` | WHMCS config | Directus (config) |

### 3.2 Contracts Collection

```javascript
// Collection: contracts
{
  id: "uuid",
  status: "published",
  
  // Contract Info
  contract_code: "string",       // Auto: HVN-YYYY-XXXX
  contract_name: "string",
  
  // Parties
  account: "m2o:accounts",
  
  // Source
  opportunity: "m2o:opportunities",
  
  // Value
  contract_value: "decimal",     // Total value (VND)
  currency: "string",            // VND
  
  // Timeline
  start_date: "date",
  end_date: "date",
  duration_months: "formula",    // Calculated
  
  // Status
  contract_status: "single_select",  // Draft, Pending Approval, Active, Completed, Terminated
  
  // Payment Terms
  payment_schedule: "single_select", // 100% Upfront, 50-50, 30-40-30, Monthly, Quarterly
  payment_terms_days: "integer",     // Net 30, Net 60
  
  // Products/Services
  items: "o2m:contract_items",
  
  // Documents
  contract_file: "file",
  signed_date: "date",
  
  // Renewal
  auto_renew: "boolean",
  renewal_reminder_date: "date",     // end_date - 60 days
  
  // Owner
  account_manager: "m2o:employees",
  
  // Related
  invoices: "o2m:invoices",
  
  // System
  user_created: "user",
  date_created: "datetime"
}
```

### 3.3 Invoices Collection

```javascript
// Collection: invoices
{
  id: "uuid",
  status: "published",
  
  // Invoice Info
  invoice_number: "string",      // From WHMCS or manual
  
  // Source
  source_system: "single_select",  // WHMCS, Manual
  whmcs_instance: "m2o:whmcs_instances",
  whmcs_invoice_id: "string",
  
  // Customer
  account: "m2o:accounts",
  contract: "m2o:contracts",
  
  // Amounts
  subtotal: "decimal",
  tax_rate: "decimal",           // 10% VAT
  tax_amount: "decimal",
  total_amount: "decimal",
  
  // Payment
  paid_amount: "decimal",        // Rollup from payments
  remaining_amount: "formula",   // total - paid
  
  // Dates
  invoice_date: "date",
  due_date: "date",
  
  // Status
  invoice_status: "single_select",  // Draft, Sent, Partial, Paid, Overdue, Cancelled
  
  // Aging
  days_overdue: "formula",       // IF(due_date < today AND status != Paid)
  aging_bucket: "formula",       // Current, 1-30, 31-60, 61-90, 90+
  
  // Payments
  payments: "o2m:payments",
  
  // System
  synced_at: "datetime",
  user_created: "user",
  date_created: "datetime"
}
```

### 3.4 Payments Collection

```javascript
// Collection: payments
{
  id: "uuid",
  status: "published",
  
  // Payment Info
  payment_date: "date",
  amount: "decimal",
  
  // Source
  source_system: "single_select",  // WHMCS, Bank, Manual
  whmcs_instance: "m2o:whmcs_instances",
  whmcs_transaction_id: "string",
  
  // Link
  invoice: "m2o:invoices",
  account: "m2o:accounts",
  
  // Method
  payment_method: "single_select",  // Bank Transfer, Cash, Credit Card, E-Wallet
  
  // Reference
  reference_number: "string",    // Bank reference
  bank_account: "m2o:bank_accounts",
  
  // Verification
  verified: "boolean",
  verified_by: "m2o:employees",
  verified_date: "datetime",
  
  // Attachments
  receipt: "file",
  
  // System
  synced_at: "datetime",
  user_created: "user",
  date_created: "datetime"
}
```

### 3.5 Expenses Collection

```javascript
// Collection: expenses
{
  id: "uuid",
  status: "published",
  
  // Expense Info
  expense_date: "date",
  amount: "decimal",
  description: "string",
  
  // Classification
  category: "m2o:expense_categories",
  
  // Link
  contract: "m2o:contracts",     // If related to contract
  account: "m2o:accounts",       // If customer-related
  
  // Vendor
  vendor_name: "string",
  
  // Approval
  requested_by: "m2o:employees",
  approval_status: "single_select",  // Pending, Approved, Rejected
  approved_by: "m2o:employees",
  approved_date: "datetime",
  rejection_reason: "text",
  
  // Payment
  payment_status: "single_select",  // Unpaid, Paid
  paid_date: "date",
  
  // Attachments
  receipts: "files",
  
  // System
  user_created: "user",
  date_created: "datetime"
}
```

### 3.6 Cash Flow Collection

```javascript
// Collection: cash_flow
{
  id: "uuid",
  status: "published",
  
  // Transaction Info
  transaction_date: "date",
  type: "single_select",         // IN, OUT
  amount: "decimal",
  
  // Classification
  category: "single_select",     // See categories below
  
  // Reference
  reference_type: "single_select",  // Payment, Expense, Transfer, Other
  reference_id: "string",
  
  // Links
  payment: "m2o:payments",       // If type = IN
  expense: "m2o:expenses",       // If type = OUT
  account: "m2o:accounts",       // Customer (if applicable)
  
  // Bank
  bank_account: "m2o:bank_accounts",
  
  // Description
  description: "string",
  
  // Verification
  verified: "boolean",
  
  // System
  user_created: "user",
  date_created: "datetime"
}
```

### 3.7 Cash Flow Categories

| Type | Categories |
|------|------------|
| **IN** | Customer Payment, Deposit Received, Refund Received, Investment, Loan, Other Income |
| **OUT** | Salary, Commission, Marketing, Operations, Vendor Payment, Tax, Loan Repayment, Expense Reimbursement, Other Expense |

---

## 4. WHMCS Integration

### 4.1 Sync Configuration

```javascript
// Collection: whmcs_instances
{
  id: "uuid",
  
  // Instance Info
  instance_name: "string",       // "HVN Main", "Brand B"
  brand: "string",
  
  // API Config
  api_url: "string",             // https://whmcs.hvn.vn/includes/api.php
  api_identifier: "string",
  api_secret: "string",          // Encrypted
  
  // Webhook
  webhook_secret: "string",
  
  // Sync Settings
  sync_enabled: "boolean",
  last_sync: "datetime",
  sync_frequency: "integer",     // Minutes
  
  // Status
  connection_status: "single_select",  // Connected, Error, Disabled
  last_error: "text"
}
```

### 4.2 WHMCS → Directus Sync Events

| WHMCS Event | Directus Action |
|-------------|-----------------|
| InvoiceCreation | Create invoice record |
| InvoicePaid | Update invoice status, create payment |
| InvoiceRefunded | Create negative payment, update invoice |
| InvoiceCancelled | Update invoice status = Cancelled |
| AddInvoicePayment | Create payment record |

### 4.3 Webhook Handler Flow

```
WHMCS Event ──▶ Webhook ──▶ Directus Flow ──▶ Process
     │                           │
     │                           ├── Validate webhook secret
     │                           ├── Identify WHMCS instance
     │                           ├── Map WHMCS client → Account
     │                           ├── Create/Update records
     │                           └── Log sync
     │
     └── [If error] ──▶ Log error, retry queue
```

### 4.4 Client Mapping Logic

```
WHMCS Client                           Directus Account
     │                                       │
     │   Step 1: Check whmcs_client_id       │
     │   ─────────────────────────────       │
     │   IF accounts.whmcs_client_id exists  │
     │   THEN link to existing account       │
     │                                       │
     │   Step 2: Check email/company         │
     │   ─────────────────────────────       │
     │   IF no whmcs_client_id match         │
     │   THEN match by email OR company_name │
     │                                       │
     │   Step 3: Create new                  │
     │   ─────────────────────────────       │
     │   IF no match found                   │
     │   THEN create new account             │
     └───────────────────────────────────────┘
```

---

## 5. Approval Workflows

### 5.1 Expense Approval

| Amount Range | Approver(s) | SLA |
|--------------|-------------|-----|
| ≤ 5,000,000 VND | Line Manager | 4 hours |
| 5M - 50M VND | Line Manager → Finance Manager | 8 hours |
| > 50M VND | Line Manager → Finance Manager → CFO | 24 hours |

### 5.2 Payment Request Approval

| Amount Range | Approver(s) |
|--------------|-------------|
| ≤ 10,000,000 VND | Finance Staff (verify only) |
| 10M - 100M VND | Finance Manager |
| > 100M VND | Finance Manager → CFO |

### 5.3 Credit Extension Approval

**Trigger:** When customer's total AR exceeds credit_limit

| Approval | Condition | Approvers |
|----------|-----------|-----------|
| Credit Extension | AR > credit_limit | Account Manager → Finance Manager → Director |

---

## 6. Automation Rules

### 6.1 Invoice Automation

| Rule | Trigger | Condition | Action |
|------|---------|-----------|--------|
| INV-001 | Invoice synced | New invoice | Link to account, set due date |
| INV-002 | Scheduled (daily) | due_date = today, status != Paid | Status = Overdue, notify AM |
| INV-003 | Scheduled (daily) | days_overdue > 30 | Escalate to manager |
| INV-004 | Scheduled (daily) | days_overdue > 60 | Escalate to director |
| INV-005 | Payment created | Linked to invoice | Update paid_amount, check if fully paid |

### 6.2 Cash Flow Automation

| Rule | Trigger | Condition | Action |
|------|---------|-----------|--------|
| CF-001 | Payment created | verified = true | Create Cash Flow IN |
| CF-002 | Expense paid | payment_status = Paid | Create Cash Flow OUT |
| CF-003 | Scheduled (daily 6PM) | - | Generate daily cash summary |

### 6.3 Contract Automation

| Rule | Trigger | Condition | Action |
|------|---------|-----------|--------|
| CON-001 | Contract created | Has payment_schedule | Generate invoice schedule |
| CON-002 | Scheduled (daily) | renewal_reminder_date = today | Notify AM for renewal |
| CON-003 | Scheduled (daily) | end_date passed, status = Active | Status = Completed |

---

## 7. AR Aging Report

### 7.1 Aging Buckets

| Bucket | Definition |
|--------|------------|
| Current | Not yet due |
| 1-30 Days | 1-30 days past due |
| 31-60 Days | 31-60 days past due |
| 61-90 Days | 61-90 days past due |
| Over 90 Days | More than 90 days past due |

### 7.2 AR Aging Dashboard

```
┌─────────────────────────────────────────────────────────────────────┐
│                        AR AGING DASHBOARD                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  SUMMARY CARDS                                                      │
│  ┌──────────┬──────────┬──────────┬──────────┬──────────┐         │
│  │  TOTAL   │ CURRENT  │  1-30    │  31-60   │   90+    │         │
│  │   AR     │          │          │          │          │         │
│  │  500M    │   200M   │   150M   │   100M   │   50M    │         │
│  └──────────┴──────────┴──────────┴──────────┴──────────┘         │
│                                                                     │
│  AGING CHART (Stacked Bar by Account Manager)                       │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                                                             │   │
│  │  AM 1: ████████ ██████ ████ ██                             │   │
│  │  AM 2: ███████████ ███ █                                   │   │
│  │  AM 3: ████████████████ ██████                             │   │
│  │                                                             │   │
│  │  Legend: Current | 1-30 | 31-60 | 61-90 | 90+              │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  TOP OVERDUE ACCOUNTS (Table)                                       │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ Account        │ Total AR  │ Overdue │ Days │ AM          │   │
│  │ Company ABC    │ 100M      │ 80M     │ 45   │ Nguyen A    │   │
│  │ Company XYZ    │ 75M       │ 60M     │ 32   │ Tran B      │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 8. Google Chat Notifications

| Event | Channel | Message |
|-------|---------|---------|
| Invoice overdue | Direct to AM | "⚠️ Invoice {number} is overdue - {customer}" |
| 30 days overdue | #finance-alerts + Manager | "🔴 Invoice {number} 30+ days overdue" |
| Payment received | #finance-alerts | "💰 Payment received: {amount} from {customer}" |
| Large payment | Direct to Finance Manager | "💰 Large payment: {amount} - needs verification" |
| Contract expiring | Direct to AM | "📋 Contract {code} expires in 60 days" |

---

## 9. Reports

### 9.1 Standard Reports

| Report | Frequency | Content |
|--------|-----------|---------|
| Daily Cash Position | Daily | Cash balance, movements |
| AR Aging | Weekly | Aging by bucket, by AM |
| Revenue Report | Monthly | Revenue by product, by customer |
| Collections Report | Weekly | Collection rate, overdue tracking |
| Expense Report | Monthly | Expenses by category |

### 9.2 KPIs

| KPI | Formula | Target |
|-----|---------|--------|
| DSO | (AR / Revenue) × Days | < 45 days |
| Collection Rate | Collected / Due | > 95% |
| Overdue Ratio | Overdue / Total AR | < 10% |
| Cash Runway | Cash / Monthly Burn | > 6 months |

---

*Document Version: 1.0*
*Last Updated: January 2026*
*Author: HVN GROUP*