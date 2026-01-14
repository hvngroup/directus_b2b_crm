# HVN Enterprise Platform - CRM B2B Module

## 1. Module Overview

### 1.1 Purpose

The CRM B2B module manages the complete sales lifecycle from lead capture to deal closure, providing a unified view of customers and sales activities.

### 1.2 Key Capabilities

- Lead management and scoring
- Account and contact management
- Sales pipeline (Kanban)
- Quotation generation
- Product catalog
- Sales analytics and dashboards

---

## 2. Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              CRM B2B MODULE ARCHITECTURE                             │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                      │
│  ┌─────────────────────────────────────────────────────────────────────────────┐   │
│  │                           SALES PIPELINE FLOW                                │   │
│  │                                                                              │   │
│  │  ┌─────────┐   ┌───────────┐   ┌─────────────┐   ┌──────────┐   ┌────────┐ │   │
│  │  │  LEAD   │──▶│ QUALIFIED │──▶│ OPPORTUNITY │──▶│ PROPOSAL │──▶│ CLOSED │ │   │
│  │  │ Capture │   │   Lead    │   │   Created   │   │   Sent   │   │  Deal  │ │   │
│  │  └─────────┘   └───────────┘   └─────────────┘   └──────────┘   └────────┘ │   │
│  │       │              │               │                │             │       │   │
│  │       ▼              ▼               ▼                ▼             ▼       │   │
│  │  [Auto-assign] [Scoring]     [Account Link]    [Quotation]    [Contract]   │   │
│  │  [Notify Sales][Nurture]     [Contact Link]    [Discount?]    [Invoice]    │   │
│  │                              [Products]         [Approval]     [Payment]   │   │
│  └─────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                      │
│  ┌────────────────────────────────────────────────────────────────────────────┐    │
│  │                    COMPONENT COLLABORATION                                  │    │
│  │                                                                             │    │
│  │  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐     │    │
│  │  │  BASE   │◀─▶│ BASE APP│◀─▶│APPROVAL │◀─▶│MESSENGER│◀─▶│ CALENDAR│     │    │
│  │  │ (Data)  │   │  (UI)   │   │(Discount)│  │(Alerts) │   │(Meetings)│    │    │
│  │  └────┬────┘   └─────────┘   └─────────┘   └─────────┘   └─────────┘     │    │
│  │       │                                                                    │    │
│  │       └────────▶ AUTOMATION (Rules & Triggers)                            │    │
│  │                                                                             │    │
│  └────────────────────────────────────────────────────────────────────────────┘    │
│                                                                                      │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Data Model

### 3.1 Collections Overview

| Collection | Purpose | Key Fields |
|------------|---------|------------|
| `leads` | Potential customers | name, company, source, status, score |
| `accounts` | Customer companies | name, industry, type, owner, credit_limit |
| `contacts` | People at companies | name, email, phone, account, role |
| `opportunities` | Sales deals | name, account, stage, value, close_date |
| `products` | Products/services | name, sku, price, category, active |
| `quotations` | Price proposals | opportunity, items, total, valid_until |

### 3.2 Leads Collection

```javascript
// Collection: leads
{
  id: "uuid",
  status: "published",
  
  // Basic Info
  lead_name: "string",           // Contact name
  company_name: "string",        // Company name
  phone: "string",
  email: "string",
  
  // Classification
  channel: "single_select",      // Website, Facebook, Google, Referral, etc.
  lead_status: "single_select",  // New, Contacted, Qualified, Unqualified, Converted
  score: "integer",              // 0-100
  
  // Assignment
  assigned_to: "m2o:employees",  // Sales rep
  team: "m2o:teams",
  
  // Source Tracking
  utm_source: "string",
  utm_medium: "string",
  utm_campaign: "string",
  referrer_url: "string",
  
  // Notes
  notes: "text",
  
  // System
  user_created: "user",
  date_created: "datetime",
  user_updated: "user",
  date_updated: "datetime",
  
  // Conversion (when converted to opportunity)
  converted_date: "date",
  converted_account: "m2o:accounts",
  converted_opportunity: "m2o:opportunities"
}
```

