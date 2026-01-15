# HVN Enterprise Platform - Architecture Overview

## 1. Introduction

### 1.1 Document Purpose

This document provides a comprehensive system architecture blueprint for the HVN Enterprise Platform - a unified B2B CRM and Business Management System built on Directus headless CMS.

### 1.2 Scope

The platform encompasses:
- **CRM B2B Core**: Customer management, Sales Pipeline
- **Customer Service**: Implementation, Onboarding, Support Cases
- **Finance & Billing**: Contracts, Invoices, Payments, Cash Flow
- **HRM Suite**: 12 HR modules for complete employee lifecycle management
- **Operations**: Asset management, Service delivery

### 1.3 Technology Stack

| Layer | Technology |
|-------|------------|
| Frontend | React 19 + Vite + Ant Design 5 |
| Backend/CMS | Directus 13+ |
| Database | PostgreSQL (Cloud SQL) |
| Cloud Platform | Google Cloud Platform |
| Authentication | Google OAuth + Directus Auth |
| Notifications | Google Chat |
| External Systems | WHMCS, WordPress, Payment Gateways |

---

## 2. High-Level Architecture

```
┌───────────────────────────────────────────────────────────────────────────────────┐
│                           HVN ENTERPRISE PLATFORM                                 │
│                        (Unified Business Ecosystem)                               │
├───────────────────────────────────────────────────────────────────────────────────┤
│                                                                                   │
│  ┌────────────────────────────────────────────────────────────────────────────┐   │
│  │                        PRESENTATION LAYER                                  │   │
│  │  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐ │   │
│  │  │ WordPress │  │  HVN Apps │  │  Staff    │  │   WHMCS   │  │   Admin   │ │   │
│  │  │  (Public) │  │  (React)  │  │  Portal   │  │ (Customer │  │  Console  │ │   │ 
│  │  │           │  │           │  │ (React)   │  │  Portal)  │  │(Directus) │ │   │
│  │  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘ │   │
│  └────────┼──────────────┼──────────────┼──────────────┼──────────────┼───────┘   │
│           │              │              │              │              │           │
│           └──────────────┴──────────────┴──────────────┴──────────────┘           │
│                                         │                                         │
│  ┌──────────────────────────────────────┼──────────────────────────────────────┐  │
│  │                           API GATEWAY LAYER                                 │  │
│  │                    (Authentication, Rate Limiting, Routing)                 │  │
│  └──────────────────────────────────────┼──────────────────────────────────────┘  │
│                                         │                                         │
│  ┌──────────────────────────────────────┴──────────────────────────────────────┐  │
│  │                                                                             │  │
│  │  ┌──────────────────────────────────────────────────────────────────────┐   │  │
│  │  │                    DIRECTUS CORE PLATFORM                            │   │  │
│  │  │              (Headless CMS + API + Data Engine)                      │   │  │
│  │  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐    │   │  │
│  │  │  │   CRM    │ │ Customer │ │  Finance │ │   HRM    │ │Operations│    │   │  │
│  │  │  │   B2B    │ │ Service  │ │ Billing  │ │  Suite   │ │ & Assets │    │   │  │
│  │  │  └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘    │   │  │
│  │  │       └────────────┴────────────┴────────────┴────────────┘          │   │  │
│  │  │                              │                                       │   │  │
│  │  │                    ┌─────────┴─────────┐                             │   │  │
│  │  │                    │   UNIFIED DATA    │                             │   │  │
│  │  │                    │      MODEL        │                             │   │  │
│  │  │                    └───────────────────┘                             │   │  │
│  │  └──────────────────────────────────────────────────────────────────────┘   │  │
│  │                                                                             │  │
│  │  ┌───────────────────────────────┐  ┌───────────────────────────────────┐   │  │
│  │  │   INTEGRATION LAYER           │  │      AUTOMATION ENGINE            │   │  │
│  │  │  ┌─────────┐ ┌─────────┐      │  │  ┌──────────┐  ┌──────────┐       │   │  │
│  │  │  │ WHMCS   │ │ Google  │      │  │  │ Directus │  │ External │       │   │  │
│  │  │  │ Adapter │ │Workspace│      │  │  │  Flows   │  │ Webhooks │       │   │  │
│  │  │  │         │ │ Adapter │      │  │  │          │  │          │       │   │  │
│  │  │  └─────────┘ └─────────┘      │  │  └──────────┘  └──────────┘       │   │  │
│  │  │  ┌─────────┐ ┌─────────┐      │  │  ┌──────────┐  ┌──────────┐       │   │  │
│  │  │  │ Payment │ │  Email  │      │  │  │Scheduled │  │  Event   │       │   │  │
│  │  │  │ Gateway │ │ Service │      │  │  │  Tasks   │  │ Triggers │       │   │  │
│  │  │  └─────────┘ └─────────┘      │  │  └──────────┘  └──────────┘       │   │  │
│  │  └───────────────────────────────┘  └───────────────────────────────────┘   │  │
│  │                                                                             │  │
│  └─────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                   │
│  ┌─────────────────────────────────────────────────────────────────────────────┐  │
│  │                        EXTERNAL SYSTEMS                                     │  │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐    │  │
│  │  │  WHMCS  │ │ Google  │ │ Google  │ │ Payment │ │ SMS/OTP │ │ Storage │    │  │
│  │  │ (Multi) │ │Workspace│ │  Chat   │ │ Gateway │ │ Service │ │  (GCS)  │    │  │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘    │  │
│  └─────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                   │
└───────────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Presentation Layer Components

### 3.1 Component Overview

| Component | Purpose | Technology | Users |
|-----------|---------|------------|-------|
| **WordPress** | Public website, marketing, lead capture | WordPress + Elementor | Public visitors |
| **HVN App** | Main business application | React 19 + Ant Design | Internal staff |
| **Staff Portal** | Employee self-service | React (part of HVN App) | All employees |
| **WHMCS Customer Portal** | Customer self-service, billing, support | WHMCS native | Customers |
| **Admin Console** | System administration | Directus Admin UI | System admins |

### 3.2 WHMCS as Customer Portal

WHMCS serves as the primary customer-facing portal, providing:

- **Account Management**: Customer profile, contacts
- **Billing & Invoices**: View/pay invoices, payment history
- **Services Management**: Active services, renewals
- **Support Tickets**: Create and track support cases
- **Knowledge Base**: Self-service documentation
- **Downloads**: Software, certificates, documents

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    WHMCS CUSTOMER PORTAL FUNCTIONS                      │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐        │
│  │   MY ACCOUNT    │  │    BILLING      │  │    SERVICES     │        │
│  │                 │  │                 │  │                 │        │
│  │ • Profile       │  │ • Invoices      │  │ • Active        │        │
│  │ • Contacts      │  │ • Payments      │  │ • Pending       │        │
│  │ • Security      │  │ • Credit        │  │ • Cancelled     │        │
│  │ • Notifications │  │ • Statements    │  │ • Renewals      │        │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘        │
│                                                                         │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐        │
│  │    SUPPORT      │  │  KNOWLEDGE BASE │  │   DOWNLOADS     │        │
│  │                 │  │                 │  │                 │        │
│  │ • Open Ticket   │  │ • Articles      │  │ • Software      │        │
│  │ • View Tickets  │  │ • FAQs          │  │ • Certificates  │        │
│  │ • Announcements │  │ • Tutorials     │  │ • Documents     │        │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘        │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Architecture Principles

### 4.1 Core Principles

| Principle | Description | Implementation |
|-----------|-------------|----------------|
| **Single Source of Truth** | Directus is the central database for all business data | All modules write to Directus |
| **Loosely Coupled Integration** | External systems connect via APIs/Webhooks | WHMCS, Google via adapters |
| **Event-Driven Architecture** | Changes trigger automated workflows | Directus Flows handle events |
| **Role-Based Access Control** | Fine-grained permissions by domain | Directus ACL system |
| **Scalable Infrastructure** | Auto-scaling cloud deployment | GCP Cloud Run |

### 4.2 Data Flow Principles

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         DATA FLOW PRINCIPLES                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. INBOUND DATA (External → Directus)                                  │
│     ─────────────────────────────────────                               │
│     • WHMCS webhooks → Directus (customers, invoices, payments)         │
│     • WordPress forms → Directus (leads)                                │
│     • Payment gateway → Directus (payment confirmations)                │
│                                                                         │
│  2. OUTBOUND DATA (Directus → External)                                 │
│     ─────────────────────────────────────                               │
│     • Directus → WHMCS (create orders, update clients)                  │
│     • Directus → Google Chat (notifications)                            │
│     • Directus → Email service (transactional emails)                   │
│                                                                         │
│  3. INTERNAL DATA (Within Directus)                                     │
│     ──────────────────────────────────                                  │
│     • Cross-module references (Customer → Contracts → Invoices)         │
│     • Automated calculations (KPIs, aggregations)                       │
│     • Audit trails (all changes logged)                                 │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 5. System Boundaries

### 5.1 What Lives in Directus

- All CRM data (Leads, Accounts, Contacts, Opportunities)
- All HR data (Employees, Attendance, Payroll, Training)
- All Finance data (Contracts, consolidated Invoices, Cash Flow)
- All Customer Service data (Cases, SLAs, Implementation projects)
- Business logic and workflows
- Reporting and analytics

### 5.2 What Lives in WHMCS

- Product catalog and pricing
- Order processing
- Billing and invoicing (source of truth for billing)
- Customer portal interface
- Payment processing
- Service provisioning automation

### 5.3 What Lives in WordPress

- Public website content
- Marketing pages
- Blog and SEO content
- Lead capture forms (data flows to Directus)

---

## 6. Document Structure

This architecture documentation is organized into the following files:

| File | Content |
|------|---------|
| `01-architecture-overview.md` | This file - High-level architecture |
| `02-domain-architecture.md` | Domain separation and entity relationships |
| `03-crm-module.md` | CRM B2B module detailed design |
| `04-customer-service-module.md` | Customer Service module detailed design |
| `05-finance-module.md` | Finance & Billing module detailed design |
| `06-hrm-suite-module.md` | HRM Suite modules detailed design |
| `07-integration-architecture.md` | WHMCS, Google, Payment integrations |
| `08-technical-infrastructure.md` | Deployment, security, performance |
| `09-implementation-roadmap.md` | Phased implementation plan |

---

*Document Version: 1.0*
*Last Updated: January 2026*
*Author: HVN GROUP*