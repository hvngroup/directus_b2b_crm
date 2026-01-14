# Triển khai CRM Sales B2B trên Lark

## Mục lục
- 1. Tổng quan Giải pháp
- 2. Kiến trúc Hệ thống
- 3. Thiết kế Database
- 4. Quy trình Phê duyệt (Approval)
- 5. Vai trò & Phân quyền
- 6. User Journey Map
- 7. Thiết kế Dashboard
- 8. Automation & Business Rules
- 9. Tích hợp Lark Components
- 10. Kế hoạch Triển khai
- 11. Vận hành & Cải tiến
- Phụ lục

---

## 1. Tổng quan Giải pháp

### 1.1. Mục tiêu

Xây dựng hệ thống CRM B2B toàn diện trên Lark, bao gồm:

| Mục tiêu | Mô tả |
|----------|-------|
| Quản lý Sales Pipeline | Từ Lead đến Close Deal |
| Quản lý Khách hàng | Account, Contact, lịch sử tương tác |
| Quản lý Hợp đồng | Contract lifecycle, renewal tracking |
| Quản lý Tài chính | Invoice, Payment, Cash Flow |
| Quản lý Nhân sự Sales | Team, Staff, KPI |
| Báo cáo & Phân tích | Dashboard real-time |

### 1.2. Phạm vi Chức năng

```
                    CRM B2B VALUE CHAIN

  MARKETING      SALES        DELIVERY      FINANCE       SUCCESS
  ─────────      ─────        ────────      ───────       ───────
  • Lead         • Qualify    • Contract    • Invoice     • Support
  • Capture      • Nurture    • Onboard     • Payment     • Renew
  • Channel      • Propose    • Deliver     • Expense     • Upsell
  • Tracking     • Close      • Handover    • Cash Flow
```

### 1.3. Nguyên tắc Thiết kế

| Nguyên tắc | Áp dụng |
|------------|---------|
| Single Source of Truth | Lark Base là database trung tâm |
| Role-based Access | Mỗi vai trò chỉ thấy dữ liệu liên quan |
| Automation First | Giảm thiểu thao tác thủ công |
| Human Decision Point | Approval cho quyết định cần người duyệt |
| Real-time Communication | Thông báo qua Messenger/Bot |

---

## 2. Kiến trúc Hệ thống

### 2.1. Kiến trúc Tổng thể

```
                    LARK BASE APP
                    (Presentation)
                    
                    • Sales Portal
                    • Finance Portal
                    • Management Portal
                    
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
    ┌──────────┐    ┌──────────────┐   ┌──────────────┐
    │LARK      │    │BASE         │   │ANYCROSS      │
    │APPROVAL  │    │AUTOMATION   │   │(Integration) │
    │(Process) │    │(Rules)      │   │              │
    ├──────────┤    ├──────────────┤   ├──────────────┤
    │• Discount│    │• Auto-assign│   │• ERP Sync    │
    │• Payment │    │• Notification│   │• Bank API    │
    │• Expense │    │• Calculation │   │• External    │
    │• Transfer│    │• Status Flow │   │  Systems     │
    └──────────┘    └──────────────┘   └──────────────┘
          │                │                │
          └────────────────┼────────────────┘
                           ▼
                    ┌──────────────┐
                    │  LARK BASE   │
                    │  (Database)  │
                    ├──────────────┤
                    │  12 Tables   │
                    │  Relationships│
                    │  Advanced    │
                    │  Permissions │
                    └──────────────┘
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
    ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
    │  LARK CHAT   │ │LARK CALENDAR │ │   LARK BOT   │
    │(Communication)│ │ (Scheduling) │ │   (Alerts)   │
    └──────────────┘ └──────────────┘ └──────────────┘
```

### 2.2. Phân tầng Chức năng

| Tầng | Components | Vai trò | Khi nào dùng |
|------|------------|---------|--------------|
| Presentation | Base App | UI/UX cho người dùng | Mọi tương tác hàng ngày |
| Process | Approval | Quy trình phê duyệt | Quyết định cần người duyệt |
| Rules | Base Automation/Workflow | Logic tự động | Không cần can thiệp người |
| Data | Base | Lưu trữ dữ liệu | Single source of truth |
| Integration | AnyCross | Kết nối bên ngoài | ERP, Bank, 3rd party |
| Communication | Chat + Bot | Thông báo real-time | Alert, reminder, notify |
| Scheduling | Calendar | Lịch hẹn, follow-up | Meeting, task reminder |

### 2.3. Tư duy Phân chia

**Câu hỏi quyết định:**

```
Cần xử lý gì?

├── Cần người quyết định? ──────────────── APPROVAL
│   (Duyệt giảm giá, duyệt chi, ...)
│
├── Tự động được? ──────────────────────── BASE AUTOMATION
│   (Tính toán, gán người, notify, ...)
│
├── Logic phức tạp, nhiều điều kiện? ───── BASE WORKFLOW
│   (IF/else, nhiều nhánh, ...)
│
├── Cần kết nối hệ thống ngoài? ────────── ANYCROSS
│   (ERP, Bank, External API, ...)
│
├── Cần thông báo ngay? ────────────────── CHAT/BOT
│   (Alert, reminder, ...)
│
└── Cần lưu/truy xuất dữ liệu? ─────────── BASE
    (CRUD operations)
```

---

## 3. Thiết kế Database

### 3.1. Entity Relationship Diagram

```
                    DATABASE ARCHITECTURE


     ┌────────┐      ┌────────┐      ┌────────┐
     │ LEADS  │──────│ACCOUNTS│──────│CONTACTS│
     └────────┘      └────────┘      └────────┘
          │               │
          ▼               ▼
     ┌────────┐      ┌────────────┐
     │ STAFF  │──────│OPPORTUNITIES│
     └────────┘      └────────────┘
          │               │
          ▼               ▼
     ┌────────┐      ┌────────┐      ┌────────┐
     │ TEAMS  │      │CONTRACTS│─────│INVOICES│
     └────────┘      └────────┘      └────────┘
          │                              │
          ▼                              ▼
     ┌────────┐                     ┌────────┐
     │  KPIs  │                     │PAYMENTS│
     └────────┘                     └────────┘
                                         │
                                         ▼
     ┌────────┐                     ┌──────────┐
     │EXPENSES│─────────────────────│CASH FLOW │
     └────────┘                     └──────────┘

     ┌────────┐
     │PRODUCTS│──────── (Linked to Opportunities, Contracts)
     └────────┘
```

### 3.2. Chi tiết Các Bảng

#### 3.2.1. LEADS (Khách hàng tiềm năng)

| Field | Type | Mô tả | Validation |
|-------|------|-------|------------|
| lead_id | Auto Number | ID tự động | PK |
| lead_name | Text | Tên người liên hệ | Required |
| company_name | Text | Tên công ty | Required |
| phone | Phone | Số điện thoại | Format check |
| email | Email | Email | Format check |
| channel | Single Select | Nguồn lead | Xem danh sách bên dưới |
| status | Single Select | Trạng thái | New/Contacted/Qualified/Converted/Lost |
| score | Number | Điểm đánh giá | 0-100 |
| assigned_to | Person | Sales phụ trách | Link to Staff |
| notes | Long Text | Ghi chú | - |
| created_date | Created Time | Ngày tạo | Auto |
| last_activity | Modified Time | Hoạt động cuối | Auto |

**Channel Options:**
- Website - Form trên website
- Facebook Ads - Quảng cáo Facebook
- Google Ads - Quảng cáo Google
- Zalo Ads - Quảng cáo Zalo
- Referral - Giới thiệu
- Event - Hội thảo, triển lãm
- Cold Call - Gọi điện trực tiếp
- Partner - Đối tác giới thiệu
- Inbound - Khách hàng tự liên hệ

#### 3.2.2. ACCOUNTS (Công ty khách hàng)

| Field | Type | Mô tả | Validation |
|-------|------|-------|------------|
| account_id | Auto Number | ID tự động | PK |
| company_name | Text | Tên công ty | Required, Unique |
| tax_code | Text | Mã số thuế | Format check |
| industry | Single Select | Ngành nghề | - |
| company_size | Single Select | Quy mô | Micro/SME/Enterprise |
| address | Text | Địa chỉ | - |
| city | Single Select | Thành phố | - |
| website | URL | Website | - |
| account_owner | Person | AM phụ trách | Link to Staff |
| account_type | Single Select | Loại | Prospect/Customer/Partner |
| credit_limit | Currency | Hạn mức công nợ | VND |
| payment_terms | Number | Số ngày thanh toán | Default: 30 |
| created_date | Created Time | Ngày tạo | Auto |
| contacts | Link | Người liên hệ | Link to Contacts |
| opportunities | Link | Cơ hội | Link to Opportunities |
| contracts | Link | Hợp đồng | Link to Contracts |

#### 3.2.3. CONTACTS (Người liên hệ)

| Field | Type | Mô tả | Validation |
|-------|------|-------|------------|
| contact_id | Auto Number | ID tự động | PK |
| full_name | Text | Họ và tên | Required |
| title | Text | Chức danh | - |
| phone | Phone | Số điện thoại | - |
| email | Email | Email | - |
| account | Link | Công ty | Link to Accounts |
| decision_role | Single Select | Vai trò quyết định | Decision Maker/Influencer/User/Gatekeeper |
| is_primary | Checkbox | Liên hệ chính | - |
| notes | Long Text | Ghi chú | - |
| birthday | Date | Ngày sinh | - |