### 3.3 Accounts Collection

```javascript
// Collection: accounts
{
  id: "uuid",
  status: "published",
  
  // Basic Info
  company_name: "string",        // Required, Unique
  tax_code: "string",            // MST
  industry: "single_select",     // Industry classification
  company_size: "single_select", // Micro, SME, Enterprise
  
  // Contact Info
  address: "string",
  city: "single_select",
  website: "string",
  
  // Account Management
  account_type: "single_select", // Prospect, Customer, Partner
  account_owner: "m2o:employees",// Account Manager
  
  // Credit
  credit_limit: "decimal",       // VND
  payment_terms: "integer",      // Days (default: 30)
  
  // External Links
  whmcs_client_id: "string",     // Link to WHMCS
  whmcs_instance: "string",      // Which WHMCS instance
  
  // Relations
  contacts: "o2m:contacts",
  opportunities: "o2m:opportunities",
  contracts: "o2m:contracts",
  support_cases: "o2m:support_cases",
  
  // System
  user_created: "user",
  date_created: "datetime"
}
```

### 3.4 Opportunities Collection

```javascript
// Collection: opportunities
{
  id: "uuid",
  status: "published",
  
  // Basic Info
  opp_name: "string",            // Deal name
  account: "m2o:accounts",       // Customer
  contact: "m2o:contacts",       // Primary contact
  
  // Pipeline
  stage: "single_select",        // Qualification, Needs Analysis, Proposal, Negotiation, Won, Lost
  probability: "integer",        // 0-100%
  
  // Value
  amount: "decimal",             // Expected value (VND)
  weighted_amount: "formula",    // amount * probability
  
  // Timeline
  close_date: "date",            // Expected close
  won_date: "date",              // Actual close (if won)
  
  // Products
  products: "m2m:products",      // via opportunity_products
  
  // Discount
  discount_percent: "decimal",   // 0-100
  discount_approved: "boolean",  // Requires approval if > 15%
  discount_approved_by: "m2o:employees",
  discount_approved_date: "datetime",
  
  // Assignment
  owner: "m2o:employees",        // Sales rep
  
  // Lost Analysis
  lost_reason: "single_select",  // Price, Competition, No Budget, etc.
  competitor: "string",
  
  // Next Steps
  next_action: "string",
  next_action_date: "date",
  
  // Source
  lead: "m2o:leads",             // If converted from lead
  
  // System
  user_created: "user",
  date_created: "datetime"
}
```

### 3.5 Pipeline Stages

| Stage | Probability | Description |
|-------|-------------|-------------|
| Qualification | 10% | Initial contact, understanding needs |
| Needs Analysis | 25% | Deep dive into requirements |
| Proposal | 50% | Quotation sent |
| Negotiation | 75% | Discussing terms |
| Closed Won | 100% | Deal closed successfully |
| Closed Lost | 0% | Deal lost |

---

## 4. Automation Rules

### 4.1 Lead Automation

| Rule ID | Trigger | Condition | Action |
|---------|---------|-----------|--------|
| L-001 | Lead created | channel = "Website" | Auto-assign to Web Team round-robin |
| L-002 | Lead created | channel = "Referral" | Assign to referrer's AM |
| L-003 | Lead created | Any | Notify assigned sales via Google Chat |
| L-004 | Lead score updated | score >= 70 | Change status to "Qualified" |
| L-005 | Scheduled (daily) | No activity 7 days | Alert "Stale lead" to owner & manager |
| L-006 | Status = "Qualified" | Has account info | Create Account + Contact + Opportunity |

