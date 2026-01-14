# HVN Enterprise Platform - Technical Infrastructure

## 1. Deployment Architecture

### 1.1 Cloud Platform: Google Cloud Platform (GCP)

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                           GCP DEPLOYMENT ARCHITECTURE                                │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                      │
│  ┌─────────────────────────────────────────────────────────────────────────────┐   │
│  │                        CLOUD LOAD BALANCER                                   │   │
│  │                    (HTTPS, SSL Termination, CDN)                             │   │
│  └──────────────────────────────────┬──────────────────────────────────────────┘   │
│                                     │                                               │
│           ┌─────────────────────────┼─────────────────────────┐                    │
│           │                         │                         │                    │
│           ▼                         ▼                         ▼                    │
│  ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐              │
│  │   FRONTEND      │     │   DIRECTUS      │     │   WORDPRESS     │              │
│  │   Cloud Run     │     │   Cloud Run     │     │   Cloud Run     │              │
│  │                 │     │                 │     │   (or GCE)      │              │
│  │  app.hvn.vn     │     │  api.hvn.vn     │     │  www.hvn.vn     │              │
│  │                 │     │                 │     │                 │              │
│  │  Auto-scale     │     │  Auto-scale     │     │  Auto-scale     │              │
│  │  0-10 instances │     │  1-20 instances │     │  1-5 instances  │              │
│  └────────┬────────┘     └────────┬────────┘     └────────┬────────┘              │
│           │                       │                       │                        │
│           └───────────────────────┼───────────────────────┘                        │
│                                   │                                                │
│  ┌────────────────────────────────┼────────────────────────────────────────────┐  │
│  │                          VPC NETWORK                                         │  │
│  │                                                                              │  │
│  │  ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐        │  │
│  │  │   Cloud SQL     │     │  Cloud Storage  │     │   Memorystore   │        │  │
│  │  │   PostgreSQL    │     │    (Files)      │     │     Redis       │        │  │
│  │  │                 │     │                 │     │                 │        │  │
│  │  │  HA: Regional   │     │  Standard       │     │  Cache Layer    │        │  │
│  │  │  Backup: Daily  │     │  CDN enabled    │     │  Session Store  │        │  │
│  │  │  Storage: SSD   │     │                 │     │                 │        │  │
│  │  └─────────────────┘     └─────────────────┘     └─────────────────┘        │  │
│  │                                                                              │  │
│  └──────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                      │
│  ┌──────────────────────────────────────────────────────────────────────────────┐  │
│  │                        EXTERNAL CONNECTIONS                                   │  │
│  │                                                                               │  │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐            │  │
│  │  │  WHMCS  │  │ Google  │  │ Payment │  │  Email  │  │   SMS   │            │  │
│  │  │ Servers │  │ APIs    │  │ Gateway │  │ Service │  │ Service │            │  │
│  │  │ (VPS)   │  │         │  │         │  │         │  │         │            │  │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘  └─────────┘            │  │
│  └──────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                      │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Service Specifications

| Service | Type | Specs | Scaling |
|---------|------|-------|---------|
| Frontend | Cloud Run | 1 vCPU, 512MB | 0-10 instances |
| Directus API | Cloud Run | 2 vCPU, 2GB | 1-20 instances |
| PostgreSQL | Cloud SQL | db-custom-2-4096 | HA Regional |
| Redis | Memorystore | 1GB Basic | Single instance |
| Storage | Cloud Storage | Standard | Auto |
| CDN | Cloud CDN | Global | Auto |

### 1.3 Environment Configuration

```bash
# Production Environment Variables

# Directus Core
ADMIN_EMAIL=admin@hvn.vn
ADMIN_PASSWORD=***
SECRET=***
KEY=***

# Database
DB_CLIENT=pg
DB_HOST=/cloudsql/project:region:instance
DB_PORT=5432
DB_DATABASE=directus
DB_USER=directus
DB_PASSWORD=***

# Storage
STORAGE_LOCATIONS=gcs
STORAGE_GCS_DRIVER=gcs
STORAGE_GCS_BUCKET=hvn-directus-files
STORAGE_GCS_KEY_FILENAME=/secrets/gcs-key.json

# Cache
CACHE_ENABLED=true
CACHE_STORE=redis
REDIS_HOST=10.0.0.x
REDIS_PORT=6379

# Authentication
AUTH_PROVIDERS=google
AUTH_GOOGLE_DRIVER=oauth2
AUTH_GOOGLE_CLIENT_ID=***
AUTH_GOOGLE_CLIENT_SECRET=***

# Rate Limiting
RATE_LIMITER_ENABLED=true
RATE_LIMITER_STORE=redis
RATE_LIMITER_POINTS=100
RATE_LIMITER_DURATION=60
```

---

## 2. Security Architecture

### 2.1 Authentication Flow