#### 3.2.4. OPPORTUNITIES (Cơ hội bán hàng)

| Field | Type | Mô tả | Validation |
|-------|------|-------|------------|
| opp_id | Auto Number | ID tự động | PK |
| opp_name | Text | Tên cơ hội | Required |
| account | Link | Công ty | Link to Accounts |
| contact | Link | Người liên hệ | Link to Contacts |
| stage | Single Select | Giai đoạn | Xem Pipeline Stages |
| amount | Currency | Giá trị | VND |
| probability | Number | Xác suất thắng | 0-100% |
| weighted_amount | Formula | Giá trị có trọng số | amount × probability |
| close_date | Date | Ngày dự kiến đóng | Required |
| owner | Person | Sales phụ trách | Link to Staff |
| products | Link | Sản phẩm | Link to Products |
| discount_percent | Number | % Giảm giá | 0-100 |
| discount_approved | Checkbox | Đã duyệt giảm giá | - |
| lost_reason | Single Select | Lý do thua | Nếu stage = Lost |
| competitor | Text | Đối thủ | - |
| next_action | Text | Hành động tiếp theo | - |
| next_action_date | Date | Ngày hành động | - |
| created_date | Created Time | Ngày tạo | Auto |
| won_date | Date | Ngày thắng | Khi stage = Won |

**Pipeline Stages:**

| Stage | Probability | Mô tả |
|-------|-------------|-------|
| Qualification | 10% | Đang tìm hiểu nhu cầu |
| Needs Analysis | 25% | Phân tích chi tiết |
| Proposal | 50% | Đã gửi báo giá |
| Negotiation | 75% | Đang đàm phán |
| Closed Won | 100% | Thắng |
| Closed Lost | 0% | Thua |

#### 3.2.5. PRODUCTS (Sản phẩm/Dịch vụ)

| Field | Type | Mô tả | Validation |
|-------|------|-------|------------|
| product_id | Auto Number | ID tự động | PK |
| sku | Text | Mã sản phẩm | Unique |
| product_name | Text | Tên sản phẩm | Required |
| category | Single Select | Danh mục | - |
| unit_price | Currency | Đơn giá | VND |
| cost | Currency | Giá vốn | VND |
| margin | Formula | Biên lợi nhuận | (unit_price - cost) / unit_price |
| description | Long Text | Mô tả | - |
| is_active | Checkbox | Đang bán | Default: true |

#### 3.2.6. STAFF (Nhân viên)

| Field | Type | Mô tả | Validation |
|-------|------|-------|------------|
| staff_id | Auto Number | ID tự động | PK |
| full_name | Text | Họ và tên | Required |
| email | Email | Email công ty | Required, Unique |
| phone | Phone | Số điện thoại | - |
| role | Single Select | Vai trò | Sales Rep/AM/Manager/Director |
| team | Link | Nhóm | Link to Teams |
| manager | Person | Quản lý trực tiếp | - |
| monthly_target | Currency | Chỉ tiêu tháng | VND |
| quarterly_target | Currency | Chỉ tiêu quý | VND |
| commission_rate | Number | % Hoa hồng | 0-100 |
| hire_date | Date | Ngày vào làm | - |
| is_active | Checkbox | Đang làm việc | Default: true |

#### 3.2.7. TEAMS (Nhóm/Phòng ban)

| Field | Type | Mô tả | Validation |
|-------|------|-------|------------|
| team_id | Auto Number | ID tự động | PK |
| team_name | Text | Tên nhóm | Required |
| department | Single Select | Phòng ban | Sales/AM/Pre-sales |
| leader | Person | Trưởng nhóm | Link to Staff |
| team_target | Currency | Chỉ tiêu nhóm | VND |
| members | Link | Thành viên | Link to Staff |

#### 3.2.8. KPIs (Chỉ số hiệu suất)

| Field | Type | Mô tả | Validation |
|-------|------|-------|------------|
| kpi_id | Auto Number | ID tự động | PK |
| staff | Link | Nhân viên | Link to Staff |
| period | Single Select | Kỳ | Monthly/Quarterly/Yearly |
| period_value | Text | Giá trị kỳ | VD: "2025-Q1", "2025-11" |
| target_revenue | Currency | Chỉ tiêu doanh thu | VND |
| actual_revenue | Currency | Doanh thu thực tế | VND |
| target_deals | Number | Chỉ tiêu số deal | - |
| actual_deals | Number | Số deal thực tế | - |
| target_activities | Number | Chỉ tiêu hoạt động | - |
| actual_activities | Number | Hoạt động thực tế | - |
| achievement_rate | Formula | Tỷ lệ hoàn thành | actual_revenue / target_revenue × 100 |
| commission_earned | Formula | Hoa hồng | Tính theo rule |

#### 3.2.9. CONTRACTS (Hợp đồng)

| Field | Type | Mô tả | Validation |
|-------|------|-------|------------|
| contract_id | Auto Number | ID tự động | PK |
| contract_code | Text | Số hợp đồng | Required, Unique |
| account | Link | Khách hàng | Link to Accounts |
| opportunity | Link | Cơ hội | Link to Opportunities |
| contract_value | Currency | Giá trị HĐ | VND |
| start_date | Date | Ngày bắt đầu | Required |
| end_date | Date | Ngày kết thúc | Required |
| duration_months | Formula | Thời hạn (tháng) | - |
| status | Single Select | Trạng thái | Draft/Active/Completed/Terminated |
| payment_schedule | Single Select | Lịch thanh toán | Xem bên dưới |
| products | Link | Sản phẩm | Link to Products |
| owner | Person | AM phụ trách | Link to Staff |
| signed_date | Date | Ngày ký | - |
| file_attachment | Attachment | File hợp đồng | - |
| invoices | Link | Hóa đơn | Link to Invoices |
| expenses | Link | Chi phí | Link to Expenses |
| renewal_reminder | Date | Nhắc gia hạn | end_date - 30 days |

**Payment Schedule Options:**
- 100% Upfront - Thanh toán 100% trước
- 50-50 - 50% trước, 50% sau
- 30-40-30 - 30% ký, 40% giữa, 30% hoàn thành
- Monthly - Hàng tháng
- Quarterly - Hàng quý
- Milestone - Theo milestone

#### 3.2.10. INVOICES (Hóa đơn)

| Field | Type | Mô tả | Validation |
|-------|------|-------|------------|
| invoice_id | Auto Number | ID tự động | PK |
| invoice_number | Text | Số hóa đơn | Required, Unique |
| contract | Link | Hợp đồng | Link to Contracts |
| account | Link | Khách hàng | Link to Accounts |
| invoice_date | Date | Ngày xuất | Required |
| due_date | Date | Hạn thanh toán | Required |
| amount | Currency | Số tiền | VND |
| vat_rate | Number | % VAT | Default: 10 |
| total_amount | Formula | Tổng cộng | amount × (1 + vat_rate/100) |
| status | Single Select | Trạng thái | Draft/Sent/Partial/Paid/Overdue |
| paid_amount | Rollup | Đã thanh toán | Sum từ Payments |
| remaining | Formula | Còn lại | total_amount - paid_amount |
| days_overdue | Formula | Số ngày quá hạn | today - due_date (nếu > 0) |
| payments | Link | Các lần thanh toán | Link to Payments |
| notes | Long Text | Ghi chú | - |

#### 3.2.11. PAYMENTS (Thanh toán)

| Field | Type | Mô tả | Validation |
|-------|------|-------|------------|
| payment_id | Auto Number | ID tự động | PK |
| payment_date | Date | Ngày thanh toán | Required |
| invoice | Link | Hóa đơn | Link to Invoices |
| amount | Currency | Số tiền | VND |
| payment_method | Single Select | Phương thức | Bank Transfer/Cash/Other |
| reference_number | Text | Số tham chiếu | - |
| bank_account | Text | Tài khoản ngân hàng | - |
| verified_by | Person | Người xác nhận | - |
| verified_date | Date | Ngày xác nhận | - |
| notes | Long Text | Ghi chú | - |
| attachment | Attachment | Chứng từ | - |

#### 3.2.12. EXPENSES (Chi phí)

| Field | Type | Mô tả | Validation |
|-------|------|-------|------------|
| expense_id | Auto Number | ID tự động | PK |
| expense_date | Date | Ngày phát sinh | Required |
| contract | Link | Hợp đồng liên quan | Link to Contracts (optional) |
| category | Single Select | Loại chi phí | Xem bên dưới |
| amount | Currency | Số tiền | VND |
| description | Text | Mô tả | Required |
| requested_by | Person | Người yêu cầu | - |
| approved_by | Person | Người duyệt | - |
| approval_status | Single Select | Trạng thái duyệt | Pending/Approved/Rejected |
| payment_status | Single Select | Trạng thái chi | Unpaid/Paid |
| paid_date | Date | Ngày chi | - |
| vendor | Text | Nhà cung cấp | - |
| attachment | Attachment | Chứng từ | - |

**Expense Categories:**
- Marketing - Chi phí marketing
- Sales Commission - Hoa hồng bán hàng
- Travel - Công tác
- Entertainment - Tiếp khách
- Operations - Vận hành
- Subcontractor - Thuê ngoài
- Equipment - Thiết bị
- Other - Khác

#### 3.2.13. CASH FLOW (Dòng tiền)

