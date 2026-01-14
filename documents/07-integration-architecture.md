# HVN Enterprise Platform - Integration Architecture

## 1. Integration Overview

### 1.1 External Systems

| System | Purpose | Integration Method |
|--------|---------|-------------------|
| **WHMCS** (Multi) | Billing, Customer Portal | REST API + Webhooks |
| **Google Workspace** | Auth, Calendar, Drive, Chat | OAuth + APIs |
| **WordPress** | Public website, Lead capture | REST API + Webhooks |
| **Payment Gateways** | Online payments | Webhooks |
| **Email Service** | Transactional emails | SMTP/API |

### 1.2 Data Flow Patterns

| Pattern | Direction | Use Case |
|---------|-----------|----------|
| Webhook Inbound | External → Directus | WHMCS events, Payments |
| API Outbound | Directus → External | Create WHMCS order, Send notifications |
| Bidirectional Sync | Both | Client/Account sync |
| Event-Driven | Trigger → Action | Notifications, Automations |

---

## 2. WHMCS Integration

### 2.1 Multi-Instance Architecture

HVN operates multiple WHMCS instances for different brands. Directus consolidates data from all instances.

```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   WHMCS #1   │  │   WHMCS #2   │  │   WHMCS #N   │
│   (hvn.vn)   │  │  (brand-b)   │  │  (brand-n)   │
└──────┬───────┘  └──────┬───────┘  └──────┬───────┘
       │                 │                 │
       └─────────────────┼─────────────────┘
                         │
                         ▼
              ┌──────────────────────┐
              │   DIRECTUS          │
              │   (Consolidated)    │
              │                     │
              │   • Unified CRM     │
              │   • Cross-brand AR  │
              │   • Reporting       │
              └──────────────────────┘
```

### 2.2 WHMCS Webhook Events

| WHMCS Hook | Payload | Directus Action |
|------------|---------|-----------------|
| ClientAdd | Client data | Create/Update Account |
| ClientEdit | Client data | Update Account |
| InvoiceCreation | Invoice data | Create Invoice |
| InvoicePaid | Invoice + Transaction | Update Invoice, Create Payment |
| InvoiceRefunded | Refund data | Create negative Payment |
| TicketOpen | Ticket data | Create Support Case |
| TicketReply | Reply data | Add Case Comment |
| TicketStatusChange | Status | Update Case Status |
| OrderAccept | Order data | Log Order, Update Account |

### 2.3 Webhook Handler (Directus Flow)

```javascript
// Webhook endpoint: POST /whmcs-webhook/:instance_id

// Step 1: Validate webhook
const payload = request.body;
const instance = await getWhmcsInstance(instance_id);
if (!validateWebhookSignature(payload, instance.webhook_secret)) {
  return { status: 401, error: "Invalid signature" };
}

// Step 2: Route by event type
const eventType = payload.event;
switch(eventType) {
  case 'ClientAdd':
  case 'ClientEdit':
    await syncClient(payload, instance);
    break;
  case 'InvoiceCreation':
    await createInvoice(payload, instance);
    break;
  case 'InvoicePaid':
    await processPayment(payload, instance);
    break;
  case 'TicketOpen':
    await createSupportCase(payload, instance);
    break;
  // ... other handlers
}

// Step 3: Log sync
await logSync(instance, eventType, payload);
```

### 2.4 Client → Account Mapping

```javascript
// Function: syncClient(payload, instance)

async function syncClient(payload, instance) {
  const whmcsClient = payload.client;
  
  // Step 1: Find existing account
  let account = await findAccount({
    whmcs_client_id: whmcsClient.id,
    whmcs_instance: instance.id
  });
  
  // Step 2: If not found, try matching by email/company
  if (!account) {
    account = await findAccount({
      OR: [
        { email: whmcsClient.email },
        { company_name: whmcsClient.companyname }
      ]
    });
  }
  
  // Step 3: Create or update
  const accountData = {
    company_name: whmcsClient.companyname || whmcsClient.firstname + ' ' + whmcsClient.lastname,
    // ... map other fields
    whmcs_client_id: whmcsClient.id,
    whmcs_instance: instance.id
  };
  
  if (account) {
    await updateAccount(account.id, accountData);
  } else {
    await createAccount(accountData);
  }
}
```

### 2.5 Directus → WHMCS API Calls

| Action | WHMCS API | Trigger |
|--------|-----------|---------|
| Create Client | AddClient | New Account (optional) |
| Update Client | UpdateClient | Account updated |
| Add Credit | AddCredit | Manual credit |
| Create Order | AddOrder | Won Opportunity (optional) |
| Reply Ticket | AddTicketReply | Support case reply |

```javascript
// Example: Create WHMCS order from won opportunity

async function createWhmcsOrder(opportunity, whmcsInstance) {
  const account = await getAccount(opportunity.account);
  
  // Call WHMCS API
  const response = await whmcsApi(whmcsInstance, 'AddOrder', {
    clientid: account.whmcs_client_id,
    pid: opportunity.products.map(p => p.whmcs_product_id),
    billingcycle: 'annually',
    // ... other params
  });
  
  return response;
}
```