### 4.2 Lead Assignment Logic

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      LEAD ASSIGNMENT RULES                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  IF channel = "Website" THEN                                            │
│    → Round-robin assign to Sales Team A                                 │
│                                                                         │
│  ELSE IF channel = "Referral" THEN                                      │
│    → Assign to Account Manager of referring customer                    │
│                                                                         │
│  ELSE IF channel = "Partner" THEN                                       │
│    → Assign to Partner Manager                                          │
│                                                                         │
│  ELSE IF channel IN ("Facebook", "Google", "Zalo") THEN                 │
│    → Round-robin assign to Digital Sales Team                           │
│                                                                         │
│  ELSE                                                                   │
│    → Assign to Lead Pool (unassigned queue)                             │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 4.3 Opportunity Automation

| Rule ID | Trigger | Condition | Action |
|---------|---------|-----------|--------|
| O-001 | Stage changed | Any | Notify owner via Google Chat |
| O-002 | Stage changed | Any | Log to pipeline_history |
| O-003 | Discount updated | discount_percent > 15% | Block save, require approval |
| O-004 | Scheduled (daily) | close_date = today, stage != Closed | Alert sales + manager |
| O-005 | Stage = "Closed Won" | - | Create Contract, Notify team |
| O-006 | Stage = "Closed Lost" | - | Log lost reason, Notify manager |
| O-007 | Probability updated | Any | Recalculate weighted_amount |

### 4.4 Deal Won Automation Flow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      DEAL WON AUTOMATION FLOW                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  TRIGGER: Opportunity.stage = "Closed Won"                              │
│                                                                         │
│  ├── Step 1: Update Opportunity                                         │
│  │   • won_date = today()                                               │
│  │   • probability = 100%                                               │
│  │                                                                      │
│  ├── Step 2: Create Contract                                            │
│  │   • Copy account, products, amount                                   │
│  │   • Set status = "Draft"                                             │
│  │   • Link to opportunity                                              │
│  │                                                                      │
│  ├── Step 3: Update Account                                             │
│  │   • account_type = "Customer" (if was Prospect)                      │
│  │   • Update total_revenue (rollup)                                    │
│  │                                                                      │
│  ├── Step 4: Notify via Google Chat                                     │
│  │   • #sales-alerts: "🎉 Deal Won: {opp_name} - {amount}"              │
│  │   • @owner: "Congratulations!"                                       │
│  │   • @am: "New contract ready for setup"                              │
│  │                                                                      │
│  └── Step 5: Update KPIs                                                │
│      • owner.actual_revenue += amount                                   │
│      • owner.actual_deals += 1                                          │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Discount Approval Workflow

### 5.1 Approval Matrix

| Discount Level | Approver(s) | SLA |
|----------------|-------------|-----|
| ≤ 15% | No approval needed | - |
| 16% - 30% | Sales Manager | 4 hours |
| 31% - 50% | Sales Manager → Sales Director | 8 hours |
| > 50% | Sales Manager → Sales Director → CEO | 24 hours |

### 5.2 Approval Process

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     DISCOUNT APPROVAL PROCESS                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Step 1: Sales rep requests discount (>15%)                             │
│          ↓                                                              │
│  Step 2: System blocks save, shows approval form                        │
│          ↓                                                              │
│  Step 3: Directus Flow triggers approval request                        │
│          ↓                                                              │
│  Step 4: Google Chat notification to approver                           │
│          "[Approval Required] Discount 25% for Deal ABC"                │
│          [Approve] [Reject] [View Details]                              │
│          ↓                                                              │
│  Step 5: Approver reviews in Directus or via link                       │
│          ↓                                                              │
│  Step 6a: APPROVED                                                      │
│           • discount_approved = true                                    │
│           • discount_approved_by = approver                             │
│           • Notify sales rep: "Discount approved!"                      │
│          ↓                                                              │
│  Step 6b: REJECTED                                                      │
│           • discount_approved = false                                   │
│           • Notify sales rep: "Discount rejected. Reason: {reason}"     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Integrations

### 6.1 WHMCS Integration

| Direction | Trigger | Action |
|-----------|---------|--------|
| WHMCS → Directus | ClientAdd | Create/Update Account |
| WHMCS → Directus | ClientEdit | Update Account |
| Directus → WHMCS | Deal Won | Create Order (optional) |
| Directus → WHMCS | Account created | Create Client (optional) |