| Field | Type | Mô tả | Validation |
|-------|------|-------|------------|
| cashflow_id | Auto Number | ID tự động | PK |
| transaction_date | Date | Ngày giao dịch | Required |
| type | Single Select | Loại | IN/OUT |
| category | Single Select | Danh mục | Xem bên dưới |
| amount | Currency | Số tiền | VND |
| reference_type | Single Select | Loại tham chiếu | Payment/Expense/Transfer/Other |
| reference_id | Text | ID tham chiếu | - |
| account | Link | Khách hàng | Link to Accounts (nếu có) |
| description | Text | Mô tả | - |
| verified | Checkbox | Đã xác nhận | - |
| bank_account | Single Select | Tài khoản NH | - |
| running_balance | Formula | Số dư lũy kế | Tính toán |

**Cash Flow Categories:**

| Type | Categories |
|------|------------|
| IN | Customer Payment, Deposit Received, Refund Received, Investment, Loan, Other Income |
| OUT | Salary, Commission, Marketing, Operations, Vendor Payment, Tax, Loan Repayment, Other Expense |

---

## 4. Quy trình Phê duyệt (Approval)

### 4.1. Tổng quan Approval Workflows

| Approval | Trigger | Business Rule | Approver |
|----------|---------|---------------|----------|
| Discount Approval | Discount > 15% | Bảo vệ biên lợi nhuận | Manager → Director |
| Payment Request | Yêu cầu thanh toán | Kiểm soát chi tiêu | AM → Finance |
| Expense Claim | Ghi nhận chi phí | Kiểm soát chi phí | Manager → Finance |
| Revenue Recognition | Xác nhận thu tiền | Đảm bảo chính xác | Finance Verify |
| Fund Transfer | Chuyển tiền | Kiểm soát ngân quỹ | Finance → CFO |
| Credit Extension | Tổng công nợ | Quản lý rủi ro | AM → Finance → Director |

### 4.2. Chi tiết Từng Approval

#### 4.2.1. Discount Approval (Duyệt Giảm giá)

**Mục đích:** Đảm bảo không giảm giá quá mức, bảo vệ biên lợi nhuận

**Trigger Conditions:**

```
1  Nếu Discount % > 15%  → Cần duyệt Manager
2  Nếu Discount % > 30%  → Cần duyệt Manager + Director
3  Nếu Discount % > 50%  → Cần duyệt CEO
```

**Form Fields:**

| Field | Type | Mô tả |
|-------|------|-------|
| Opportunity | Reference | Link từ Opportunities |
| Customer | Reference | Tự động lấy từ Opportunity |
| Original Amount | Currency | Giá trị gốc |
| Discount % | Number | % Giảm giá đề xuất |
| Discount Amount | Formula | Số tiền giảm |
| Final Amount | Formula | Giá trị sau giảm |
| Reason | Long Text | Lý do xin giảm |
| Competitor Info | Text | Thông tin đối thủ |
| Attachment | File | Tài liệu đính kèm |

**Approval Flow:**

```
┌─────────┐     ┌──────────┐     ┌──────────┐
│ Submit  │────▶│ Manager  │────▶│ Director │
│ (Sales) │     │ (if >15%)│     │ (if >30%)│
└─────────┘     └──────────┘     └──────────┘
                     │                │
                     ▼                ▼
                 Approved         Approved
                    OR               OR
                 Rejected         Rejected
```

**After Approval Actions:**
- Update Opportunity: discount_approved = true
- Notify Sales via Messenger
- Log activity in Opportunity

#### 4.2.2. Payment Request (Yêu cầu Thanh toán)

**Mục đích:** Kiểm soát việc chi tiền, đảm bảo có đủ hóa đơn/chứng từ

**Trigger:** Button từ Expense record hoặc manual submit

**Form Fields:**

| Field | Type | Mô tả |
|-------|------|-------|
| Request Type | Select | Chi phí/Vendor/Khác |
| Expense/Invoice | Reference | Link nguồn |
| Amount | Currency | Số tiền cần chi |
| Vendor/Payee | Text | Người nhận |
| Bank Account | Text | Tài khoản nhận |
| Due Date | Date | Hạn thanh toán |
| Reason | Long Text | Lý do |
| Attachment | File | Chứng từ |

**Approval Matrix:**

| Amount | Approvers |
|--------|-----------|
| ≤ 5,000,000 VND | Line Manager |
| 5M - 50M VND | Line Manager → Finance Manager |
| > 50,000,000 VND | Line Manager → Finance Manager → CFO |

**After Approval Actions:**
- Create Cash Flow OUT record
- Update Expense: payment_status = Paid
- Notify requester

#### 4.2.3. Expense Claim (Ghi nhận Chi phí)

**Mục đích:** Ghi nhận và duyệt các chi phí phát sinh

**Form Fields:**

| Field | Type | Mô tả |
|-------|------|-------|
| Expense Date | Date | Ngày phát sinh |
| Category | Select | Loại chi phí |
| Amount | Currency | Số tiền |
| Contract | Reference | Hợp đồng liên quan (nếu có) |
| Description | Long Text | Mô tả chi tiết |
| Vendor | Text | Nhà cung cấp |
| Attachments | File | Hóa đơn, chứng từ |

**Approval Flow:**

```
┌─────────┐     ┌──────────────┐     ┌─────────┐
│ Submit  │────▶│ Line Manager │────▶│ Finance │
│ (Staff) │     │              │     │(if > 5M)│
└─────────┘     └──────────────┘     └─────────┘
                     │
                     ▼
                ┌─────────┐
                │ Create  │
                │ Expense │
                │ Record  │
                └─────────┘
```

#### 4.2.4. Revenue Recognition (Hoạch toán Doanh thu)

**Mục đích:** Xác nhận đã nhận được tiền từ khách hàng

**Trigger:** Payment received notification hoặc Bank statement import

**Form Fields:**

| Field | Type | Mô tả |
|-------|------|-------|
| Payment Date | Date | Ngày nhận tiền |
| Amount | Currency | Số tiền |
| Invoice | Reference | Hóa đơn liên quan |
| Customer | Reference | Khách hàng |
| Bank Reference | Text | Số tham chiếu ngân hàng |
| Bank Account | Select | Tài khoản |
| Attachment | File | Sao kê/Chứng từ |

**After Approval Actions:**
- Create Payment record
- Update Invoice status
- Create Cash Flow IN record
- Update KPI actual_revenue

#### 4.2.5. Fund Transfer (Lệnh Chuyển tiền)

**Mục đích:** Kiểm soát việc chuyển tiền đi

**Form Fields:**

| Field | Type | Mô tả |
|-------|------|-------|
| Transfer Type | Select | Internal/External |
| Amount | Currency | Số tiền |
| From Account | Select | Tài khoản nguồn |
| To Account | Text | Tài khoản đích |
| Beneficiary | Text | Người thụ hưởng |
| Purpose | Long Text | Mục đích chuyển |
| Scheduled Date | Date | Ngày chuyển |
| Attachment | File | Chứng từ |

**Approval Matrix:**

| Amount | Approvers |
|--------|-----------|
| < 50M VND | Finance Manager |
| 50M - 500M VND | Finance Manager → CFO |
| > 500M VND | Finance Manager → CFO → CEO |

**After Approval Actions:**
- Create Cash Flow OUT record
- Notify Bank team
- Update accounting records

#### 4.2.6. Credit Extension (Gia hạn Công nợ)

**Mục đích:** Cho phép khách hàng nợ thêm hoặc quá hạn

**Trigger:** Khi AR > credit limit hoặc overdue > X days

**Form Fields:**

| Field | Type | Mô tả |
|-------|------|-------|
| Customer | Reference | Khách hàng |
| Current AR | Currency | Công nợ hiện tại |
| Current Limit | Currency | Hạn mức hiện tại |
| Requested Limit | Currency | Hạn mức đề xuất |
| Days Overdue | Number | Số ngày quá hạn |
| Extension Days | Number | Số ngày gia hạn |
| Reason | Long Text | Lý do |
| Payment Plan | Long Text | Kế hoạch thanh toán |

**Approval Flow:**

```
┌─────────┐     ┌────────────┐     ┌─────────┐     ┌──────────┐
│ Submit  │────▶│ Account Mgr│────▶│ Finance │────▶│ Director │
│  (AM)   │     │  (Review)  │     │ (Verify)│     │ (Approve)│
└─────────┘     └────────────┘     └─────────┘     └──────────┘
```

### 4.3. Tích hợp Approval với Base

**Sync Data từ Approval về Base:**
1. Vào Approval app → Data Management
2. Chọn "Add Approval Data"
3. Chọn Approval cần sync
4. Dữ liệu tự động cập nhật về Base mỗi giờ

**Automation khi Approval hoàn thành:**

```
Trigger: Approval approved/rejected
Action:
  - Send Lark message to submitter
  - Initiate new approval (if needed)
  - Webhook to update Base (via AnyCross)
```

---

## 5. Vai trò & Phân quyền

### 5.1. Định nghĩa Vai trò

| Vai trò | Mô tả | Số lượng ước tính |
|---------|-------|-------------------|
| Sales Rep | Nhân viên bán hàng | 30-Oct |
| Account Manager | Quản lý khách hàng | 15-May |
| Sales Manager | Quản lý Sales team | 5-Feb |
| Finance Staff | Nhân viên kế toán | 5-Feb |
| Finance Manager | Quản lý tài chính | 2-Jan |
| Director | Giám đốc | 2-Jan |
| Admin | Quản trị hệ thống | 2-Jan |

