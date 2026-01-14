# HVN Enterprise Platform - Customer Service Module

## 1. Module Overview

### 1.1 Purpose

The Customer Service module manages post-sale customer engagement including service implementation, onboarding, and ongoing support.

### 1.2 Key Capabilities

- Service implementation project tracking
- Customer onboarding workflows
- Omnichannel support case management
- SLA monitoring and escalation
- Knowledge base management
- Customer satisfaction tracking

---

## 2. Service Delivery Lifecycle

### 2.1 Lifecycle Stages

| Stage | Duration | Owner | Activities |
|-------|----------|-------|------------|
| **Deal Won** | Day 0 | Sales | Handoff to Customer Success |
| **Implementation** | 1-4 weeks | CSM | Setup, configuration, testing |
| **Onboarding** | 1-2 weeks | CSM | Training, documentation, go-live |
| **Active Service** | Ongoing | Support | Support cases, renewals, upsell |

### 2.2 Implementation Project Structure

```javascript
// Collection: implementation_projects
{
  id: "uuid",
  status: "published",
  
  // Project Info
  project_name: "string",
  account: "m2o:accounts",
  contract: "m2o:contracts",
  opportunity: "m2o:opportunities",
  
  // Timeline
  start_date: "date",
  target_end_date: "date",
  actual_end_date: "date",
  
  // Status
  project_status: "single_select",  // Planning, In Progress, Testing, Completed, On Hold
  progress_percent: "integer",       // 0-100
  
  // Team
  project_manager: "m2o:employees",
  team_members: "m2m:employees",
  
  // Tasks
  tasks: "o2m:implementation_tasks",
  
  // Notes
  notes: "text",
  blockers: "text",
  
  // System
  user_created: "user",
  date_created: "datetime"
}
```

---

## 3. Support Case Management

### 3.1 Omnichannel Intake

| Channel | Source | Integration Method |
|---------|--------|-------------------|
| **Email** | support@hvn.vn | Email parsing → Directus |
| **Phone** | Call log | Manual entry |
| **WHMCS Ticket** | Customer portal | WHMCS webhook → Directus |
| **Chat** | Website widget | Webhook → Directus |
| **Internal** | Staff request | Direct entry |

### 3.2 Support Case Collection

```javascript
// Collection: support_cases
{
  id: "uuid",
  status: "published",
  
  // Case Info
  case_number: "string",         // Auto-generated: CS-YYYYMMDD-XXX
  subject: "string",
  description: "text",
  
  // Classification
  case_type: "single_select",    // Technical, Billing, General, Feature Request
  priority: "single_select",     // Critical, High, Medium, Low
  category: "m2o:case_categories",
  
  // Customer
  account: "m2o:accounts",
  contact: "m2o:contacts",
  
  // Status
  case_status: "single_select",  // New, In Progress, Pending Customer, Resolved, Closed
  
  // Assignment
  assigned_to: "m2o:employees",
  queue: "m2o:support_queues",
  
  // SLA
  sla_definition: "m2o:sla_definitions",
  first_response_due: "datetime",
  resolution_due: "datetime",
  first_response_at: "datetime",
  resolved_at: "datetime",
  sla_breached: "boolean",
  
  // Source
  source_channel: "single_select",  // Email, Phone, WHMCS, Chat, Internal
  whmcs_ticket_id: "string",        // If from WHMCS
  
  // Resolution
  resolution_notes: "text",
  root_cause: "single_select",
  
  // Related
  related_cases: "m2m:support_cases",
  related_services: "m2m:whmcs_services",
  
  // Interactions
  comments: "o2m:case_comments",
  attachments: "o2m:case_attachments",
  
  // Feedback
  csat_score: "integer",           // 1-5
  csat_feedback: "text",
  
  // System
  user_created: "user",
  date_created: "datetime"
}
```

### 3.3 Case Status Flow

```
NEW ──▶ IN PROGRESS ──▶ PENDING CUSTOMER ──▶ IN PROGRESS ──▶ RESOLVED ──▶ CLOSED
 │           │                  │                               │
 │           │                  └───────────────────────────────┘
 │           │                    (Customer responds)
 │           │
 └───────────┴──▶ ESCALATED (if SLA breach imminent)
```

---

## 4. SLA Management

### 4.1 SLA Definitions

| Priority | First Response | Resolution | Escalation Path |
|----------|---------------|------------|-----------------|
| **Critical** | 1 hour | 4 hours | Agent → Supervisor → Manager → Director |
| **High** | 4 hours | 8 hours | Agent → Supervisor → Manager |
| **Medium** | 8 hours | 24 hours | Agent → Supervisor |
| **Low** | 24 hours | 72 hours | Agent |

### 4.2 SLA Collection

```javascript
// Collection: sla_definitions
{
  id: "uuid",
  
  sla_name: "string",            // "Critical SLA", "Standard SLA"
  priority: "single_select",     // Critical, High, Medium, Low
  
  // Targets (in minutes)
  first_response_minutes: "integer",
  resolution_minutes: "integer",
  
  // Business Hours
  business_hours_only: "boolean",
  business_start: "time",        // 08:00
  business_end: "time",          // 18:00
  business_days: "json",         // [1,2,3,4,5] (Mon-Fri)
  
  // Escalation
  escalation_levels: "json",     // [{minutes: 60, notify: "supervisor"}, ...]
  
  active: "boolean"
}
```

### 4.3 SLA Monitoring Automation

