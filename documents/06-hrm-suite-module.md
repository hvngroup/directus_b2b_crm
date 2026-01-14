# HVN Enterprise Platform - HRM Suite Module

## 1. Module Overview

### 1.1 Purpose

The HRM Suite provides comprehensive human resource management across the entire employee lifecycle, from recruitment to offboarding.

### 1.2 HRM Applications

| Application | Purpose | Key Features |
|-------------|---------|--------------|
| **HVN Account** | Employee self-service portal | Profile, tasks, policies, KPI dashboard |
| **HVN Hiring** | Recruitment management | Campaigns, applications, interviews, tests |
| **HVN Onboard** | New employee onboarding | Checklists, documentation, training |
| **HVN LMS** | Learning management | Courses, quizzes, certifications |
| **HVN Checkin** | Time & attendance | Clock in/out, GPS, reports |
| **HVN Timeoff** | Leave management | Requests, approvals, balances |
| **HVN Payroll** | Salary processing | Calculations, payslips, tax |
| **HVN Case** | Violation records | Incidents, disciplinary actions |
| **HVN Reward** | Recognition & rewards | Achievements, bonuses |
| **HVN Assets** | Asset management | Assignment, tracking |
| **HVN KPIs** | Performance management | Goals, reviews, scores |
| **HVN HRM** | HR administration | Master data, org structure |

---

## 2. Employee Lifecycle

```
CANDIDATE ──▶ HIRED ──▶ ONBOARDING ──▶ ACTIVE ──▶ DEVELOP ──▶ EXIT
    │           │           │            │           │         │
    ▼           ▼           ▼            ▼           ▼         ▼
[Hiring]   [Onboard]   [Checkin]    [Timeoff]    [LMS]    [Offboard]
[Tests]    [Assets]    [KPIs]       [Payroll]    [Reward]  [Exit Int]
           [Account]   [Case]       [Assets]     [KPIs]    [Clearance]
```

---

## 3. Core Data Model

### 3.1 Employee Master

```javascript
// Collection: employees
{
  id: "uuid",
  status: "published",
  
  // Personal Info
  employee_code: "string",       // Auto: HVN-XXXX
  full_name: "string",
  date_of_birth: "date",
  gender: "single_select",
  phone: "string",
  personal_email: "string",
  
  // Work Info
  work_email: "string",
  department: "m2o:departments",
  position: "m2o:positions",
  team: "m2o:teams",
  manager: "m2o:employees",
  
  // Employment
  employment_type: "single_select",  // Full-time, Part-time, Contract, Intern
  hire_date: "date",
  probation_end_date: "date",
  employment_status: "single_select", // Active, On Leave, Resigned, Terminated
  
  // Compensation
  base_salary: "decimal",
  currency: "string",
  bank_account: "string",
  bank_name: "string",
  tax_code: "string",
  
  // Leave
  annual_leave_balance: "decimal",
  sick_leave_balance: "decimal",
  
  // System User Link
  directus_user: "m2o:directus_users",
  
  // Documents
  id_card_number: "string",
  id_card_date: "date",
  id_card_place: "string",
  avatar: "file",
  
  // Emergency Contact
  emergency_contact_name: "string",
  emergency_contact_phone: "string",
  emergency_contact_relationship: "string",
  
  // Relations (O2M)
  contracts: "o2m:hr_contracts",
  attendance_records: "o2m:attendance_records",
  timeoff_requests: "o2m:timeoff_requests",
  payroll_records: "o2m:payroll_records",
  kpi_records: "o2m:kpi_records",
  case_records: "o2m:case_records",
  reward_records: "o2m:reward_records",
  assigned_assets: "o2m:asset_assignments",
  enrollments: "o2m:lms_enrollments",
  
  // System
  user_created: "user",
  date_created: "datetime"
}
```

---

## 4. HVN Hiring Module

### 4.1 Data Structure