### 5.2. Ma trận Phân quyền

#### 5.2.1. Quyền trên Tables

| Table | Sales Rep | AM | Sales Mgr | Finance | Director | Admin |
|-------|-----------|-----|-----------|---------|----------|-------|
| Leads | CRUD (own) | View | CRUD (team) | View | View All | Full |
| Accounts | View | CRUD (own) | CRUD (team) | View | View All | Full |
| Contacts | View | CRUD | View (team) | View | View All | Full |
| Opportunities | CRUD (own) | View | CRUD (team) | View | View All | Full |
| Products | View | View | View | View | Edit | Full |
| Staff | View own | View own | View team | View | View All | Full |
| Teams | View own | View own | Edit own | View | View All | Full |
| KPIs | View own | View own | View team | View | View All | Full |
| Contracts | View | CRUD (own) | View (team) | View | View All | Full |
| Invoices | View | View | View | CRUD | View All | Full |
| Payments | - | View | View | CRUD | View All | Full |
| Expenses | Create | Create | Approve | CRUD | View All | Full |
| Cash Flow | - | - | View | CRUD | View All | Full |

**Chú thích:**
- CRUD = Create, Read, Update, Delete
- (own) = Chỉ records mình tạo/được assign
- (team) = Records của team mình quản lý

#### 5.2.2. Quyền trên Dashboards

| Dashboard | Sales Rep | AM | Sales Mgr | Finance | Director | Admin |
|-----------|-----------|-----|-----------|---------|----------|-------|
| My Performance | ✓ | ✓ | ✓ | - | - | ✓ |
| Sales Pipeline | ✓ | ✓ | ✓ | - | ✓ | ✓ |
| Team Performance | - | - | ✓ | - | ✓ | ✓ |
| Finance Overview | - | - | - | ✓ | ✓ | ✓ |
| Cash Flow | - | - | - | ✓ | ✓ | ✓ |
| AR Aging | - | ✓ | ✓ | ✓ | ✓ | ✓ |
| Executive Summary | - | - | - | - | ✓ | ✓ |

#### 5.2.3. Quyền trên Approvals

| Approval | Submitter | Approver |
|----------|-----------|----------|
| Discount | Sales Rep | Manager → Director |
| Payment Request | AM, Finance | Manager → Finance → CFO |
| Expense Claim | All Staff | Manager → Finance |
| Revenue Recognition | Finance | Finance Manager |
| Fund Transfer | Finance | Finance → CFO |
| Credit Extension | AM | AM → Finance → Director |

### 5.3. Cấu hình Advanced Permissions trong Base

**Bước 1: Enable Advanced Permissions**
1. Mở Base → Click "Advanced Permissions" ở góc phải trên
2. Chọn "Enable Advanced Permissions"

**Bước 2: Tạo Roles**

```
Role 1: Sales Rep
  - Tables: Leads (CRUD where assigned_to = me), Opportunities (CRUD where owner = me)
  - Dashboards: My Performance, Sales Pipeline (filtered)

Role 2: Account Manager
  - Tables: Accounts (CRUD where account_owner = me), Contracts, Invoices (view)
  - Dashboards: My Performance, AR Aging

Role 3: Sales Manager
  - Tables: All sales tables (view team), Expenses (approve)
  - Dashboards: Team Performance, Sales Pipeline

Role 4: Finance
  - Tables: Invoices, Payments, Expenses, Cash Flow (CRUD)
  - Dashboards: Finance Overview, Cash Flow, AR Aging

Role 5: Director
  - Tables: All (view)
  - Dashboards: All

Role 6: Admin
  - Full access
```

**Bước 3: Assign Users to Roles**
1. Trong Advanced Permissions panel → Click vào Role
2. Click "Assign Role" → Thêm users

---

## 6. User Journey Map

### 6.1. Sales Rep Journey

```
┌───────────────────────────────────────────────────────────────┐
│                  SALES REP - DAILY JOURNEY                    │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌───────┐│
│  │ RECEIVE │─▶│ QUALIFY │─▶│ CREATE  │─▶│ NURTURE │─▶│ CLOSE ││
│  │  LEAD   │  │  LEAD   │  │   OPP   │  │   OPP   │  │ DEAL  ││
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘  └───────┘│
│       │           │            │            │           │     │
│       ▼           ▼            ▼            ▼           ▼     │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌───────┐│
│  │Messenger│  │ Base App│  │ Base App│  │ Base App│  │Approval│
│  │Bot notify│ │Lead form│  │Opp form │  │Log calls│  │Discount│
│  │         │  │Score lead│ │Add prods│  │Send quote│ │(if any)││
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘  └───────┘│
│                                                               │
│  TOOLS USED:                                                  │
│  • Messenger: Nhận notification về lead mới                   │
│  • Base App: Xem/cập nhật Leads, Opportunities                │
│  • Calendar: Lịch hẹn, Follow-up                              │
│  • Approval: Submit discount request (nếu > 15%)              │
│                                                               │
│  KPIs TRACKED:                                                │
│  • Số Lead được assign                                        │
│  • Conversion rate (Lead → Opp)                               │
│  • Win rate                                                   │
│  • Revenue closed                                             │
└───────────────────────────────────────────────────────────────┘
```

**Chi tiết từng bước:**

| Bước | Action | Tool | Outcome |
|------|--------|------|---------|
| 1. Receive Lead | Nhận thông báo lead mới | Messenger Bot | Biết có lead mới |
| 2. Qualify | Gọi điện, đánh giá nhu cầu | Base App (Lead form) | Lead qualified/disqualified |
| 3. Create Opp | Tạo Opportunity | Base App | Opp in pipeline |
| 4. Nurture | Follow-up, gửi tài liệu | Calendar, Base App | Move stages |
| 5. Close | Đàm phán, chốt deal | Approval (nếu discount) | Won/Lost |

### 6.2. Account Manager Journey

```
┌───────────────────────────────────────────────────────────────┐
│               ACCOUNT MANAGER - DAILY JOURNEY                 │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌───────┐│
│  │ RECEIVE │  │ CREATE  │  │ MANAGE  │  │ COLLECT │  │RENEWAL││
│  │  WON    │  │CONTRACT │  │DELIVERY │  │ PAYMENT │  │/UPSELL││
│  │  OPP    │  │         │  │         │  │         │  │       ││
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘  └───────┘│
│       │           │            │            │           │     │
│       ▼           ▼            ▼            ▼           ▼     │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌───────┐│
│  │Messenger│  │ Base App│  │ Base App│  │ Approval│  │Base App│
│  │Notify   │  │ Contract│  │Log expense│ │ Payment │  │Dashboard│
│  │         │  │ Create  │  │Track issue│ │ Request │  │Renewal ││
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘  └───────┘│
│                                                               │
│  TOOLS USED:                                                  │
│  • Base App: Quản lý Accounts, Contracts, track activities    │
│  • Approval: Submit expense claim, payment request            │
│  • Calendar: Meeting với khách hàng                           │
│  • Dashboard: Xem AR aging, contract renewal                  │
│                                                               │
│  KPIs TRACKED:                                                │
│  • Contract value managed                                     │
│  • Collection rate                                            │
│  • Customer satisfaction                                      │
│  • Renewal rate                                               │
└───────────────────────────────────────────────────────────────┘
```

### 6.3. Finance Staff Journey

```
┌───────────────────────────────────────────────────────────────┐
│               FINANCE STAFF - DAILY JOURNEY                   │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌───────┐│
│  │ CREATE  │  │ VERIFY  │  │ PROCESS │  │ RECORD  │  │ REPORT││
│  │ INVOICE │  │ PAYMENT │  │ EXPENSE │  │CASH FLOW│  │       ││
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘  └───────┘│
│       │           │            │            │           │     │
│       ▼           ▼            ▼            ▼           ▼     │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌───────┐│
│  │ Base App│  │ Approval│  │ Approval│  │ Base App│  │Dashboard│
│  │ Invoice │  │ Revenue │  │ Payment │  │Cash Flow│  │Finance ││
│  │ Create  │  │ Recog.  │  │ Request │  │  Entry  │  │Overview││
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘  └───────┘│
│                                                               │
│  TOOLS USED:                                                  │
│  • Base App: Quản lý Invoices, Payments, Expenses, Cash Flow  │
│  • Approval: Duyệt expense, payment, revenue recognition      │
│  • Dashboard: Finance overview, AR aging, cash flow report    │
│                                                               │
│  KPIs TRACKED:                                                │
│  • DSO (Days Sales Outstanding)                               │
│  • Collection rate                                            │
│  • Cash flow accuracy                                         │
│  • Processing time                                            │
└───────────────────────────────────────────────────────────────┘
```

### 6.4. Manager Journey

```
┌───────────────────────────────────────────────────────────────┐
│                   MANAGER - DAILY JOURNEY                     │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌───────┐│
│  │ REVIEW  │  │ APPROVE │  │  COACH  │  │FORECAST │  │ REPORT││
│  │DASHBOARD│  │ REQUESTS│  │  TEAM   │  │         │  │       ││
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘  └───────┘│
│       │           │            │            │           │     │
│       ▼           ▼            ▼            ▼           ▼     │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌───────┐│
│  │Dashboard│  │ Approval│  │Messenger│  │ Base App│  │Dashboard│
│  │Team Perf│  │Discount │  │Chat/Call│  │ Pipeline│  │Executive│
│  │Pipeline │  │Expense  │  │         │  │ Review  │  │Summary ││
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘  └───────┘│
│                                                               │
│  DAILY ROUTINE:                                               │
│  08:00 - Review team dashboard                                │
│  09:00 - Process pending approvals                            │
│  10:00 - 1:1 với team members                                 │
│  14:00 - Pipeline review                                      │
│  16:00 - Forecast update                                      │
└───────────────────────────────────────────────────────────────┘
```