| Rule | Trigger | Condition | Action |
|------|---------|-----------|--------|
| SLA-001 | Case created | Has SLA | Calculate due dates |
| SLA-002 | Scheduled (5 min) | first_response_due in 30 min | Alert agent |
| SLA-003 | Scheduled (5 min) | first_response_due passed | Escalate + flag breach |
| SLA-004 | Scheduled (5 min) | resolution_due in 1 hour | Alert agent + supervisor |
| SLA-005 | Scheduled (5 min) | resolution_due passed | Escalate + flag breach |
| SLA-006 | Case resolved | Any | Calculate actual SLA metrics |

---

## 5. WHMCS Support Integration

### 5.1 Ticket Sync Flow

```
WHMCS Customer Portal                    Directus CRM
      │                                       │
      │  [Customer opens ticket]              │
      ▼                                       │
  WHMCS Ticket ──── Webhook ────────────▶ Create support_case
      │                                       │
      │  [Staff replies in Directus]          │
      │                                       ▼
      │              ◀─── API Call ──── Update WHMCS ticket
      │                                       │
      │  [Customer replies in WHMCS]          │
      ▼                                       │
  WHMCS Reply ───── Webhook ────────────▶ Add case_comment
```

### 5.2 Sync Rules

| WHMCS Event | Directus Action |
|-------------|-----------------|
| TicketOpen | Create support_case |
| TicketReply (Customer) | Add case_comment |
| TicketReply (Staff) | Add case_comment (sync back) |
| TicketClose | Update case_status = Closed |
| TicketStatusChange | Update case_status |

### 5.3 Why Dual System?

- **WHMCS**: Customer-facing portal (they already use it)
- **Directus**: Internal management, unified view, analytics
- **Benefit**: Customers use familiar portal, staff have full CRM context

---

## 6. Knowledge Base

### 6.1 Article Structure

```javascript
// Collection: knowledge_articles
{
  id: "uuid",
  status: "published",  // draft, published, archived
  
  // Content
  title: "string",
  slug: "string",
  content: "text",       // Markdown
  excerpt: "string",
  
  // Classification
  category: "m2o:kb_categories",
  tags: "m2m:kb_tags",
  
  // Visibility
  visibility: "single_select",  // Public, Internal, Customer
  
  // Related
  related_products: "m2m:products",
  related_articles: "m2m:knowledge_articles",
  
  // Metrics
  view_count: "integer",
  helpful_count: "integer",
  not_helpful_count: "integer",
  
  // System
  author: "m2o:employees",
  user_created: "user",
  date_created: "datetime"
}
```

### 6.2 Article Visibility

| Visibility | Who Can Access |
|------------|----------------|
| **Public** | Anyone (synced to WHMCS KB) |
| **Customer** | Logged-in customers (WHMCS) |
| **Internal** | Staff only (Directus) |

---

## 7. Customer Satisfaction (CSAT)

### 7.1 Survey Trigger

| Trigger | Survey Type | Timing |
|---------|-------------|--------|
| Case Resolved | Post-case CSAT | Immediately after resolution |
| Implementation Complete | Onboarding NPS | 1 week after go-live |
| Contract Renewal | Relationship NPS | 30 days before renewal |

### 7.2 CSAT Collection

```javascript
// Collection: csat_surveys
{
  id: "uuid",
  
  // Reference
  survey_type: "single_select",   // Case, Onboarding, Renewal
  support_case: "m2o:support_cases",
  account: "m2o:accounts",
  contact: "m2o:contacts",
  
  // Response
  score: "integer",               // 1-5 or 0-10 (NPS)
  feedback: "text",
  
  // Timestamps
  sent_at: "datetime",
  responded_at: "datetime",
  
  // System
  user_created: "user",
  date_created: "datetime"
}
```

---

## 8. Automation Rules

### 8.1 Case Automation

| Rule | Trigger | Condition | Action |
|------|---------|-----------|--------|
| CS-001 | Case created | Any | Auto-assign to queue based on category |
| CS-002 | Case created | From WHMCS | Link to account via whmcs_client_id |
| CS-003 | Case created | Priority = Critical | Notify team lead immediately |
| CS-004 | Case status changed | Resolved | Send CSAT survey after 1 hour |
| CS-005 | Scheduled (daily) | Open cases > 7 days | Alert manager |

### 8.2 Google Chat Notifications

| Event | Channel | Message |
|-------|---------|---------|
| Critical case created | #support-alerts | "🚨 Critical Case: {subject} - {account}" |
| SLA breach imminent | Direct + Supervisor | "⚠️ SLA warning: Case {number} due in 30 min" |
| SLA breached | #support-alerts + Manager | "❌ SLA BREACH: Case {number}" |
| Case resolved | #support-alerts | "✅ Resolved: {case_number}" |

---

## 9. Dashboards & Reports

### 9.1 Support Dashboard

| Metric | Description |
|--------|-------------|
| Open Cases | Count by priority |
| SLA Performance | % cases meeting SLA |
| Avg Resolution Time | By priority, by agent |
| CSAT Score | Average and trend |
| Cases by Category | Distribution |
| Agent Performance | Cases handled, resolution time |

### 9.2 Key Reports

| Report | Frequency | Audience |
|--------|-----------|----------|
| Daily Support Summary | Daily | Support Team |
| SLA Performance | Weekly | Management |
| Agent Productivity | Weekly | Supervisors |
| Customer Satisfaction | Monthly | Management |
| Top Issues | Monthly | Product Team |

---

*Document Version: 1.0*
*Last Updated: January 2026*
*Author: HVN GROUP*