```
User Request
      │
      ▼
┌──────────────┐
│   Frontend   │
│   (React)    │
└──────┬───────┘
       │ (1) Redirect to Google OAuth
       ▼
┌──────────────┐
│    Google    │
│    OAuth     │
└──────┬───────┘
       │ (2) Return auth code
       ▼
┌──────────────┐
│   Directus   │
│     API      │
└──────┬───────┘
       │ (3) Exchange code for tokens
       │ (4) Create/Update user
       │ (5) Return JWT
       ▼
┌──────────────┐
│   Frontend   │
│   (Stores    │
│    JWT)      │
└──────────────┘
```

### 2.2 Role Hierarchy

```
┌─────────────────────────────────────────────────────────────────────┐
│                         ROLE HIERARCHY                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│                         SUPER ADMIN                                 │
│                    (Full system access)                             │
│                             │                                       │
│           ┌─────────────────┼─────────────────┐                    │
│           │                 │                 │                    │
│           ▼                 ▼                 ▼                    │
│     ┌───────────┐    ┌───────────┐    ┌───────────┐               │
│     │ CRM ADMIN │    │ HR ADMIN  │    │  FINANCE  │               │
│     │           │    │           │    │   ADMIN   │               │
│     └─────┬─────┘    └─────┬─────┘    └─────┬─────┘               │
│           │                │                │                      │
│     ┌─────┴─────┐    ┌─────┴─────┐    ┌─────┴─────┐               │
│     │Sales Mgr  │    │HR Manager │    │Finance Mgr│               │
│     │Account Mgr│    │HR Staff   │    │Accountant │               │
│     │Sales Rep  │    │           │    │           │               │
│     └───────────┘    └───────────┘    └───────────┘               │
│                                                                     │
│     ┌───────────────────────────────────────────────────┐          │
│     │                  ALL EMPLOYEES                     │          │
│     │            (Self-service via HVN Account)          │          │
│     └───────────────────────────────────────────────────┘          │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.3 Permission Matrix

| Collection | Sales Rep | Sales Mgr | Finance | HR | Admin |
|------------|-----------|-----------|---------|-----|-------|
| leads | Own CRUD | Team CRUD | Read | - | Full |
| accounts | Read | Team CRUD | Read | - | Full |
| opportunities | Own CRUD | Team CRUD | Read | - | Full |
| contracts | Read own | Read team | Full | - | Full |
| invoices | - | - | Full | - | Full |
| employees | Self | Team read | Read | Full | Full |
| attendance | Self | Team read | - | Full | Full |
| payroll | Self read | - | Read | Full | Full |

### 2.4 Data Security

| Layer | Implementation |
|-------|----------------|
| **Transport** | TLS 1.3, HTTPS only |
| **At Rest** | Cloud SQL encryption, GCS encryption |
| **Application** | Field-level ACL, Row-level filters |
| **Secrets** | Secret Manager, encrypted env vars |
| **Audit** | All changes logged with user/timestamp |

### 2.5 API Security

```javascript
// Rate Limiting Configuration
{
  "rateLimiter": {
    "enabled": true,
    "points": 100,      // Requests
    "duration": 60,     // Per minute
    "keyPrefix": "rl:",
    "whitelist": ["internal-service-key"]
  }
}

// CORS Configuration
{
  "cors": {
    "origin": [
      "https://app.hvn.vn",
      "https://www.hvn.vn"
    ],
    "methods": ["GET", "POST", "PATCH", "DELETE"],
    "credentials": true
  }
}
```

---

## 3. Performance Optimization

### 3.1 Caching Strategy

| Cache Type | Implementation | TTL | Use Case |
|------------|----------------|-----|----------|
| API Response | Redis | 5 min | Read-heavy endpoints |
| Session | Redis | 24 hours | User sessions |
| Static Assets | Cloud CDN | 1 year | JS, CSS, images |
| Database Query | Directus built-in | 5 min | Aggregations |

### 3.2 Database Optimization

```sql
-- Key Indexes

-- Leads
CREATE INDEX idx_leads_status ON leads(lead_status);
CREATE INDEX idx_leads_assigned ON leads(assigned_to);
CREATE INDEX idx_leads_created ON leads(date_created);

-- Opportunities
CREATE INDEX idx_opp_stage ON opportunities(stage);
CREATE INDEX idx_opp_owner ON opportunities(owner);
CREATE INDEX idx_opp_close_date ON opportunities(close_date);
CREATE INDEX idx_opp_account ON opportunities(account);

-- Invoices
CREATE INDEX idx_invoice_status ON invoices(invoice_status);
CREATE INDEX idx_invoice_due ON invoices(due_date);
CREATE INDEX idx_invoice_account ON invoices(account);

-- Attendance
CREATE INDEX idx_attendance_employee_date ON attendance_records(employee, date);

-- Support Cases
CREATE INDEX idx_case_status ON support_cases(case_status);
CREATE INDEX idx_case_assigned ON support_cases(assigned_to);
CREATE INDEX idx_case_sla ON support_cases(first_response_due);
```

### 3.3 Cloud Run Optimization

```yaml
# Cloud Run Service Configuration

apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: directus-api
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/minScale: "1"
        autoscaling.knative.dev/maxScale: "20"
        run.googleapis.com/cpu-throttling: "false"
        run.googleapis.com/startup-cpu-boost: "true"
    spec:
      containerConcurrency: 80
      timeoutSeconds: 300
      containers:
        - image: gcr.io/hvn-project/directus:latest
          resources:
            limits:
              cpu: "2"
              memory: "2Gi"
```

---

## 4. Monitoring & Logging

### 4.1 Monitoring Stack

| Tool | Purpose | Metrics |
|------|---------|---------|
| Cloud Monitoring | Infrastructure | CPU, Memory, Latency |
| Cloud Logging | Application logs | Errors, requests |
| Uptime Checks | Availability | Endpoint health |
| Error Reporting | Exceptions | Stack traces |

### 4.2 Key Metrics

| Metric | Target | Alert Threshold |
|--------|--------|-----------------|
| API Latency (p95) | < 500ms | > 1000ms |
| Error Rate | < 1% | > 5% |
| Database CPU | < 70% | > 80% |
| Memory Usage | < 80% | > 90% |
| Uptime | 99.9% | < 99.5% |

### 4.3 Alert Configuration

```yaml
# Cloud Monitoring Alert Policy

displayName: "High API Latency"
conditions:
  - displayName: "Latency > 1s"
    conditionThreshold:
      filter: 'resource.type="cloud_run_revision" AND metric.type="run.googleapis.com/request_latencies"'
      comparison: COMPARISON_GT
      thresholdValue: 1000
      duration: 300s
      aggregations:
        - alignmentPeriod: 60s
          perSeriesAligner: ALIGN_PERCENTILE_95
notificationChannels:
  - projects/hvn-project/notificationChannels/gchat-alerts
```

### 4.4 Logging Structure

```javascript
// Structured Log Format

{
  "timestamp": "2026-01-15T10:30:00Z",
  "severity": "INFO",
  "message": "API Request",
  "context": {
    "request_id": "abc123",
    "method": "POST",
    "path": "/items/leads",
    "user_id": "user-456",
    "duration_ms": 150,
    "status_code": 200
  },
  "labels": {
    "environment": "production",
    "service": "directus-api"
  }
}
```

---

## 5. Backup & Disaster Recovery

### 5.1 Backup Strategy

| Data | Method | Frequency | Retention |
|------|--------|-----------|-----------|
| PostgreSQL | Automated backup | Daily | 30 days |
| PostgreSQL | Point-in-time recovery | Continuous | 7 days |
| Cloud Storage | Object versioning | On change | 30 versions |
| Config/Code | Git repository | On change | Unlimited |

### 5.2 Recovery Procedures

| Scenario | RTO | RPO | Procedure |
|----------|-----|-----|-----------|
| Database failure | 1 hour | 0 | Failover to standby |
| Data corruption | 4 hours | 24 hours | Restore from backup |
| Region outage | 24 hours | 1 hour | Deploy to backup region |
| Full disaster | 48 hours | 24 hours | Full rebuild from backups |

### 5.3 Disaster Recovery Checklist

```markdown
## DR Checklist

### Database Recovery
- [ ] Identify failure point
- [ ] Determine RPO requirement
- [ ] Restore from appropriate backup
- [ ] Verify data integrity
- [ ] Update connection strings
- [ ] Test application functionality

### Application Recovery
- [ ] Deploy new Cloud Run instances
- [ ] Update load balancer
- [ ] Verify integrations (WHMCS, Google)
- [ ] Test all critical workflows
- [ ] Notify stakeholders
```

---

## 6. DevOps & CI/CD

### 6.1 CI/CD Pipeline

```yaml
# Cloud Build configuration

steps:
  # Build Frontend
  - name: 'node:20'
    dir: 'frontend'
    entrypoint: 'npm'
    args: ['ci']
  
  - name: 'node:20'
    dir: 'frontend'
    entrypoint: 'npm'
    args: ['run', 'build']
  
  # Build Docker Image
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'gcr.io/$PROJECT_ID/directus:$COMMIT_SHA', '.']
  
  # Push to Registry
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'gcr.io/$PROJECT_ID/directus:$COMMIT_SHA']
  
  # Deploy to Cloud Run
  - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
    entrypoint: 'gcloud'
    args:
      - 'run'
      - 'deploy'
      - 'directus-api'
      - '--image=gcr.io/$PROJECT_ID/directus:$COMMIT_SHA'
      - '--region=asia-southeast1'
      - '--platform=managed'

timeout: '1200s'
```

### 6.2 Environment Promotion

```
Development ──▶ Staging ──▶ Production
     │              │            │
     │              │            │
   Direct        Manual       Approval
   Deploy        Deploy       Required
```

---

*Document Version: 1.0*
*Last Updated: January 2026*
*Author: HVN GROUP*