---

## 7. Thiết kế Dashboard

### 7.1. Dashboard Architecture

```
┌───────────────────────────────────────────────────────────────┐
│                    DASHBOARD HIERARCHY                        │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│                ┌─────────────────────────┐                    │
│                │    EXECUTIVE SUMMARY    │  ← Director/CEO    │
│                │   (High-level KPIs)     │                    │
│                └───────────┬─────────────┘                    │
│                            │                                  │
│          ┌─────────────────┼─────────────────┐                │
│          ▼                 ▼                 ▼                │
│  ┌───────────────┐ ┌───────────────┐ ┌───────────────┐        │
│  │SALES DASHBOARD│ │FINANCE DASHBOARD│ │OPS DASHBOARD│        │
│  │(Sales Manager)│ │ (Finance Mgr) │ │ (Ops Manager)│        │
│  └───────┬───────┘ └───────┬───────┘ └───────┬───────┘        │
│          │                 │                 │                │
│          ▼                 ▼                 ▼                │
│  ┌───────────────┐ ┌───────────────┐ ┌───────────────┐        │
│  │MY PERFORMANCE │ │   AR AGING    │ │CONTRACT STATUS│        │
│  │  (Sales/AM)   │ │ (AM/Finance)  │ │     (AM)      │        │
│  └───────────────┘ └───────────────┘ └───────────────┘        │
└───────────────────────────────────────────────────────────────┘
```

### 7.2. Dashboard 1: My Performance (Cá nhân)

**Audience:** Sales Rep, Account Manager

**Mục đích:** Theo dõi hiệu suất cá nhân

**Layout:**

```
┌───────────────────────────────────────────────────────────────┐
│                      MY PERFORMANCE                           │
├───────────────────────────────────────────────────────────────┤
│  ┌─────────────┬─────────────┬─────────────┬─────────────┐    │
│  │   TARGET    │   ACTUAL    │ ACHIEVEMENT │    DEALS    │    │
│  │  100,000$   │  75,000$    │     75%     │     12      │    │
│  │ This Month  │ This Month  │             │     Won     │    │
│  └─────────────┴─────────────┴─────────────┴─────────────┘    │
│                                                               │
│  ┌────────────────────────────┬────────────────────────┐      │
│  │   PIPELINE BY STAGE        │      ACTIVITIES        │      │
│  │  ███████ Qualification     │       Calls: 25        │      │
│  │  █████████ Needs Analysis  │       Meetings: 8      │      │
│  │  ████ Proposal             │       Emails: 45       │      │
│  │  ██ Negotiation            │       Proposals: 5     │      │
│  └────────────────────────────┴────────────────────────┘      │
│                                                               │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                   MY OPPORTUNITIES                       │  │
│  │  │   Name   │ Account  │  Amount  │  Stage   │ Close  │   │
│  │  │ Deal ABC │Company X │ $50,000  │ Proposal │  Dec   │   │
│  │  │Project XYZ│Company Y│ $30,000  │  Negot.  │  Nov   │   │
│  └─────────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────────┘
```

**Blocks cần thiết:**

| Block Type | Data Source | Filter |
|------------|-------------|--------|
| Number Card | KPIs | staff = current user; period = current month |
| Progress Bar | KPIs | achievement_rate |
| Pie Chart | Opportunities | stage, where owner = current user |
| Summary | Activities | where staff = current user; this month |
| List View | Opportunities | where owner = current user; open only |

### 7.3. Dashboard 2: Sales Pipeline (Team)

**Audience:** Sales Manager, Director

**Mục đích:** Theo dõi pipeline toàn team

**Layout:**

```
┌───────────────────────────────────────────────────────────────┐
│                     SALES PIPELINE                            │
├───────────────────────────────────────────────────────────────┤
│  ┌─────────────┬─────────────┬─────────────┬─────────────┐    │
│  │    TOTAL    │  WEIGHTED   │  WIN RATE   │  AVG DEAL   │    │
│  │   PIPELINE  │  PIPELINE   │             │    SIZE     │    │
│  │    $500K    │   $250M     │     30%     │    $25K     │    │
│  └─────────────┴─────────────┴─────────────┴─────────────┘    │
│                                                               │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                   PIPELINE FUNNEL                        │  │
│  │         Qualification (15 deals, $150K)                  │  │
│  │           Needs Analysis (10 deals, $100K)               │  │
│  │              Proposal (8 deals, $80K)                    │  │
│  │                Negotiation (5, $50K)                     │  │
│  └─────────────────────────────────────────────────────────┘  │
│                                                               │
│  ┌────────────────────────────┬────────────────────────┐      │
│  │    PIPELINE BY REP         │   PIPELINE BY PRODUCT  │      │
│  │   ██████ Rep A: $120K      │  ████████ Product X    │      │
│  │   █████ Rep B: $100K       │  ████ Product Y        │      │
│  │   ███ Rep C: $80K          │  ██ Product Z          │      │
│  └────────────────────────────┴────────────────────────┘      │
│                                                               │
│  │            DEALS CLOSING THIS MONTH                      │  │
│  │  Kanban View: Qualification | Analysis | Proposal | Nego │  │
└───────────────────────────────────────────────────────────────┘
```

### 7.4. Dashboard 3: Finance Overview

**Audience:** Finance Team, Director

**Mục đích:** Theo dõi tình hình tài chính

**Layout:**

```
┌───────────────────────────────────────────────────────────────┐
│                    FINANCE OVERVIEW                           │
├───────────────────────────────────────────────────────────────┤
│  ┌─────────────┬─────────────┬─────────────┬─────────────┐    │
│  │   REVENUE   │  COLLECTED  │     AR      │     DSO     │    │
│  │ This Month  │ This Month  │ Outstanding │    Days     │    │
│  │   $200K     │   $150K     │   $300K     │     45      │    │
│  └─────────────┴─────────────┴─────────────┴─────────────┘    │
│                                                               │
│  ┌────────────────────────────┬────────────────────────┐      │
│  │     REVENUE TREND          │       CASH FLOW        │      │
│  │  $200K ────────────────────│  +50K ───────/─────────│      │
│  │  $150K ────/───────────────│     0 ─────────────────│      │
│  │  $100K ────────────────────│  -50K ─────────────────│      │
│  │       Jan Feb Mar Apr      │       Jan Feb Mar Apr  │      │
│  └────────────────────────────┴────────────────────────┘      │
│                                                               │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                   AR AGING SUMMARY                       │  │
│  │  Current   : $100K  ████████████████████████████████████ │  │
│  │  1-30 days : $80K   ████████████████████████████         │  │
│  │  31-60 days: $60K   ████████████████████                 │  │
│  │  61-90 days: $40K   ████████████                         │  │
│  │  90+ days  : $20K   ████                                 │  │
│  └─────────────────────────────────────────────────────────┘  │
│                                                               │
│  │           OVERDUE INVOICES (ACTION NEEDED)               │  │
│  │  Invoice # │ Customer  │  Amount  │ Days Overdue         │  │
│  │  INV-001   │ Company A │  $10,000 │  45 days   [Action]  │  │
│  │  INV-002   │ Company B │   $8,000 │  32 days   [Action]  │  │
└───────────────────────────────────────────────────────────────┘
```

### 7.5. Dashboard 4: Cash Flow

**Audience:** Finance Manager, CFO

**Mục đích:** Theo dõi dòng tiền chi tiết

**Layout:**

```
┌───────────────────────────────────────────────────────────────┐
│                       CASH FLOW                               │
├───────────────────────────────────────────────────────────────┤
│  ┌─────────────┬─────────────┬─────────────┬─────────────┐    │
│  │   CASH IN   │  CASH OUT   │  NET FLOW   │   BALANCE   │    │
│  │ This Month  │ This Month  │ This Month  │   Current   │    │
│  │   $150K     │   $100K     │    +$50K    │    $500K    │    │
│  │   ▲ +15%    │   ▲ +10%    │   ▲ +25%    │             │    │
│  └─────────────┴─────────────┴─────────────┴─────────────┘    │
│                                                               │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │              CASH FLOW TREND (12 months)                 │  │
│  │  +100K ───────────────/────────────────────────────────  │  │
│  │      0 ─────────/───────────────────────────────────────  │  │
│  │  -50K ──────────────────────────────────────────────────  │  │
│  │       J F M A M J J A S O N D                            │  │
│  │       ─── Cash In  ─── Cash Out  ─── Net                 │  │
│  └─────────────────────────────────────────────────────────┘  │
│                                                               │
│  ┌────────────────────────────┬────────────────────────────┐  │
│  │   CASH IN BY CATEGORY      │   CASH OUT BY CATEGORY     │  │
│  │   Customer Payment: 85%    │   Salary: 40%              │  │
│  │   Other Income: 15%        │   Operations: 30%          │  │
│  │                            │   Marketing: 20%           │  │
│  │                            │   Other: 10%               │  │
│  └────────────────────────────┴────────────────────────────┘  │
│                                                               │
│  │                  RECENT TRANSACTIONS                     │  │
│  │  Date   │ Type │ Category        │ Amount   │ Ref        │  │
│  │  Nov 20 │  IN  │ Customer Payment│ +$15K    │ PAY-01     │  │
│  │  Nov 19 │ OUT  │ Salary          │ -$30K    │ EXP-03     │  │
│  │  Nov 18 │  IN  │ Customer Payment│ +$25K    │ PAY-02     │  │
└───────────────────────────────────────────────────────────────┘
```