```javascript
// Collection: hiring_campaigns
{
  id: "uuid",
  campaign_name: "string",
  position: "m2o:positions",
  department: "m2o:departments",
  quantity: "integer",
  status: "single_select",       // Draft, Active, Closed
  budget: "decimal",
  deadline: "date",
  hiring_manager: "m2o:employees",
  applications: "o2m:job_applications"
}

// Collection: job_applications
{
  id: "uuid",
  campaign: "m2o:hiring_campaigns",
  
  // Applicant Info
  applicant_name: "string",
  email: "string",
  phone: "string",
  date_of_birth: "date",
  
  // Application
  cv_file: "file",
  cover_letter: "text",
  source: "m2o:recruitment_sources",
  referrer: "m2o:employees",
  
  // Pipeline Stage
  stage: "single_select",        // Applied, Screening, Test, Interview, Offer, Hired, Rejected
  
  // AI Analysis
  ai_cv_analysis: "json",
  ai_score: "decimal",
  
  // Evaluations
  evaluations: "o2m:interview_evaluations",
  tests: "o2m:candidate_tests",
  
  // Offer
  offered_salary: "decimal",
  offer_status: "single_select", // Pending, Accepted, Rejected
  expected_start_date: "date",
  
  // If Hired
  hired_employee: "m2o:employees"
}
```

### 4.2 Hiring Pipeline

```
APPLIED ──▶ SCREENING ──▶ TEST ──▶ INTERVIEW ──▶ OFFER ──▶ HIRED
                                      │
                                      └──▶ REJECTED (at any stage)
```

### 4.3 Interview Scheduling (Google Calendar)

| Trigger | Action |
|---------|--------|
| Interview scheduled | Create Google Calendar event |
| Interview rescheduled | Update Google Calendar event |
| Interview cancelled | Delete Google Calendar event |

---

## 5. HVN LMS + Test Center

### 5.1 Course Structure

```javascript
// Collection: lms_courses
{
  id: "uuid",
  title: "string",
  description: "text",
  learning_objectives: "text",
  
  // Classification
  category: "m2o:course_categories",
  difficulty: "single_select",   // Beginner, Intermediate, Advanced
  
  // Access
  department_filter: "json",     // [] = public, ["it","sales"] = restricted
  
  // Content
  modules: "o2m:lms_modules",
  
  // Duration
  estimated_duration: "integer", // Minutes
  
  // Status
  status: "single_select",       // Draft, Published, Archived
  
  // Image
  thumbnail: "file"
}

// Collection: lms_modules
{
  id: "uuid",
  course: "m2o:lms_courses",
  title: "string",
  sort: "integer",
  lessons: "o2m:lms_lessons"
}

// Collection: lms_lessons
{
  id: "uuid",
  module: "m2o:lms_modules",
  title: "string",
  type: "single_select",         // video, article, file, quiz
  
  // Content by type
  video_url: "string",
  video_provider: "single_select",  // youtube, vimeo, direct
  content: "text",               // For article
  file: "file",                  // For file
  quiz: "m2o:lms_quizzes",       // For quiz
  
  duration: "integer",           // Minutes
  is_required: "boolean",
  sort: "integer"
}
```

### 5.2 Quiz & Assessment

```javascript
// Collection: lms_quizzes
{
  id: "uuid",
  title: "string",
  pass_score: "integer",         // 0-100
  time_limit: "integer",         // Minutes (null = unlimited)
  max_attempts: "integer",
  randomize: "boolean",
  questions: "o2m:lms_questions"
}

// Collection: lms_questions
{
  id: "uuid",
  quiz: "m2o:lms_quizzes",
  question_text: "text",
  type: "single_select",         // single, multiple, text
  options: "json",               // {A: "...", B: "...", correct: ["A"]}
  points: "integer",
  sort: "integer"
}
```

### 5.3 Enrollment & Progress

```javascript
// Collection: lms_enrollments
{
  id: "uuid",
  employee: "m2o:employees",
  course: "m2o:lms_courses",
  
  // Assignment
  assigned_by: "m2o:employees",
  due_date: "date",
  
  // Progress
  status: "single_select",       // Assigned, In Progress, Completed, Expired
  progress_percent: "integer",   // 0-100
  started_at: "datetime",
  completed_at: "datetime",
  
  // Lesson Progress
  lesson_progress: "o2m:lms_lesson_progress",
  
  // Quiz Attempts
  quiz_attempts: "o2m:lms_quiz_attempts"
}
```

---

## 6. HVN Checkin (Time & Attendance)

### 6.1 Check-in Methods

| Method | Use Case |
|--------|----------|
| **GPS** | Mobile app with location |
| **WiFi** | Office network detection |
| **QR Code** | Scan at office entrance |
| **Manual** | Admin override |

