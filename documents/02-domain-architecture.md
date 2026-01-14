# HVN Enterprise Platform - Domain Architecture

## 1. Domain Overview

The HVN Enterprise Platform is organized into distinct business domains, each with clear boundaries and responsibilities.

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              HVN DOMAIN ARCHITECTURE                                 │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                      │
│  ┌─────────────────────────────────────────────────────────────────────────────┐   │
│  │                           CUSTOMER-FACING DOMAINS                            │   │
│  │                                                                              │   │
│  │  ┌─────────────────────┐    ┌─────────────────────┐    ┌────────────────┐  │   │
│  │  │   CRM B2B DOMAIN    │    │CUSTOMER SVC DOMAIN │    │ FINANCE DOMAIN │  │   │
│  │  │                     │    │                     │    │                │  │   │
│  │  │ • Leads             │    │ • Implementation    │    │ • Contracts    │  │   │
│  │  │ • Accounts          │    │ • Onboarding        │    │ • Invoices     │  │   │
│  │  │ • Contacts          │    │ • Support Cases     │    │ • Payments     │  │   │
│  │  │ • Opportunities     │    │ • SLA Tracking      │    │ • Cash Flow    │  │   │
│  │  │ • Products/Services │    │ • Knowledge Base    │    │ • Expenses     │  │   │
│  │  │ • Quotations        │    │                     │    │ • Revenue Rec. │  │   │
│  │  │                     │    │                     │    │                │  │   │
│  │  └──────────┬──────────┘    └──────────┬──────────┘    └───────┬────────┘  │   │
│  │             │                          │                        │           │   │
│  │             └──────────────────────────┼────────────────────────┘           │   │
│  │                                        │                                    │   │
│  │                           ┌────────────┴────────────┐                       │   │
│  │                           │   CUSTOMER ENTITY       │                       │   │
│  │                           │   (Single Source)       │                       │   │
│  │                           └─────────────────────────┘                       │   │
│  └─────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                      │
│  ┌─────────────────────────────────────────────────────────────────────────────┐   │
│  │                           INTERNAL OPERATIONS DOMAINS                        │   │
│  │                                                                              │   │
│  │  ┌───────────────────────────────────────────────────────────────────────┐ │   │
│  │  │                         HRM SUITE DOMAIN                               │ │   │
│  │  │                                                                        │ │   │
│  │  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐   │ │   │
│  │  │  │  HVN     │ │  HVN     │ │  HVN     │ │  HVN     │ │  HVN     │   │ │   │
│  │  │  │ Account  │ │ Hiring   │ │ Payroll  │ │ Timeoff  │ │ Checkin  │   │ │   │
│  │  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘   │ │   │
│  │  │                                                                        │ │   │
│  │  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐   │ │   │
│  │  │  │  HVN     │ │  HVN     │ │  HVN     │ │  HVN     │ │  HVN     │   │ │   │
│  │  │  │ Onboard  │ │   LMS    │ │  Case    │ │ Reward   │ │  KPIs    │   │ │   │
│  │  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘   │ │   │
│  │  │                                                                        │ │   │
│  │  │                          ┌──────────┐                                  │ │   │
│  │  │                          │  HVN     │                                  │ │   │
│  │  │                          │ Assets   │                                  │ │   │
│  │  │                          └──────────┘                                  │ │   │
│  │  │                                                                        │ │   │
│  │  │                    ┌───────────────────────┐                          │ │   │
│  │  │                    │    EMPLOYEE ENTITY    │                          │ │   │
│  │  │                    │    (Single Source)    │                          │ │   │
│  │  │                    └───────────────────────┘                          │ │   │
│  │  └───────────────────────────────────────────────────────────────────────┘ │   │
│  └─────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                      │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Domain Definitions

### 2.1 CRM B2B Domain

**Purpose**: Manage the complete sales lifecycle from lead to customer.

**Bounded Context**:
- Lead generation and qualification
- Account and contact management
- Sales pipeline and opportunities
- Quotations and proposals
- Product/service catalog

**Key Entities**:
- `leads` - Potential customers
- `accounts` - Customer companies
- `contacts` - People at customer companies
- `opportunities` - Sales deals
- `products` - Products and services offered
- `quotations` - Price proposals

### 2.2 Customer Service Domain

**Purpose**: Deliver and support services to customers post-sale.

**Bounded Context**:
- Service implementation projects
- Customer onboarding
- Support case management
- SLA monitoring
- Knowledge management

**Key Entities**:
- `implementation_projects` - Service setup projects
- `onboarding_tasks` - Customer onboarding checklist
- `support_cases` - Customer support tickets
- `sla_definitions` - Service level agreements
- `knowledge_articles` - Self-service documentation

### 2.3 Finance Domain

**Purpose**: Manage financial operations, billing, and cash flow.

**Bounded Context**:
- Contract management
- Invoice and payment processing
- Expense management
- Cash flow tracking
- Revenue recognition

**Key Entities**:
- `contracts` - Customer contracts
- `invoices` - Customer invoices
- `payments` - Payment records
- `expenses` - Company expenses
- `cash_flow` - Cash movement records