### 7.6. Dashboard 5: Executive Summary

**Audience:** Director, CEO

**Mục đích:** Overview toàn bộ business

**Layout:**

```
┌───────────────────────────────────────────────────────────────┐
│                    EXECUTIVE SUMMARY                          │
├───────────────────────────────────────────────────────────────┤
│  ┌─────────────┬─────────────┬─────────────┬─────────────┐    │
│  │   REVENUE   │   GROWTH    │  PIPELINE   │    CASH     │    │
│  │     YTD     │     YoY     │    Value    │  Position   │    │
│  │    $2.5M    │    +25%     │    $500M    │    $500K    │    │
│  └─────────────┴─────────────┴─────────────┴─────────────┘    │
│                                                               │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                 REVENUE vs TARGET                        │  │
│  │  Target: $3M                                             │  │
│  │  Actual: $2.5M  █████████████████████████████████ 83%    │  │
│  └─────────────────────────────────────────────────────────┘  │
│                                                               │
│  ┌────────────────────────────┬────────────────────────────┐  │
│  │     TOP 5 ACCOUNTS         │     TOP PERFORMERS         │  │
│  │  1. Company A: $500K       │  1. Rep A: 125% target     │  │
│  │  2. Company B: $400K       │  2. Rep B: 110% target     │  │
│  │  3. Company C: $300K       │  3. Rep C: 105% target     │  │
│  │  4. Company D: $200K       │  4. Rep D: 95% target      │  │
│  │  5. Company E: $100K       │  5. Rep E: 90% target      │  │
│  └────────────────────────────┴────────────────────────────┘  │
│                                                               │
│  ┌────────────────────────────┬────────────────────────────┐  │
│  │     ALERTS & ACTIONS       │     KEY METRICS TREND      │  │
│  │  ⚠ 5 invoices overdue      │  Revenue ▲ +25% YoY        │  │
│  │  ⚠ 3 contracts expiring    │  Win Rate ▲ +5%            │  │
│  │  ✓ Cash flow positive      │  DSO ▼ -10 days            │  │
│  │  ⚠ 2 deals need help       │  NPS ▲ +10 points          │  │
│  └────────────────────────────┴────────────────────────────┘  │
└───────────────────────────────────────────────────────────────┘
```

---

## 8. Automation & Business Rules

### 8.1. Tổng quan Automation

```
┌───────────────────────────────────────────────────────────────┐
│                  AUTOMATION ARCHITECTURE                      │
├───────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                    BASE AUTOMATION                       │  │
│  │  Simple triggers → Simple actions                        │  │
│  │  • On record create/update                               │  │
│  │  • Send notification                                     │  │
│  │  • Update field                                          │  │
│  └─────────────────────────────────────────────────────────┘  │
│                            │                                  │
│                            ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                    BASE WORKFLOW                         │  │
│  │  Complex logic với IF/ELSE                               │  │
│  │  • Conditional branches                                  │  │
│  │  • Multiple actions                                      │  │
│  │  • Visual flow builder                                   │  │
│  └─────────────────────────────────────────────────────────┘  │
│                            │                                  │
│                            ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                      ANYCROSS                            │  │
│  │  Cross-app orchestration                                 │  │
│  │  • Connect external systems                              │  │
│  │  • Complex multi-step workflows                          │  │
│  │  • API integrations                                      │  │
│  └─────────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────────┘
```

### 8.2. Automation Rules Chi tiết

#### 8.2.1. Lead Management

| Rule | Trigger | Condition | Action |
|------|---------|-----------|--------|
| Auto-assign Lead | Lead created | channel = specific | Assign to specific sales rep |
| Lead Score Update | Lead updated | score >= 70 | Change status to "Qualified" |
| Lead Notification | Lead created | - | Send Messenger to assigned sales |
| Stale Lead Alert | Scheduled (daily) | No activity 7 days | Notify sales + manager |
| Lead Conversion | Status = "Converted" | - | Create Account + Contact + Opportunity |

**Auto-assign Logic:**

```
1  IF channel = "Website" THEN
2    Round-robin assign to Sales Team A
3  ELSE IF channel = "Referral" THEN
4    Assign to Rep who owns referring account
5  ELSE IF channel = "Partner" THEN
6    Assign to Partner Manager
7  ELSE
8    Assign to Lead Pool
```

#### 8.2.2. Opportunity Management

| Rule | Trigger | Condition | Action |
|------|---------|-----------|--------|
| Stage Change Notify | Opp.stage changed | - | Notify manager + log activity |
| Discount Alert | discount_percent changed | > 15% | Notify manager, block if not approved |
| Close Date Passed | Scheduled (daily) | close_date = today, not closed | Alert sales + manager |
| Won Deal Process | Stage = "Closed Won" | - | Create Contract, notify team, update KPI |
| Lost Deal Process | Stage = "Closed Lost" | - | Log reason, notify manager, update KPI |
| Probability Update | Stage changed | - | Auto-update probability based on stage |

**Won Deal Automation Flow:**

```
1  Trigger: Opportunity.stage = "Closed Won"
2  │
3  ├── Update Opportunity
4  │   • won_date = today
5  │   • probability = 100%
6  │
7  ├── Create Contract
8  │   • Copy account, products, amount
9  │   • Set status = "Draft"
10 │
11 ├── Notify via Messenger
12 │   • Sales Rep: "Congratulations!"
13 │   • Manager: "New deal won"
14 │   • Team channel: Announcement
15 │
16 ├── Update KPI
17 │   • Add to actual_revenue
18 │   • Increment actual_deals
19 │
20 └── Create Calendar Event
21     • Contract signing meeting
22     • Invite relevant parties
```

#### 8.2.3. Contract Management

| Rule | Trigger | Condition | Action |
|------|---------|-----------|--------|
| Contract Activation | signed_date set | - | Change status to "Active", notify AM |
| Renewal Reminder | Scheduled (daily) | end_date - 60 days | Notify AM to start renewal |
| Contract Expiring | Scheduled (daily) | end_date - 30 days | Alert AM + Manager |
| Invoice Generation | Contract activated | payment_schedule set | Create invoices based on schedule |

**Invoice Generation Logic:**

```
1  IF payment_schedule = "100% Upfront" THEN
2    Create 1 Invoice: amount = contract_value, due = signed_date + 7
3
4  ELSE IF payment_schedule = "50-50" THEN
5    Create Invoice 1: amount = 50%, due = signed_date + 7
6    Create Invoice 2: amount = 50%, due = end_date - 30
7
8  ELSE IF payment_schedule = "30-40-30" THEN
9    Create Invoice 1: amount = 30%, due = signed_date + 7
10   Create Invoice 2: amount = 40%, due = (end_date - start_date)/2
11   Create Invoice 3: amount = 30%, due = end_date
12
13 ELSE IF payment_schedule = "Monthly" THEN
14   FOR each month to contract duration
15     Create Invoice: amount = contract_value / months, due = month_start + 5
```

#### 8.2.4. Invoice & Payment

| Rule | Trigger | Condition | Action |
|------|---------|-----------|--------|
| Invoice Sent | status = "Sent" | - | Notify AM, start due date tracking |
| Payment Received | Payment created | - | Update invoice paid_amount, check if fully paid |
| Invoice Overdue | Scheduled (daily) | due_date = today, not paid | Alert AM + Finance, escalate |
| Invoice Paid | paid_amount >= total_amount | - | Update status, notify AM |

**Payment Processing Flow:**

```
1  Trigger: Payment record created
2  │
3  ├── Update Invoice
4  │   • Add payment to paid_amount (rollup)
5  │   • IF paid_amount >= total_amount
6  │       THEN status = "Paid"
7  │   • ELSE IF paid_amount > 0
8  │       THEN status = "Partial"
9  │
10 ├── Create Cash Flow IN
11 │   • transaction_date = payment_date
12 │   • type = "IN"
13 │   • category = "Customer Payment"
14 │   • amount = payment_amount
15 │   • reference = Payment ID
16 │
17 ├── Update Account
18 │   • Recalculate total AR
19 │
20 └── Notify
21     • AM: "Payment received from [Customer]"
22     • Finance: "New payment recorded"
```

#### 8.2.5. Expense & Cash Flow

| Rule | Trigger | Condition | Action |
|------|---------|-----------|--------|
| Expense Approved | approval_status = "Approved" | - | Create Cash Flow OUT entry |
| Expense Paid | payment_status = "Paid" | - | Update Cash Flow, notify requester |
| Daily Cash Summary | Scheduled (daily 6PM) | - | Send cash summary to Finance |
| Low Cash Alert | Cash balance checked | balance < threshold | Alert CFO |

### 8.3. Notification Templates

#### 8.3.1. Messenger Notifications

**New Lead Assigned:**

```
1  📋 Lead Mới Được Phân Công
2
3  Tên: {{lead_name}}
4  Công ty: {{company_name}}
5  Nguồn: {{channel}}
6  Điện thoại: {{phone}}
7
8  👉 Xem chi tiết: [Link to Lead]
9
10 Vui lòng Liên hệ trong vòng 24h!
```