### 6.2 Attendance Record

```javascript
// Collection: attendance_records
{
  id: "uuid",
  employee: "m2o:employees",
  date: "date",
  
  // Clock In
  check_in_time: "datetime",
  check_in_method: "single_select",  // GPS, WiFi, QR, Manual
  check_in_location: "geometry",     // GPS coordinates
  check_in_photo: "file",
  
  // Clock Out
  check_out_time: "datetime",
  check_out_method: "single_select",
  check_out_location: "geometry",
  
  // Calculations
  work_hours: "decimal",             // Calculated
  overtime_hours: "decimal",
  late_minutes: "integer",
  early_leave_minutes: "integer",
  
  // Status
  status: "single_select",           // Present, Late, Absent, Leave, Holiday
  
  // Notes
  notes: "text",
  adjusted_by: "m2o:employees"
}
```

### 6.3 Automation

| Rule | Trigger | Action |
|------|---------|--------|
| ATT-001 | check_out_time set | Calculate work_hours, overtime |
| ATT-002 | check_in_time > 09:00 | late_minutes = diff, status = Late |
| ATT-003 | Scheduled (end of day) | Mark absent for no check-in |
| ATT-004 | End of month | Generate attendance summary |

---

## 7. HVN Timeoff (Leave Management)

### 7.1 Leave Types

| Type | Annual Entitlement | Carries Over |
|------|-------------------|--------------|
| Annual Leave | 12 days | Yes (max 5) |
| Sick Leave | 30 days | No |
| Personal Leave | 3 days | No |
| Maternity | 180 days | N/A |
| Paternity | 5 days | N/A |
| Unpaid Leave | Unlimited | N/A |

### 7.2 Leave Request

```javascript
// Collection: timeoff_requests
{
  id: "uuid",
  employee: "m2o:employees",
  
  // Request Details
  leave_type: "m2o:leave_types",
  start_date: "date",
  end_date: "date",
  total_days: "decimal",         // Calculated
  reason: "text",
  
  // Status
  status: "single_select",       // Pending, Approved, Rejected, Cancelled
  
  // Approval
  approved_by: "m2o:employees",
  approved_date: "datetime",
  rejection_reason: "text",
  
  // Attachments (for sick leave)
  attachments: "files",
  
  // System
  user_created: "user",
  date_created: "datetime"
}
```

### 7.3 Approval Flow

```
Employee submits ──▶ Manager review ──▶ HR notification ──▶ Calendar update
                          │
                    ┌─────┴─────┐
                    ▼           ▼
               APPROVED     REJECTED
                    │           │
                    ▼           ▼
            Update balance   Notify employee
            Block calendar
```

---

## 8. HVN Payroll

### 8.1 Payroll Components

| Category | Components |
|----------|------------|
| **Earnings** | Base Salary, Overtime, Allowances, Bonus, Commission |
| **Deductions** | Social Insurance, Health Insurance, Unemployment, PIT, Advances |

### 8.2 Payroll Record

```javascript
// Collection: payroll_records
{
  id: "uuid",
  employee: "m2o:employees",
  period: "string",              // YYYY-MM
  
  // Work Data
  work_days: "decimal",
  overtime_hours: "decimal",
  late_count: "integer",
  absence_count: "integer",
  
  // Earnings
  base_salary: "decimal",
  overtime_pay: "decimal",
  allowances: "decimal",
  bonus: "decimal",
  commission: "decimal",
  gross_income: "formula",
  
  // Deductions
  social_insurance: "decimal",   // 8%
  health_insurance: "decimal",   // 1.5%
  unemployment: "decimal",       // 1%
  personal_income_tax: "decimal",
  advances: "decimal",
  other_deductions: "decimal",
  total_deductions: "formula",
  
  // Net
  net_salary: "formula",
  
  // Status
  status: "single_select",       // Draft, Approved, Paid
  paid_date: "date",
  
  // System
  calculated_by: "m2o:employees",
  calculated_date: "datetime",
  approved_by: "m2o:employees"
}
```

### 8.3 PIT Calculation (Vietnam)