---

## 3. Google Workspace Integration

### 3.1 Services Used

| Service | Purpose | Use Cases |
|---------|---------|-----------|
| **Google OAuth** | Authentication | Staff login via Google account |
| **Google Chat** | Notifications | Alerts, approvals, updates |
| **Google Calendar** | Scheduling | Interviews, meetings, reminders |
| **Google Drive** | Storage | Contract files, documents |
| **Gmail API** | Email | Transactional emails (optional) |

### 3.2 Google OAuth Configuration

```javascript
// Directus SSO Configuration (env)

AUTH_GOOGLE_DRIVER=oauth2
AUTH_GOOGLE_CLIENT_ID=xxx.apps.googleusercontent.com
AUTH_GOOGLE_CLIENT_SECRET=xxx
AUTH_GOOGLE_AUTHORIZE_URL=https://accounts.google.com/o/oauth2/auth
AUTH_GOOGLE_ACCESS_URL=https://oauth2.googleapis.com/token
AUTH_GOOGLE_PROFILE_URL=https://www.googleapis.com/oauth2/v2/userinfo
AUTH_GOOGLE_SCOPE=email profile
AUTH_GOOGLE_ALLOW_PUBLIC_REGISTRATION=false
AUTH_GOOGLE_DEFAULT_ROLE_ID=xxx
AUTH_GOOGLE_ICON=google

// Restrict to company domain
AUTH_GOOGLE_PARAMS={"hd": "hvn.vn"}
```

### 3.3 Google Chat Notifications

```javascript
// Collection: notification_channels
{
  id: "uuid",
  channel_type: "google_chat",
  channel_name: "Sales Alerts",
  webhook_url: "https://chat.googleapis.com/v1/spaces/xxx/messages?key=xxx",
  active: "boolean"
}

// Notification template
async function sendGoogleChatNotification(channel, message, card = null) {
  const payload = card ? {
    cards: [card]
  } : {
    text: message
  };
  
  await fetch(channel.webhook_url, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(payload)
  });
}

// Rich card example
const card = {
  header: {
    title: "🎉 Deal Won!",
    subtitle: opportunity.opp_name
  },
  sections: [{
    widgets: [{
      keyValue: {
        topLabel: "Customer",
        content: opportunity.account.company_name
      }
    }, {
      keyValue: {
        topLabel: "Value",
        content: formatCurrency(opportunity.amount)
      }
    }, {
      buttons: [{
        textButton: {
          text: "VIEW DEAL",
          onClick: { openLink: { url: `https://app.hvn.vn/opportunities/${opportunity.id}` }}
        }
      }]
    }]
  }]
};
```

### 3.4 Google Calendar Integration

```javascript
// Create calendar event for interview
async function createInterviewEvent(interview) {
  const calendar = google.calendar({ version: 'v3', auth: oauth2Client });
  
  const event = {
    summary: `Interview: ${interview.candidate.applicant_name}`,
    description: `Position: ${interview.campaign.position.name}\nCandidate: ${interview.candidate.email}`,
    start: {
      dateTime: interview.scheduled_time,
      timeZone: 'Asia/Ho_Chi_Minh'
    },
    end: {
      dateTime: addMinutes(interview.scheduled_time, 60),
      timeZone: 'Asia/Ho_Chi_Minh'
    },
    attendees: [
      { email: interview.interviewer.work_email },
      { email: interview.candidate.email }
    ],
    conferenceData: {
      createRequest: { requestId: interview.id }
    }
  };
  
  const response = await calendar.events.insert({
    calendarId: 'primary',
    resource: event,
    conferenceDataVersion: 1,
    sendNotifications: true
  });
  
  return response.data;
}
```

---

## 4. WordPress Integration

### 4.1 Lead Capture Flow

```
WordPress Form Submit
        │
        ▼
  Webhook to Directus
        │
        ▼
   Directus Flow:
   • Validate data
   • Create Lead
   • Auto-assign
   • Notify Sales
```

### 4.2 Webhook Handler

```javascript
// Endpoint: POST /wordpress-lead