**Deal Won:**

```
1  🎉 CHÚC MỪNG! Deal Thắng!
2
3  Deal: {{opp_name}}
4  Khách hàng: {{account.company_name}}
5  Giá trị: {{amount}}
6  Sales: {{owner.full_name}}
7
8  Tổng doanh thu tháng này: {{monthly_total}}
```

**Invoice Overdue:**

```
1  ⚠ HÓA ĐƠN QUÁ HẠN
2
3  Invoice: {{invoice_number}}
4  Khách hàng: {{account.company_name}}
5  Số tiền: {{total_amount}}
6  Quá hạn: {{days_overdue}} ngày
7
8  AM phụ trách: {{account.account_owner.full_name}}
9
10 👉 Cần follow-up ngay!
```

**Approval Request:**

```
1  📝 YÊU CẦU PHÊ DUYỆT
2
3  Loại: {{approval_type}}
4  Người gửi: {{submitter.full_name}}
5  Số tiền: {{amount}}
6  Mô tả: {{description}}
7
8  👉 Duyệt/Từ chối: [Link to Approval]
```

---

## 9. Tích hợp Lark Components

### 9.1. Integration Matrix

| Component | Vai trò | Kết nối với | Cách tích hợp |
|-----------|---------|-------------|---------------|
| Base App | UI/UX | Base (data) | Pages, Blocks, Buttons |
| Base | Database | All components | Tables, Links, Formulas |
| Base Automation | Simple rules | Base, Messenger | Triggers, Actions |
| Base Workflow | Complex logic | Base, Approval | IF/ELSE, Multi-step |
| Approval | Human decisions | Base, Messenger | Forms, Approvers |
| Messenger | Communication | All | Bot, Notifications |
| Calendar | Scheduling | Base, Messenger | Events, Reminders |
| AnyCross | External systems | External APIs | Connectors, Webhooks |

### 9.2. Base App Configuration

#### 9.2.1. Pages Structure

```
1  BASE APP: CRM B2B
2  │
3  ├── 📱 Sales Portal
4  │   ├── Home (Dashboard: My Performance)
5  │   ├── My Leads (List: Leads where assigned_to = me)
6  │   ├── My Opportunities (Kanban: by stage)
7  │   ├── Accounts (List: all I have access)
8  │   └── Activities (Calendar view)
9  │
10 ├── 📱 AM Portal
11 │   ├── Home (Dashboard: AR Overview)
12 │   ├── My Accounts (List: where account_owner = me)
13 │   ├── Contracts (List: my contracts)
14 │   ├── Invoices (List: my accounts' invoices)
15 │   └── Collection Tasks (List: overdue invoices)
16 │
17 ├── 📱 Finance Portal
18 │   ├── Home (Dashboard: Finance Overview)
19 │   ├── Invoices (List: all)
20 │   ├── Payments (List: all)
21 │   ├── Expenses (List: pending approval)
22 │   └── Cash Flow (List + Chart)
23 │
24 └── 📱 Management Portal
25     ├── Executive Summary (Dashboard)
26     ├── Team Performance (Dashboard)
27     ├── Pipeline Review (Kanban)
28     └── Approvals (Pending items)
```

#### 9.2.2. Action Buttons

| Button | Location | Action | Triggers |
|--------|----------|--------|----------|
| "Convert to Opp" | Lead record | Create Opportunity | Workflow |
| "Create Contract" | Opportunity (Won) | Create Contract | Workflow |
| "Generate Invoice" | Contract | Create Invoice(s) | Workflow |
| "Record Payment" | Invoice | Open Payment form | Form |
| "Request Discount" | Opportunity | Submit Approval | Approval |

### 9.3. Approval Integration

**Sync Approval Data to Base:**
1. Approval App → Data Management → Add Approval Data
2. Select approval form(s) to sync
3. Data auto-syncs to Base every hour

**Reference Base Data in Approval:**
1. Approval Admin → Form Design
2. Add "Data from Base" widget
3. Select Base table and fields
4. Users can lookup Base data when submitting

**Automation after Approval:**

```
1  Approval Form: Discount Request
2  │
3  ├── Trigger: Process Approved
4  │   ├── Action 1: Send Lark message to submitter
5  │   │   └── "Discount approved! Deal: {{opp_name}}"
6  │   │
7  │   └── Action 2: Update Base via AnyCross/webhook
8  │       └── Opportunity.discount_approved = true
9  │
10 └── Trigger: Process Rejected
11     └── Action: Send Lark message to submitter
12             └── "Discount rejected. Reason: {{reject_reason}}"
```

### 9.4. Messenger Bot Integration

**Bot Capabilities:**

| Feature | Use Case | Setup |
|---------|----------|-------|
| Notifications | Alert on events | Base Automation → Send to Messenger |
| Quick Actions | Approve from chat | Message card with buttons |
| Lookup | Query data | Bot command + API |
| Reminders | Scheduled alerts | Automation + Bot |

**Notification Flow:**

```
1  Base Automation
2  │
3  ├── Trigger: Record created/updated
4  │
5  ├── Condition: Check rules
6  │
7  └── Action: Send to Messenger
8      ├── To: Person field / Specific user / Group
9      ├── Content: Template with {{field}} variables
10     └── Buttons: Links to Base record, Approval, etc.
```

### 9.5. Calendar Integration

**Use Cases:**

| Event Type | Trigger | Attendees | Notes |
|------------|---------|-----------|-------|
| Follow-up Call | Opp.next_action_date | Sales + Contact | Auto-create from Opp |
| Contract Signing | Contract created | AM + Customer | With contract link |
| Renewal Discussion | 60 days before end | AM + Customer | Reminder automation |
| Collection Call | Invoice overdue | AM | With invoice details |

**Automation:**

```
1  Trigger: Opportunity.next_action_date set
2  │
3  └── Action: Create Calendar Event
4      ├── Title: "Follow-up: {{opp_name}}"
5      ├── Date/Time: {{next_action_date}}
6      ├── Attendees: {{owner}}
7      ├── Description: "Action: {{next_action}}"
8      └── Link: [Opportunity record]
```

### 9.6. AnyCross Integration

**When to use AnyCross:**
- Connect to external systems (ERP, Accounting, Bank)
- Complex multi-app workflows
- Two-way data sync
- Scheduled data imports

**Example: Bank Statement Import**

```
1  AnyCross Workflow
2  │
3  ├── Trigger: Scheduled (Daily 9AM)
4  │
5  ├── Step 1: Fetch bank statement from Bank API
6  │   ├── API Call: GET /transactions
7  │   └── Filter: date = yesterday
8  │
9  └── Step 2: Loop through transactions
10     │
11     ├── Match with Invoices (by reference_number)
12     │
13     ├── IF matched:
14     │   ├── Create Payment record in Base
15     │   ├── Update Invoice status
16     │   └── Create Cash Flow IN
17     │
18     └── ELSE:
19         └── Create "Unmatched Payment" record for review
20
21 └── Step 3: Send summary to Finance
22     └── Messenger: "Daily bank import complete: X payments matched"
```

---

## 10. Kế hoạch Triển khai

### 10.1. Phương pháp Triển khai

**Approach:** Phased Rollout (Triển khai theo giai đoạn)

```
┌───────────────────────────────────────────────────────────────┐
│                  IMPLEMENTATION PHASES                        │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  Phase 1         Phase 2         Phase 3         Phase 4      │
│  FOUNDATION      SALES CRM       FINANCE         ADVANCED     │
│  (2 weeks)       (3 weeks)       (3 weeks)       (2 weeks)    │
│                                                               │
│  • Database      • Lead Mgmt     • Invoicing     • Dashboards │
│  • Permissions   • Opportunity   • Payment       • KPI Track  │
│  • Basic Views   • Account Mgmt  • Expense       • AnyCross   │
│  • Core Approval • Pipeline      • Cash Flow     • Optimize   │
└───────────────────────────────────────────────────────────────┘
```

### 10.2. Chi tiết Từng Phase

#### Phase 1: Foundation (Tuần 1-2)

| Tuần | Hoạt động | Deliverables |
|------|-----------|--------------|
| 1.1 | Setup Base structure | All 12 tables created |
| 1.1 | Import master data | Products, Staff, Teams |
| 1.2 | Configure permissions | Roles, access control |
| 1.2 | Basic Approval setup | Discount, Expense forms |
| 1.2 | UAT & Training | User acceptance, training session |

**Checklist Phase 1:**
- [ ] Tạo Base với 12 tables
- [ ] Thiết lập relationships giữa tables
- [ ] Import Products, Staff, Teams data
- [ ] Cấu hình Advanced Permissions
- [ ] Tạo basic views cho mỗi table
- [ ] Setup 2 Approval forms (Discount, Expense)
- [ ] Test với 2-3 users
- [ ] Training session cho pilot users

#### Phase 2: Sales CRM (Tuần 3-5)

| Tuần | Hoạt động | Deliverables |
|------|-----------|--------------|
| 3.1 | Lead management | Lead capture, assignment automation |
| 3.2 | Opportunity management | Pipeline views, stage automation |
| 4.1 | Account & Contact | Account views, contact linking |
| 4.2 | Sales automation | Notifications, reminders |
| 5.1 | Sales Portal App | Base App for sales team |
| 5.2 | UAT & Training | Pilot with sales team |