```
Taxable Income = Gross - Social Insurance - Dependents (11M each)

Tax Brackets:
  ≤ 5M:        5%
  5M - 10M:    10%
  10M - 18M:   15%
  18M - 32M:   20%
  32M - 52M:   25%
  52M - 80M:   30%
  > 80M:       35%
```

---

## 9. HVN Case & HVN Reward

### 9.1 Case (Violation) Record

```javascript
// Collection: case_records
{
  id: "uuid",
  employee: "m2o:employees",
  
  // Incident
  case_date: "date",
  case_type: "m2o:case_types",   // Late, Absent, Policy Violation, etc.
  description: "text",
  
  // Evidence
  attachments: "files",
  
  // Action
  action_taken: "single_select", // Warning, Written Warning, Suspension, Termination
  penalty_amount: "decimal",     // If financial penalty
  
  // Approval
  reported_by: "m2o:employees",
  approved_by: "m2o:employees",
  
  // Status
  status: "single_select",       // Pending, Confirmed, Appealed, Resolved
  
  // Link to Payroll
  payroll_deducted: "boolean"
}
```

### 9.2 Reward Record

```javascript
// Collection: reward_records
{
  id: "uuid",
  employee: "m2o:employees",
  
  // Achievement
  reward_date: "date",
  reward_type: "m2o:reward_types", // Performance, Innovation, Sales, etc.
  description: "text",
  
  // Reward
  reward_value: "decimal",       // Bonus amount
  reward_item: "string",         // Non-cash reward
  
  // Recognition
  nominated_by: "m2o:employees",
  approved_by: "m2o:employees",
  
  // Status
  status: "single_select",       // Pending, Approved, Paid/Given
  
  // Link to Payroll
  payroll_added: "boolean"
}
```

---

## 10. HVN KPIs

### 10.1 KPI Structure

```javascript
// Collection: kpi_records
{
  id: "uuid",
  employee: "m2o:employees",
  
  // Period
  period: "single_select",       // Monthly, Quarterly, Yearly
  period_value: "string",        // 2026-Q1, 2026-01
  
  // Targets
  target_revenue: "decimal",
  target_deals: "integer",
  target_activities: "integer",
  
  // Actuals (can be auto-calculated)
  actual_revenue: "decimal",
  actual_deals: "integer",
  actual_activities: "integer",
  
  // Score
  achievement_rate: "formula",   // actual/target × 100
  overall_score: "decimal",
  
  // Review
  self_assessment: "text",
  manager_feedback: "text",
  reviewed_by: "m2o:employees",
  review_date: "date",
  
  // Status
  status: "single_select"        // Draft, Submitted, Reviewed, Finalized
}
```

### 10.2 KPI Categories by Role

| Role | KPIs |
|------|------|
| **Sales Rep** | Revenue, Deals won, Leads converted, Activities |
| **Account Manager** | Revenue retained, CSAT, Renewals, Upsell |
| **Support Agent** | Cases resolved, CSAT, Response time, SLA compliance |
| **Developer** | Tasks completed, Bug fixes, Code quality, On-time delivery |

---

## 11. HVN Assets

### 11.1 Asset Assignment

```javascript
// Collection: asset_assignments
{
  id: "uuid",
  asset: "m2o:assets",
  employee: "m2o:employees",
  
  // Assignment
  assigned_date: "date",
  expected_return_date: "date",
  actual_return_date: "date",
  
  // Status
  status: "single_select",       // Assigned, Returned, Lost, Damaged
  
  // Condition
  condition_at_assign: "single_select",
  condition_at_return: "single_select",
  
  // Notes
  notes: "text",
  
  // Signatures
  assigned_by: "m2o:employees",
  received_signature: "file"
}
```

---

## 12. Google Chat Notifications

| Event | Channel | Message |
|-------|---------|---------|
| New leave request | Direct to Manager | "📅 Leave request from {name}: {dates}" |
| Leave approved | Direct to Employee | "✅ Your leave request has been approved" |
| Payroll ready | Direct to Employee | "💰 Your payslip for {month} is ready" |
| New case recorded | Direct to Employee + HR | "⚠️ Case recorded for {employee}" |
| Training due | Direct to Employee | "📚 Training due: {course} by {date}" |
| KPI review pending | Direct to Manager | "📊 KPI review pending for {employee}" |

---

*Document Version: 1.0*
*Last Updated: January 2026*
*Author: HVN GROUP*