### 2.4 HRM Suite Domain

**Purpose**: Manage complete employee lifecycle and HR operations.

**Bounded Context**:
- Recruitment and hiring
- Employee information management
- Time and attendance
- Payroll processing
- Learning and development
- Performance management

**Key Entities**:
- `employees` - Employee master data
- `candidates` - Job applicants
- `attendance_records` - Check-in/out logs
- `payroll_records` - Salary records
- `courses` - Training courses
- `kpi_records` - Performance metrics

---

## 3. Entity Relationships

### 3.1 Customer-Centric Entity Model

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                            CORE ENTITY RELATIONSHIPS                                 │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                      │
│                                  ┌──────────────┐                                   │
│                                  │   CUSTOMER   │                                   │
│                                  │   (Master)   │                                   │
│                                  └──────┬───────┘                                   │
│                                         │                                           │
│           ┌─────────────────────────────┼─────────────────────────────┐             │
│           │                             │                             │             │
│           ▼                             ▼                             ▼             │
│  ┌─────────────────┐          ┌─────────────────┐          ┌─────────────────┐     │
│  │    CONTACTS     │          │    ACCOUNTS     │          │  WHMCS_CLIENTS  │     │
│  │  (CRM Domain)   │          │  (CRM Domain)   │          │   (External)    │     │
│  └────────┬────────┘          └────────┬────────┘          └─────────────────┘     │
│           │                            │                                            │
│           │                            │                                            │
│           ▼                            ▼                                            │
│  ┌─────────────────┐          ┌─────────────────┐                                  │
│  │     LEADS       │          │ OPPORTUNITIES   │                                  │
│  └─────────────────┘          └────────┬────────┘                                  │
│                                        │                                            │
│                          ┌─────────────┼─────────────┐                             │
│                          │             │             │                             │
│                          ▼             ▼             ▼                             │
│                 ┌─────────────┐ ┌───────────┐ ┌───────────┐                        │
│                 │ QUOTATIONS  │ │ CONTRACTS │ │  ORDERS   │                        │
│                 └─────────────┘ └─────┬─────┘ │ (WHMCS)   │                        │
│                                       │       └───────────┘                        │
│                                       │                                            │
│                          ┌────────────┼────────────┐                               │
│                          │            │            │                               │
│                          ▼            ▼            ▼                               │
│                 ┌─────────────┐ ┌───────────┐ ┌───────────┐                        │
│                 │  INVOICES   │ │ PAYMENTS  │ │ SERVICES  │                        │
│                 └─────────────┘ └───────────┘ └───────────┘                        │
│                                                    │                               │
│                                                    ▼                               │
│                                           ┌───────────────┐                        │
│                                           │SUPPORT CASES  │                        │
│                                           └───────────────┘                        │
│                                                                                      │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### 3.2 Employee-Centric Entity Model

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                           HRM ENTITY RELATIONSHIPS                                   │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                      │
│                                 ┌──────────────┐                                    │
│                                 │   EMPLOYEE   │                                    │
│                                 │   (Master)   │                                    │
│                                 └──────┬───────┘                                    │
│                                        │                                            │
│    ┌───────────────┬───────────────┬───┼───┬───────────────┬───────────────┐       │
│    │               │               │   │   │               │               │       │
│    ▼               ▼               ▼   │   ▼               ▼               ▼       │
│ ┌───────┐    ┌─────────┐    ┌─────────┐│┌─────────┐  ┌─────────┐    ┌─────────┐   │
│ │CHECKIN│    │ TIMEOFF │    │ PAYROLL ││ │  KPIs   │  │  CASES  │    │ REWARDS │   │
│ │Records│    │Requests │    │ Records ││ │ Records │  │(Violate)│    │ Records │   │
│ └───────┘    └─────────┘    └─────────┘│└─────────┘  └─────────┘    └─────────┘   │
│                                        │                                            │
│    ┌───────────────┬───────────────┬───┘                                           │
│    │               │               │                                                │
│    ▼               ▼               ▼                                                │
│ ┌───────────┐ ┌───────────┐ ┌───────────┐                                          │
│ │  ASSETS   │ │   LMS     │ │ CONTRACTS │                                          │
│ │ Assigned  │ │ Courses   │ │   (HR)    │                                          │
│ └───────────┘ └─────┬─────┘ └───────────┘                                          │
│                     │                                                               │
│         ┌───────────┼───────────┐                                                  │
│         │           │           │                                                  │
│         ▼           ▼           ▼                                                  │
│    ┌─────────┐ ┌─────────┐ ┌─────────┐                                             │
│    │ MODULES │ │ LESSONS │ │ QUIZZES │                                             │
│    └─────────┘ └─────────┘ └─────────┘                                             │
│                                                                                      │
│                                                                                      │
│                       ┌──────────────┐                                              │
│                       │  CANDIDATE   │                                              │
│                       │  (Hiring)    │                                              │
│                       └──────┬───────┘                                              │
│                              │                                                      │
│              ┌───────────────┼───────────────┐                                     │
│              │               │               │                                     │
│              ▼               ▼               ▼                                     │
│       ┌───────────┐   ┌───────────┐   ┌───────────┐                               │
│       │APPLICATIONS│   │INTERVIEWS │   │   TESTS   │                               │
│       └───────────┘   └───────────┘   └───────────┘                               │
│              │                                                                      │
│              ▼ (if hired)                                                          │
│       ┌───────────┐                                                                │
│       │ ONBOARDING│                                                                │
│       └─────┬─────┘                                                                │
│             │                                                                       │
│             ▼                                                                       │
│       ┌───────────┐                                                                │
│       │ EMPLOYEE  │ (Created)                                                      │
│       └───────────┘                                                                │
│                                                                                      │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Cross-Domain Relationships