**Checklist Phase 2:**
- [ ] Lead assignment automation
- [ ] Lead scoring rules
- [ ] Pipeline Kanban view
- [ ] Stage change notifications
- [ ] Discount approval integration
- [ ] Account 360 view
- [ ] Sales Portal (Base App)
- [ ] Mobile testing
- [ ] Sales team training

#### Phase 3: Finance (Tuần 6-8)

| Tuần | Hoạt động | Deliverables |
|------|-----------|--------------|
| 6.1 | Contract management | Contract creation, lifecycle |
| 6.2 | Invoice generation | Auto-create from contract |
| 7.1 | Payment tracking | Payment recording, matching |
| 7.2 | Expense management | Expense claim workflow |
| 8.1 | Cash Flow | Cash flow tracking, reports |
| 8.2 | Finance Portal | Base App for finance team |

**Checklist Phase 3:**
- [ ] Contract lifecycle automation
- [ ] Invoice generation rules
- [ ] Payment approval workflow
- [ ] Expense claim approval
- [ ] Revenue recognition process
- [ ] Cash Flow IN/OUT automation
- [ ] AR Aging report
- [ ] Finance Portal (Base App)
- [ ] Finance team training

#### Phase 4: Advanced (Tuần 9-10)

| Tuần | Hoạt động | Deliverables |
|------|-----------|--------------|
| 9.1 | Dashboards | All 5 dashboards |
| 9.2 | KPI tracking | KPI automation, leaderboard |
| 10.1 | AnyCross integration | External system connection |
| 10.2 | Optimization | Performance tuning, feedback |

**Checklist Phase 4:**
- [ ] 5 Dashboards hoàn thiện
- [ ] KPI calculation automation
- [ ] Management Portal
- [ ] AnyCross workflow (if needed)
- [ ] Performance optimization
- [ ] Full UAT
- [ ] Go-live preparation
- [ ] Documentation hoàn thiện

### 10.3. Timeline Tổng thể

```
┌───────────────────────────────────────────────────────────────┐
│                      10-WEEK TIMELINE                         │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  Week: 1   2   3   4   5   6   7   8   9   10                 │
│        │   │   │   │   │   │   │   │   │   │                  │
│        ████████                               Phase 1         │
│              Foundation                                       │
│                                                               │
│              ██████████████████               Phase 2         │
│                    Sales CRM                                  │
│                                                               │
│                          ██████████████████   Phase 3         │
│                              Finance                          │
│                                                               │
│                                      ████████ Phase 4         │
│                                      Advanced                 │
│                                                               │
│  Milestones:                                                  │
│  ▲ M1: Foundation complete, pilot users trained               │
│  ▲ M2: Sales CRM live for sales team                          │
│  ▲ M3: Finance module live                                    │
│  ▲ M10: Full system live, all dashboards active               │
└───────────────────────────────────────────────────────────────┘
```

### 10.4. Resource Requirements

| Role | Effort | Phase |
|------|--------|-------|
| Project Manager | 50% | All |
| Lark Consultant | 100% | Phase 1-3, 60% Phase 4 |
| Sales Champion | 25% | Phase 1-2 |
| Finance Champion | 25% | Phase 3 |
| IT Support | 10% | All |

### 10.5. Risk Management

| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| Data migration issues | High | Medium | Cleanse data before import, validate |
| User adoption | High | Medium | Early involvement, training, champions |
| Scope creep | Medium | High | Clear scope document, change control |
| Integration delays | Medium | Low | Start simple, add integrations later |
| Performance issues | Medium | Low | Monitor usage, optimize as needed |

---

## 11. Vận hành & Cải tiến

### 11.1. Governance Model

```
┌───────────────────────────────────────────────────────────────┐
│                   GOVERNANCE STRUCTURE                        │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│                    ┌─────────────────┐                        │
│                    │    STEERING     │                        │
│                    │   COMMITTEE     │                        │
│                    │   (Monthly)     │                        │
│                    └────────┬────────┘                        │
│                             │                                 │
│          ┌──────────────────┼──────────────────┐              │
│          ▼                  ▼                  ▼              │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐       │
│  │ SYSTEM OWNER │   │   BUSINESS   │   │  IT SUPPORT  │       │
│  │ (Day-to-day) │   │  CHAMPIONS   │   │              │       │
│  │              │   │  (Per dept)  │   │              │       │
│  └──────────────┘   └──────────────┘   └──────────────┘       │
└───────────────────────────────────────────────────────────────┘
```

**Roles:**

| Role | Responsibility | Cadence |
|------|----------------|---------|
| Steering Committee | Strategic decisions, budget | Monthly |
| System Owner | Day-to-day management, training | Daily |
| Business Champions | Feedback collection, adoption | Weekly |
| IT Support | Technical issues, integrations | As needed |

### 11.2. Change Management

**Change Request Process:**

```
1  1. User submits change request (via form)
2  │
3  2. System Owner reviews
4  │
5  3. Classify change:
6     • Minor (field label, view) → Implement directly
7     • Standard (new field, new view) → Business Champion approval
8     • Major (new table, workflow change) → Steering Committee approval
9  │
10 4. Implement & Test
11 │
12 5. Deploy & Communicate
13 │
14 6. Monitor & Feedback
```

### 11.3. Monitoring & KPIs

**System Health Metrics:**

| Metric | Target | Measurement |
|--------|--------|-------------|
| User Adoption | >80% active weekly | Login tracking |
| Data Quality | <5% incomplete records | Field completion rate |
| Automation Success | >95% | Automation run logs |
| Response Time | <3 seconds | User feedback |

**Business KPIs:**

| KPI | Baseline | Target | Measurement |
|-----|----------|--------|-------------|
| Lead Response Time | 48h | <4h | Lead created → first contact |
| Pipeline Visibility | 80% | 95% | Opps with updated stages |
| Invoice Accuracy | 90% | 99% | Invoices without correction |
| Collection Rate | 80% | 95% | Paid within terms |
| Forecast Accuracy | 70% | 85% | Predicted vs Actual |

### 11.4. Continuous Improvement

**Feedback Loop:**

```
┌───────────────────────────────────────────────────────────────┐
│                    IMPROVEMENT CYCLE                          │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│       ┌─────────────┐                                         │
│       │   COLLECT   │ ← User feedback, usage data             │
│       │   FEEDBACK  │                                         │
│       └──────┬──────┘                                         │
│              │                                                │
│              ▼                                                │
│       ┌─────────────┐                                         │
│       │   ANALYZE   │ ← Identify patterns, pain points        │
│       └──────┬──────┘                                         │
│              │                                                │
│              ▼                                                │
│       ┌─────────────┐                                         │
│       │ PRIORITIZE  │ ← Impact vs Effort matrix               │
│       └──────┬──────┘                                         │
│              │                                                │
│              ▼                                                │
│       ┌─────────────┐                                         │
│       │  IMPLEMENT  │ ← Sprint/iteration                      │
│       └──────┬──────┘                                         │
│              │                                                │
│              ▼                                                │
│       ┌─────────────┐                                         │
│       │   MEASURE   │ ← Track improvement                     │
│       └──────┬──────┘                                         │
│              │                                                │
│              └────────────────── (Loop back to COLLECT)       │
└───────────────────────────────────────────────────────────────┘
```

**Sprint Cadence:**

| Activity | Frequency |
|----------|-----------|
| Collect Feedback | Ongoing |
| Review & Prioritize | Bi-weekly |
| Implement Changes | 2-week sprints |
| Release Updates | Bi-weekly |
| Major Reviews | Quarterly |

### 11.5. Documentation & Training

**Documentation Artifacts:**

| Document | Owner | Update Frequency |
|----------|-------|------------------|
| User Guide | System Owner | Per release |
| Admin Guide | System Owner | Per release |
| Process Flows | Business Champion | Quarterly |
| Training Materials | System Owner | Per release |
| FAQ | System Owner | Weekly |

**Training Program:**

| Audience | Format | Duration | Frequency |
|----------|--------|----------|-----------|
| New Users | Onboarding session | 2 hours | On hire |
| All Users | Feature updates | 30 min | Per release |
| Champions | Deep dive | 4 hours | Quarterly |
| Managers | Dashboard training | 1 hour | Quarterly |

---

## Phụ lục

### A. Glossary (Thuật ngữ)

| Thuật ngữ | Định nghĩa |
|-----------|------------|
| Lead | Khách hàng tiềm năng chưa qualified |
| Account | Công ty khách hàng |
| Contact | Người liên hệ tại Account |
| Opportunity | Cơ hội bán hàng đang theo đuổi |
| Pipeline | Tập hợp các Opportunities |
| AR | Accounts Receivable - Công nợ phải thu |
| DSO | Days Sales Outstanding - Số ngày thu tiền trung bình |
| KPI | Key Performance Indicator - Chỉ số hiệu suất chính |
| Base App | Ứng dụng UI/UX xây dựng từ Lark Base |
| Workflow | Quy trình tự động hóa trong Lark Base |

### B. Checklist Triển khai

**Pre-Go-Live Checklist:**
- [ ] Tất cả tables đã tạo và test
- [ ] Relationships hoạt động đúng
- [ ] Permissions đã cấu hình cho tất cả roles
- [ ] Automation rules đã test
- [ ] Approval workflows đã test
- [ ] Dashboards hiển thị đúng data
- [ ] Base App pages hoạt động
- [ ] Mobile testing hoàn thành
- [ ] Data migration hoàn thành
- [ ] User training hoàn thành
- [ ] Documentation hoàn thành
- [ ] Support process sẵn sàng