### 6.2 WordPress Integration

| Integration Point | Method | Data Flow |
|-------------------|--------|-----------|
| Lead Capture Forms | Webhook | WordPress → Directus (create lead) |
| Contact Forms | Webhook | WordPress → Directus (create lead) |
| Product Display | REST API | Directus → WordPress (read products) |

### 6.3 Google Chat Notifications

| Event | Channel | Message Template |
|-------|---------|------------------|
| New Lead | #sales-alerts | "🔔 New Lead: {company} - {channel}" |
| Lead Assigned | Direct Message | "Lead assigned to you: {company}" |
| Stage Changed | #sales-alerts | "📊 {opp_name} moved to {stage}" |
| Deal Won | #sales-alerts | "🎉 DEAL WON: {opp_name} - {amount}" |
| Deal Lost | #sales-alerts | "❌ Deal Lost: {opp_name} - {reason}" |
| Stale Alert | Direct + Manager | "⚠️ No activity on {lead/opp} for 7 days" |

---

## 7. User Interface

### 7.1 Sales Portal Pages

| Page | Purpose | Key Features |
|------|---------|--------------|
| Dashboard | KPI overview | Metrics, charts, alerts |
| My Leads | Lead management | List/Kanban, filters, actions |
| My Opportunities | Pipeline | Kanban board, drag-drop stages |
| Accounts | Customer list | Search, filter, account details |
| Quotations | Proposals | Create, preview, send |
| Activities | Task list | Calls, meetings, follow-ups |

### 7.2 Dashboard Metrics

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        SALES DASHBOARD                                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ROW 1: KEY METRICS (Cards)                                             │
│  ┌──────────────┬──────────────┬──────────────┬──────────────┐         │
│  │    TARGET    │    ACTUAL    │  ACHIEVEMENT │  PIPELINE    │         │
│  │     100M     │     75M      │     75%      │    250M      │         │
│  │  This Month  │  This Month  │              │   12 Deals   │         │
│  └──────────────┴──────────────┴──────────────┴──────────────┘         │
│                                                                         │
│  ROW 2: CHARTS                                                          │
│  ┌─────────────────────────────┐ ┌─────────────────────────────┐       │
│  │    Revenue Trend (6 mo)    │ │    Pipeline by Stage        │       │
│  │         [Line Chart]        │ │      [Funnel Chart]         │       │
│  └─────────────────────────────┘ └─────────────────────────────┘       │
│                                                                         │
│  ROW 3: LISTS                                                           │
│  ┌─────────────────────────────┐ ┌─────────────────────────────┐       │
│  │    Deals Closing Soon       │ │    Overdue Follow-ups       │       │
│  │      [Table View]           │ │      [Table View]           │       │
│  └─────────────────────────────┘ └─────────────────────────────┘       │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 8. Reports

### 8.1 Standard Reports

| Report | Frequency | Audience | Key Metrics |
|--------|-----------|----------|-------------|
| Pipeline Report | Daily | Sales Team | Open deals, stages, values |
| Lead Source Analysis | Weekly | Marketing + Sales | Leads by source, conversion |
| Win/Loss Analysis | Monthly | Management | Win rate, lost reasons |
| Sales Performance | Monthly | Management | Rep performance vs target |
| Forecast Report | Weekly | Management | Predicted revenue |

### 8.2 KPI Definitions

| KPI | Formula | Target |
|-----|---------|--------|
| Lead Conversion Rate | Qualified Leads / Total Leads | > 30% |
| Win Rate | Won Deals / Total Closed Deals | > 35% |
| Average Deal Size | Total Won Value / Won Deals | Varies |
| Sales Cycle Length | Avg days from Create to Close | < 30 days |
| Lead Response Time | Time to first contact | < 4 hours |

---

*Document Version: 1.0*
*Last Updated: January 2026*
*Author: HVN GROUP*