### 4.1 Customer → Employee Links

| From Domain | To Domain | Relationship | Use Case |
|-------------|-----------|--------------|----------|
| CRM | HRM | Account → Account Manager | Sales assignment |
| CRM | HRM | Opportunity → Sales Rep | Deal ownership |
| Customer Service | HRM | Case → Support Agent | Ticket assignment |
| Finance | HRM | Expense → Requester | Expense claims |
| Finance | HRM | Payment → Verifier | Payment approval |

### 4.2 Shared Entities

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                           SHARED ENTITIES ACROSS DOMAINS                             │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                      │
│  ┌─────────────────────────────────────────────────────────────────────────────┐   │
│  │                         DIRECTUS_USERS (System)                              │   │
│  │                                                                              │   │
│  │  Used by: All domains for user authentication and audit trails              │   │
│  │                                                                              │   │
│  │  Links to:                                                                   │   │
│  │  • employees (for internal staff)                                            │   │
│  │  • External customers (optional, for portal access)                          │   │
│  └─────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                      │
│  ┌─────────────────────────────────────────────────────────────────────────────┐   │
│  │                         DEPARTMENTS (Organization)                           │   │
│  │                                                                              │   │
│  │  Used by: HRM (employee assignment), CRM (team structure)                    │   │
│  │                                                                              │   │
│  │  Links to:                                                                   │   │
│  │  • employees.department                                                      │   │
│  │  • teams.department                                                          │   │
│  │  • kpis.department_target                                                    │   │
│  └─────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                      │
│  ┌─────────────────────────────────────────────────────────────────────────────┐   │
│  │                         PRODUCTS (Catalog)                                   │   │
│  │                                                                              │   │
│  │  Used by: CRM (quotes), Finance (contracts), WHMCS (orders)                  │   │
│  │                                                                              │   │
│  │  Links to:                                                                   │   │
│  │  • opportunity_products                                                      │   │
│  │  • quotation_items                                                           │   │
│  │  • contract_items                                                            │   │
│  │  • whmcs_products (sync)                                                     │   │
│  └─────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                      │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Domain Access Control

### 5.1 Role-Domain Matrix

| Role | CRM | Customer Service | Finance | HRM |
|------|-----|------------------|---------|-----|
| Super Admin | Full | Full | Full | Full |
| Sales Manager | Full | View Cases | View Own Accounts | View Team |
| Sales Rep | Own Leads/Opps | View Own Cases | View Own Accounts | Self Only |
| Account Manager | View Accounts | Own Projects | View Own Accounts | Self Only |
| Support Agent | View Accounts | Own Cases | - | Self Only |
| Finance Manager | View | View | Full | View |
| Accountant | View | - | CRUD | Self Only |
| HR Director | View | - | View | Full |
| HR Staff | - | - | - | Assigned Functions |
| Employee | - | - | - | Self Only |

### 5.2 Data Visibility Rules

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                           DATA VISIBILITY RULES                                      │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                      │
│  CRM DOMAIN:                                                                        │
│  ───────────                                                                        │
│  • Leads: Owner sees own, Manager sees team, Director sees all                      │
│  • Accounts: AM sees assigned, Sales sees linked, All can view                      │
│  • Opportunities: Owner sees own, Manager sees team pipeline                        │
│                                                                                      │
│  CUSTOMER SERVICE DOMAIN:                                                           │
│  ─────────────────────────                                                          │
│  • Cases: Agent sees assigned, Supervisor sees queue, Admin sees all                │
│  • Projects: PM sees own projects, Director sees all                                │
│                                                                                      │
│  FINANCE DOMAIN:                                                                    │
│  ───────────────                                                                    │
│  • Invoices: Finance team full access, Others view own customers                    │
│  • Payments: Finance team only                                                      │
│  • Expenses: Submitter sees own, Manager approves team                              │
│                                                                                      │
│  HRM DOMAIN:                                                                        │
│  ───────────                                                                        │
│  • Employee data: Self sees own, Manager sees direct reports                        │
│  • Payroll: HR + Finance only                                                       │
│  • Cases/Rewards: Self sees own, HR full access                                     │
│  • Attendance: Self sees own, Manager sees team                                     │
│                                                                                      │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

*Document Version: 1.0*
*Last Updated: January 2026*
*Author: HVN GROUP*