// Directus Flow configuration
{
  "trigger": "webhook",
  "method": "POST",
  "path": "/wordpress-lead",
  "operations": [
    {
      "name": "create_lead",
      "action": "item.create",
      "collection": "leads",
      "payload": {
        "lead_name": "{{$trigger.body.name}}",
        "company_name": "{{$trigger.body.company}}",
        "email": "{{$trigger.body.email}}",
        "phone": "{{$trigger.body.phone}}",
        "channel": "Website",
        "utm_source": "{{$trigger.body.utm_source}}",
        "utm_medium": "{{$trigger.body.utm_medium}}",
        "utm_campaign": "{{$trigger.body.utm_campaign}}",
        "lead_status": "New"
      }
    },
    {
      "name": "assign_lead",
      "action": "run_script",
      "script": "autoAssignLead"
    },
    {
      "name": "notify_sales",
      "action": "notification",
      "channel": "google_chat",
      "template": "new_lead"
    }
  ]
}
```

---

## 5. Payment Gateway Integration

### 5.1 Supported Gateways

| Gateway | Type | Use Case |
|---------|------|----------|
| **VNPay** | Bank/Card | Primary online payment |
| **Momo** | E-wallet | Consumer payments |
| **SePay** | Bank webhook | Auto bank reconciliation |
| **Manual** | Bank transfer | Enterprise customers |

### 5.2 SePay Webhook (Bank Auto-Reconciliation)

```javascript
// Webhook endpoint: POST /sepay-webhook

// Payload from SePay
{
  "id": "txn_123",
  "amount": 5000000,
  "description": "INV-2026-0001 - ABC Company",
  "bank_account": "1234567890",
  "transaction_date": "2026-01-15T10:30:00Z",
  "reference": "SEPAYVN123456"
}

// Handler
async function processSepayPayment(payload) {
  // Step 1: Extract invoice number from description
  const invoiceMatch = payload.description.match(/INV-\d{4}-\d+/);
  
  if (invoiceMatch) {
    const invoiceNumber = invoiceMatch[0];
    const invoice = await findInvoice({ invoice_number: invoiceNumber });
    
    if (invoice) {
      // Step 2: Create payment record
      const payment = await createPayment({
        invoice: invoice.id,
        account: invoice.account,
        amount: payload.amount,
        payment_date: payload.transaction_date,
        payment_method: 'Bank Transfer',
        reference_number: payload.reference,
        source_system: 'SePay',
        verified: true
      });
      
      // Step 3: Update invoice
      await updateInvoicePaidAmount(invoice);
      
      // Step 4: Create cash flow
      await createCashFlow({
        type: 'IN',
        amount: payload.amount,
        category: 'Customer Payment',
        payment: payment.id,
        account: invoice.account,
        description: `Payment for ${invoiceNumber}`
      });
      
      // Step 5: Notify
      await notifyPaymentReceived(payment);
      
      return { status: 'matched', payment_id: payment.id };
    }
  }
  
  // If no match, create unmatched payment for review
  return { status: 'unmatched', requires_review: true };
}
```

---

## 6. Notification Architecture

### 6.1 Notification Channels

| Channel | Target | Use Case |
|---------|--------|----------|
| Google Chat (Space) | Team | Team-wide alerts |
| Google Chat (DM) | Individual | Personal notifications |
| Email | Customer/Staff | Formal communications |
| In-App | Staff | Dashboard notifications |

### 6.2 Notification Templates

```javascript
// Collection: notification_templates
{
  id: "uuid",
  template_name: "new_lead",
  channel_type: "google_chat",
  subject: "🔔 New Lead: {{lead_name}}",
  body: "New lead from {{channel}}\n\nCompany: {{company_name}}\nPhone: {{phone}}\nEmail: {{email}}",
  card_template: { /* Google Chat card JSON */ },
  active: "boolean"
}
```

### 6.3 Notification Triggers (Directus Flows)

| Event | Notification | Channel |
|-------|--------------|---------|
| Lead created | New Lead Alert | #sales-alerts |
| Lead assigned | Lead Assignment | DM to Sales |
| Opp stage changed | Pipeline Update | #sales-alerts |
| Deal won | Deal Won Celebration | #sales-alerts + Team |
| Invoice overdue | Overdue Alert | DM to AM |
| Case created | New Case | #support-alerts |
| SLA breach | SLA Warning | DM + Manager |
| Leave approved | Leave Notification | DM to Employee |
| Payroll ready | Payslip Ready | DM to Employee |

---

## 7. Error Handling & Monitoring

### 7.1 Integration Logs

```javascript
// Collection: integration_logs
{
  id: "uuid",
  timestamp: "datetime",
  integration: "single_select",  // whmcs, google, wordpress, payment
  event_type: "string",
  direction: "single_select",    // inbound, outbound
  status: "single_select",       // success, error, retry
  request_payload: "json",
  response_payload: "json",
  error_message: "text",
  retry_count: "integer"
}
```

### 7.2 Retry Strategy

| Scenario | Retry | Interval | Max Attempts |
|----------|-------|----------|--------------|
| Network timeout | Yes | Exponential | 5 |
| API rate limit | Yes | Fixed (60s) | 3 |
| Invalid data | No | - | - |
| Auth failure | No | - | - |

### 7.3 Monitoring Alerts

| Alert | Condition | Notification |
|-------|-----------|--------------|
| Sync failure | 3 consecutive errors | #system-alerts |
| High error rate | >10% in 1 hour | DM to Admin |
| WHMCS disconnected | Connection test fails | #system-alerts |

---

*Document Version: 1.0*
*Last Updated: January 2026*
*Author: HVN GROUP*