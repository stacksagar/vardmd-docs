# VardMD Pharmacy Portal - Software Requirements Specification (SRS)

**Project:** VardMD Pharmacy Portal  
**Version:** 1.0  
**Date:** January 4, 2026  
**Status:** Approved for Development

---

## 1. Introduction

### 1.1 Purpose

This Software Requirements Specification (SRS) describes the complete software requirements for the VardMD Pharmacy Portal. It is intended for developers, testers, project managers, and stakeholders to understand the system's functionality, constraints, and interfaces.

### 1.2 Scope

The **VardMD Pharmacy Portal** is a web-based pharmacy management system that enables:

- Pharmacists to process prescriptions safely with automated drug interaction checking
- Real-time inventory management with batch tracking and expiry monitoring
- Automated compliance documentation and audit trails
- Purchase order management and supplier relationships
- Financial tracking and reporting

**Benefits:**

- Reduce medication errors from 5-10% to <1%
- Decrease prescription processing time from 15 min to 5 min
- Increase inventory accuracy from 85% to 98%
- Automate regulatory compliance documentation

### 1.3 Definitions, Acronyms, and Abbreviations

| Term            | Definition                                |
| --------------- | ----------------------------------------- |
| API             | Application Programming Interface         |
| FIFO            | First In, First Out (inventory method)    |
| JWT             | JSON Web Token (authentication)           |
| MRN             | Medical Record Number                     |
| OTC             | Over-The-Counter (no prescription needed) |
| PCN             | Pharmacy Council of Nigeria               |
| PRN             | As needed (Pro Re Nata)                   |
| Rx              | Prescription                              |
| SPA             | Single Page Application                   |
| STAT            | Immediately (urgent)                      |
| TID / BID / QID | Three/Twice/Four times daily              |

### 1.4 References

- VardMD Database Schema Documentation
- VardMD Pharmacy API Guide (1019 lines)
- Pharmacy Portal Design Brief
- Pharmacy Portal Functional Requirements Document
- Pharmacy Portal Product Requirements Document
- Nigerian Pharmacy Council Regulations
- WHO Guidelines for Safe Medication Practices

### 1.5 Overview

This SRS is organized into:

- Section 2: Overall system description
- Section 3: Specific functional requirements
- Section 4: External interface requirements
- Section 5: Non-functional requirements
- Section 6: Database requirements
- Section 7: Security requirements

---

## 2. Overall Description

### 2.1 Product Perspective

The Pharmacy Portal is a component of the larger VardMD healthcare ecosystem:

```
┌─────────────────────────────────────────────────┐
│          VardMD Healthcare Platform             │
├─────────────────────────────────────────────────┤
│                                                 │
│  Provider Portal ──► Pharmacy Portal ──► Patient│
│         │                   │                   │
│         └──► EMR Portal ────┘                   │
│                                                 │
│  Super Admin Portal (Monitoring & Reports)      │
└─────────────────────────────────────────────────┘
```

**System Interfaces:**

- **Input:** Prescriptions from Provider Portal and EMR Portal
- **Output:** Dispensing notifications to patients, audit logs to regulators
- **External:** Drug interaction database API, supplier systems (email-based initially)

### 2.2 Product Functions

Major functions include:

1. **Prescription Management**

   - Receive digital prescriptions from doctors
   - Queue management with priority handling
   - Drug interaction and allergy checking
   - Medication dispensing workflow
   - Patient counseling documentation

2. **Inventory Management**

   - Real-time stock level tracking
   - Batch number tracking (FIFO)
   - Expiry date monitoring
   - Low stock alerts
   - Stock adjustments and reconciliation

3. **Procurement**

   - Supplier database management
   - Purchase order creation and tracking
   - Shipment receiving and verification
   - Automatic inventory updates

4. **Financial Management**

   - Revenue and cost tracking
   - Profit margin analysis
   - Payment processing
   - Insurance claim documentation

5. **Reporting & Analytics**
   - Dispensing analytics
   - Inventory reports
   - Financial performance
   - Regulatory compliance reports

### 2.3 User Characteristics

| User Type               | Characteristics                                  | Technical Expertise                   | Usage Pattern                              |
| ----------------------- | ------------------------------------------------ | ------------------------------------- | ------------------------------------------ |
| **Pharmacist**          | Licensed pharmacist, 25-45 years, B.Pharm degree | Moderate (smartphone, basic computer) | Daily, 8-10 hours, 30-50 prescriptions/day |
| **Pharmacy Admin**      | Manager/owner, 30-55 years, B.Pharm + management | Moderate to High                      | Daily, focus on reports and inventory      |
| **Pharmacy Technician** | Support staff, 20-40 years, diploma              | Low to Moderate                       | Daily, assist with inventory and OTC sales |

### 2.4 Constraints

**Regulatory Constraints:**

- MUST comply with Nigerian Pharmacy Council (PCN) regulations
- MUST maintain 7-year record retention
- MUST log all controlled substance dispensing
- MUST have licensed pharmacist verify prescriptions

**Technical Constraints:**

- MUST use PostgreSQL database (existing VardMD infrastructure)
- MUST use Prisma ORM (consistency with other portals)
- MUST support modern browsers (Chrome, Firefox, Safari, Edge)
- MUST be responsive (desktop, tablet, mobile)

**Business Constraints:**

- Multi-tenant architecture required (data isolation per pharmacy)
- 4-week development timeline for Phase 1
- Budget constraints (use existing VardMD infrastructure)

**Security Constraints:**

- MUST encrypt data at rest (AES-256) and in transit (HTTPS/TLS 1.2+)
- MUST implement role-based access control
- MUST log all critical actions for audit
- MUST prevent unauthorized cross-tenant access

### 2.5 Assumptions and Dependencies

**Assumptions:**

- Internet connectivity available during pharmacy operating hours
- Pharmacists have basic computer literacy
- Drug interaction database API is available and maintained
- Prescriptions from Provider Portal contain complete information
- Barcode scanners available for batch number scanning

**Dependencies:**

- VardMD authentication and authorization system
- Provider Portal operational (source of prescriptions)
- EMR Portal operational (hospital medication orders)
- Third-party drug interaction database API
- DigitalOcean infrastructure availability
- PostgreSQL database availability

---

## 3. Specific Functional Requirements

### 3.1 User Authentication and Authorization

**FR-AUTH-001: User Login**

- **Description:** System shall authenticate users via email and password
- **Input:** Email address, password
- **Processing:** Validate credentials, generate JWT token
- **Output:** JWT token with expiry (8 hours for pharmacist, 12 hours for admin)
- **Priority:** CRITICAL

**FR-AUTH-002: Role-Based Access Control**

- **Description:** System shall enforce role-based permissions
- **Roles:**
  - `PHARMACY`: Can process prescriptions, dispense, view inventory
  - `PHARMACY_ADMIN`: All PHARMACY permissions + manage suppliers, POs, financial reports
- **Priority:** CRITICAL

**FR-AUTH-003: Session Management**

- **Description:** System shall manage user sessions
- **Processing:**
  - Auto-refresh token before expiry
  - Logout on inactivity after 8 hours
  - Lock account after 5 failed login attempts (15 min cooldown)
- **Priority:** HIGH

### 3.2 Dashboard

**FR-DASH-001: Dashboard Statistics**

- **Description:** System shall display real-time pharmacy statistics
- **Output:**
  - Pending prescriptions count
  - Prescriptions dispensed today
  - Low stock items count
  - Total inventory value (₦)
  - Today's revenue
- **Update Frequency:** Every 5 minutes or on-demand refresh
- **Priority:** HIGH

**FR-DASH-002: Alerts Summary**

- **Description:** System shall display critical alerts
- **Alert Types:**
  - Low stock (quantity < reorder level)
  - Expiring medications (within 90, 30, 7 days)
  - STAT prescriptions pending
  - System errors
- **Priority:** HIGH

### 3.3 Prescription Queue Management

**FR-RX-001: Prescription Queue Display**

- **Description:** System shall display all prescriptions in a prioritized queue
- **Display Fields:**
  - Rx number, patient name/MRN, provider name
  - Status, priority, date/time prescribed
  - Billing status
- **Sorting:** STAT > Urgent > Routine, then by date (oldest first)
- **Filtering:** By status, priority, date range, search by patient/Rx number
- **Pagination:** 20, 50, 100 items per page
- **Priority:** CRITICAL

**FR-RX-002: Prescription Detail View**

- **Description:** System shall display complete prescription details
- **Patient Info:** Name, age, gender, MRN, allergies, current medications, insurance
- **Prescription Info:** Provider, date, diagnosis, consultation link
- **Medications:** For each item:
  - Drug name (brand/generic), strength, form
  - Dosage, frequency, duration, quantity
  - Instructions, price
- **Priority:** CRITICAL

**FR-RX-003: Prescription Status Workflow**

- **Description:** System shall enforce status transitions
- **Valid Transitions:**
  ```
  pending → validated → ready_for_dispense → dispensed → completed
     ↓
  queried
     ↓
  rejected
  ```
- **Rules:**
  - Cannot skip statuses
  - Cannot move backward
  - Status changes logged with timestamp and user
- **Priority:** CRITICAL

### 3.4 Drug Interaction and Safety Checking

**FR-SAFETY-001: Automatic Interaction Check**

- **Description:** System shall automatically check drug interactions
- **When:** When pharmacist reviews prescription (clicks "Review")
- **Checks:**
  - Drug-drug interactions (new meds vs. current meds)
  - Drug-allergy interactions (meds vs. patient allergies)
  - Duplicate therapy
  - Dosage validation (within safe range)
- **Data Source:** Third-party drug interaction database API
- **Performance:** Complete within 3 seconds
- **Priority:** CRITICAL (SAFETY)

**FR-SAFETY-002: Interaction Severity Levels**

- **Description:** System shall categorize interactions by severity
- **Severity Levels:**
  - **Critical:** Life-threatening, absolutely contraindicated (RED alert)
  - **Major:** Serious, requires intervention (ORANGE alert)
  - **Moderate:** Monitor patient (YELLOW alert)
  - **Minor:** Generally acceptable (INFO)
- **Display:** Color-coded alerts with clear descriptions and recommendations
- **Priority:** CRITICAL

**FR-SAFETY-003: Interaction Override Workflow**

- **Description:** System shall allow overriding critical interactions with approval
- **Requirements:**
  - Documented reason (text field, mandatory)
  - Prescriber contacted and approved (checkbox)
  - Senior pharmacist approval (digital signature)
  - Patient informed consent (checkbox)
- **Audit:** All overrides logged with full details
- **Priority:** CRITICAL

**FR-SAFETY-004: Allergy Alert System**

- **Description:** System shall display prominent allergy warnings
- **Display:** Red banner across prescription detail screen
- **Information:** Allergen, reaction history, alternatives
- **Cannot Dismiss:** Acknowledgment required before proceeding
- **Priority:** CRITICAL

**FR-SAFETY-005: Interaction History Log**

- **Description:** System shall maintain audit log of all interaction checks
- **Log Fields:** Date/time, prescription ID, pharmacist, alerts found, actions taken
- **Retention:** 7 years minimum
- **Export:** CSV format for regulatory inspection
- **Priority:** HIGH

### 3.5 Medication Dispensing

**FR-DISP-001: Dispensing Workflow**

- **Description:** System shall provide structured 6-step dispensing workflow
- **Steps:**
  1. Patient verification (ID check)
  2. Safety check (interactions, allergies)
  3. Medication preparation (select batch via FIFO)
  4. Patient counseling (mandatory checklist)
  5. Payment processing
  6. Finalization (signature, print label/receipt)
- **Rules:** Steps completed in order, counseling cannot be skipped
- **Priority:** CRITICAL

**FR-DISP-002: Batch Selection (FIFO)**

- **Description:** System shall auto-select medication batch using FIFO
- **Logic:** Earliest expiry date first
- **Validation:**
  - Expiry > 30 days away (warning 7-30 days)
  - Expiry < 7 days (block dispensing)
- **Override:** Admin can override FIFO with documented reason
- **Priority:** HIGH

**FR-DISP-003: Patient Counseling Checklist**

- **Description:** System shall display mandatory counseling points
- **Checklist Items:**
  - How to take (dosage, frequency, timing)
  - Common side effects
  - What to do if dose missed
  - Storage instructions
  - When to follow up with doctor
- **Validation:** All checkboxes must be checked before proceeding
- **Priority:** HIGH

**FR-DISP-004: Payment Processing**

- **Description:** System shall calculate and record payment
- **Calculation:**
  - Total amount = Σ(item price × quantity)
  - Insurance coverage (if applicable)
  - Patient co-pay = Total - Insurance
- **Payment Methods:** Cash, card, insurance, subscription
- **Receipt:** Auto-generate with receipt number
- **Priority:** HIGH

**FR-DISP-005: Automatic Inventory Deduction**

- **Description:** System shall automatically update inventory upon dispensing
- **Processing:**
  - Deduct quantity from selected batch
  - Create inventory transaction (type: DISPENSING)
  - Create dispensing record (audit trail)
  - Check if stock < reorder level → trigger alert
- **Atomicity:** All or nothing (database transaction)
- **Priority:** CRITICAL

**FR-DISP-006: Partial Dispensing**

- **Description:** System shall support partial fills when stock insufficient
- **Workflow:**
  - Dispense available quantity
  - Calculate adjusted payment
  - Create backorder for remaining quantity
  - Notify patient when backorder fulfilled
- **Priority:** MEDIUM

### 3.6 Inventory Management

**FR-INV-001: Real-Time Inventory Tracking**

- **Description:** System shall track inventory in real-time
- **Data per Drug:**
  - Drug name, code, category
  - Current quantity, reorder level, max level
  - Average monthly usage
- **Data per Batch:**
  - Batch number, quantity, expiry date
  - Cost price, selling price
  - Supplier, date received, storage location
- **Update Triggers:** Dispensing, receiving, adjustments
- **Priority:** CRITICAL

**FR-INV-002: Low Stock Alerts**

- **Description:** System shall automatically generate low stock alerts
- **Trigger Conditions:**
  - Quantity < reorder level
  - Quantity < 5 days average usage
  - Only one batch remaining and partially used
- **Alert Display:** Dashboard badge, alert list with details
- **Actions:** Quick create PO, snooze, mark as ordered
- **Notification:** Daily email summary at 8:00 AM
- **Priority:** HIGH

**FR-INV-003: Expiry Date Monitoring**

- **Description:** System shall monitor and alert on expiring medications
- **Alert Thresholds:**
  - 90 days: Warning (yellow) - discount pricing recommended
  - 30 days: Urgent (orange) - return to supplier if possible
  - 7 days: Critical (red) - remove from dispensing inventory
  - Expired: Auto-flag for disposal
- **Enforcement:** Cannot dispense if expiry < 7 days
- **Priority:** HIGH

**FR-INV-004: Stock Adjustment**

- **Description:** System shall allow manual inventory adjustments
- **Adjustment Types:**
  - Damaged, expired, stolen/lost, returned, supplier credit, stock count correction
- **Required Fields:**
  - Drug/batch, quantity (+/-), type, reason, authorized by
- **Validation:**
  - Adjustments >10 units or >₦10,000 require admin approval
- **Audit:** Create inventory transaction record
- **Priority:** MEDIUM

**FR-INV-005: Inventory Valuation**

- **Description:** System shall calculate inventory value
- **Methods:**
  - At cost = Σ(Quantity × Cost Price)
  - At retail = Σ(Quantity × Selling Price)
  - Potential profit = Retail - Cost
- **Breakdown:** By category, by supplier
- **Historical:** Monthly trend comparison
- **Priority:** MEDIUM

**FR-INV-006: Stock Movement History**

- **Description:** System shall track all inventory transactions
- **Transaction Types:**
  - RECEIPT (from supplier)
  - DISPENSING (to patient)
  - ADJUSTMENT (damaged, expired, etc.)
  - RETURN (patient returned)
- **Log Fields:** Type, quantity, balance after, reason, user, timestamp
- **Query:** By drug, by date range, by type
- **Priority:** HIGH

### 3.7 Drug Formulary Management

**FR-FORM-001: Formulary Database**

- **Description:** System shall maintain comprehensive drug formulary
- **Data Fields:**
  - Drug code, brand/generic name, active ingredient
  - Strength, form, route, manufacturer
  - Therapeutic category, indications, contraindications
  - Cost price, selling price, profit margin
  - Prescription required (yes/no), substitutable (yes/no)
  - Active/inactive status
- **Multi-Tenant:** Each organization has own formulary
- **Priority:** HIGH

**FR-FORM-002: Formulary CRUD Operations**

- **Description:** System shall allow add/edit/deactivate drugs (PHARMACY_ADMIN only)
- **Operations:**
  - Add: All fields, validate uniqueness of code
  - Edit: Update any field, log price changes
  - Deactivate: Soft delete (mark inactive, keep history)
  - Delete: Not allowed (maintain audit trail)
- **Priority:** MEDIUM

**FR-FORM-003: Generic Substitution Lookup**

- **Description:** System shall recommend generic alternatives
- **When:** Brand drug out of stock or pharmacist requests
- **Display:**
  - Original (brand name, price)
  - Alternatives (generic name, manufacturer, price, savings %, bioequivalent status, stock availability)
- **Actions:** Call prescriber for approval, log substitution
- **Priority:** MEDIUM

**FR-FORM-004: Formulary Search**

- **Description:** System shall provide search functionality
- **Search Fields:** Name, code, category, manufacturer
- **Filters:** Prescription required, substitutable, active/inactive
- **Autocomplete:** Type-ahead suggestions
- **Priority:** MEDIUM

### 3.8 Supplier and Purchase Order Management

**FR-SUPP-001: Supplier Database**

- **Description:** System shall maintain supplier information (PHARMACY_ADMIN only)
- **Data Fields:**
  - Name, contact person, phone, email, address
  - Payment terms, minimum order, delivery time, return policy
  - Performance metrics (on-time %, quality issues, rating)
- **Status:** Active, Inactive, Blacklisted
- **Priority:** LOW

**FR-SUPP-002: Purchase Order Creation**

- **Description:** System shall allow creating purchase orders
- **Header:** PO number (auto-gen), supplier, order/delivery dates, payment terms
- **Line Items:** Drug, quantity, unit price, total
  - Suggested quantity = (Reorder level - Current stock) + Safety stock
- **Calculations:** Subtotal, VAT (7.5%), shipping, grand total
- **Validation:**
  - Minimum order check
  - Price variance alert (>10% from last order)
  - Manager approval required if >₦100,000
- **Actions:** Save draft, submit (email to supplier), cancel
- **Priority:** MEDIUM

**FR-SUPP-003: Shipment Receiving**

- **Description:** System shall provide receiving workflow
- **Steps:**
  1. Select PO
  2. Verify each item (drug match, quantity, batch, expiry, condition)
  3. Flag discrepancies (short ship, wrong item, damaged)
  4. Finalize (update inventory, create GRN, update PO status)
- **Validation:**
  - Cannot receive expired items
  - Cannot receive >100% of ordered quantity without approval
  - Expiry must be >6 months away
- **Auto-Triggers:**
  - Inventory updated
  - Low stock alerts cleared if applicable
  - Payment scheduled if terms are immediate
- **Priority:** MEDIUM

**FR-SUPP-004: Supplier Performance Tracking**

- **Description:** System shall track supplier metrics
- **Metrics per Supplier:**
  - Order count and total value
  - On-time delivery rate (%)
  - Average delivery time (days)
  - Quality issues count
  - Return rate (%)
- **Ranking:** By reliability, cost, quality
- **Report:** Quarterly supplier performance report
- **Priority:** LOW

### 3.9 Reporting and Analytics

**FR-REPORT-001: Dispensing Analytics Report**

- **Description:** System shall generate dispensing analytics
- **Metrics:**
  - Total prescriptions dispensed (by day/week/month)
  - Average prescriptions per day
  - Breakdown by category, provider, payment method, priority
- **Charts:** Line (trends), pie (distribution), bar (top 10 drugs)
- **Export:** PDF, Excel
- **Priority:** MEDIUM

**FR-REPORT-002: Inventory Report**

- **Description:** System shall generate inventory reports
- **Report Types:**
  - Stock levels (current quantities, values)
  - Movement report (opening, receipts, dispensed, closing)
  - Slow-moving items (<5 units sold in 90 days)
  - Fast-moving items (>50 units sold in 30 days)
- **Filters:** Category, supplier, date range
- **Priority:** MEDIUM

**FR-REPORT-003: Financial Performance Report**

- **Description:** System shall generate financial reports
- **Metrics:**
  - Total revenue (prescriptions + OTC)
  - Cost of goods sold (COGS)
  - Gross profit and margin %
  - Breakdown by payment method, category
- **Calculations:**
  - COGS = Σ(Cost price × Quantity dispensed)
  - Gross profit = Revenue - COGS
  - Margin % = (Gross profit / Revenue) × 100
- **Comparison:** Month-over-month, year-over-year
- **Priority:** MEDIUM

**FR-REPORT-004: Audit and Compliance Report**

- **Description:** System shall generate regulatory compliance reports
- **Reports:**
  - Dispensing audit trail (all prescriptions with details)
  - Drug interaction overrides log
  - Controlled substances log (receipts, dispensing, adjustments)
- **Fields:** Patient, drug, pharmacist, date/time, batch number
- **Retention:** 7 years minimum
- **Export:** CSV for regulatory inspection
- **Priority:** HIGH

**FR-REPORT-005: Report Scheduling**

- **Description:** System shall support automated report generation
- **Schedule:** Daily, weekly, monthly
- **Delivery:** Email to specified recipients
- **Format:** PDF or Excel attachment
- **Priority:** LOW

### 3.10 Walk-In Sales (Over-The-Counter)

**FR-OTC-001: OTC Sales Processing**

- **Description:** System shall support walk-in sales for OTC medications
- **Workflow:**
  1. Search OTC drugs (filter: prescription not required)
  2. Add items to cart (multiple items, adjust quantities)
  3. Apply discounts (if authorized)
  4. Pharmacist counseling (prompted for specific drugs)
  5. Process payment
  6. Print receipt
- **Inventory:** Auto-deduct from stock
- **Limits:** Quantity limits on certain OTC drugs (e.g., max 20 paracetamol)
- **Priority:** LOW

---

## 4. External Interface Requirements

### 4.1 User Interfaces

**UI-001: Responsive Design**

- System shall be fully responsive (desktop ≥1920px, tablet 768-1024px, mobile 375-767px)
- Navigation: Desktop (sidebar), Mobile (hamburger menu)
- Critical actions accessible within 3 clicks

**UI-002: Design System**

- Use shadcn/ui components (consistency with VardMD ecosystem)
- Tailwind CSS for styling
- Dark mode support (toggle in settings)
- Color coding:
  - Red: Urgent, critical alerts, errors
  - Amber: Warnings, low stock, expiring
  - Green: Success, adequate stock
  - Blue: Info, normal workflow

**UI-003: Accessibility**

- WCAG 2.1 Level AA compliance
- Screen reader compatible (ARIA labels)
- Keyboard navigation support
- Minimum contrast ratio 4.5:1
- Text resize up to 200%

**UI-004: Performance**

- Initial load: <3 seconds
- Page transitions: <1 second
- Dashboard refresh: <2 seconds
- Interaction check: <3 seconds

### 4.2 Hardware Interfaces

**HW-001: Barcode Scanner**

- Support USB barcode scanners (keyboard wedge mode)
- Scan batch numbers during receiving
- Scan drug codes during dispensing (optional)

**HW-002: Receipt Printer**

- Support standard thermal receipt printers
- Print receipt upon payment
- Print medication labels

**HW-003: Mobile Device Camera** (Future)

- Scan barcodes using phone/tablet camera
- Capture photos of damaged goods

### 4.3 Software Interfaces

**SI-001: VardMD Authentication Service**

- Interface: REST API
- Protocol: HTTPS
- Authentication: JWT tokens
- Endpoints:
  - POST /auth/login
  - POST /auth/refresh
  - POST /auth/logout

**SI-002: Drug Interaction Database API**

- Interface: REST API (third-party)
- Protocol: HTTPS
- Authentication: API key
- Endpoint: POST /check-interactions
- Request: Patient meds + new meds
- Response: Interaction alerts with severity
- SLA: 99% uptime, <3 second response

**SI-003: Provider Portal**

- Interface: Shared PostgreSQL database
- Data Flow: Provider creates Prescription → Pharmacy receives
- Real-time: WebSocket notifications for new prescriptions

**SI-004: EMR Portal**

- Interface: Shared PostgreSQL database
- Data Flow: EMR creates hospital medication orders → Pharmacy receives
- Similar prescription structure as Provider Portal

**SI-005: Email Service**

- Interface: SMTP
- Purpose: Send POs to suppliers, daily alerts to admins
- Provider: SendGrid or similar

### 4.4 Communication Interfaces

**COM-001: HTTPS/TLS**

- All API communication over HTTPS
- TLS 1.2 or higher
- Certificate validation required

**COM-002: WebSocket**

- Real-time notifications (new prescriptions, stock alerts)
- Fallback to polling if WebSocket unavailable

**COM-003: REST API**

- JSON request/response format
- Standard HTTP methods (GET, POST, PATCH, DELETE)
- RESTful URL structure

---

## 5. Non-Functional Requirements

### 5.1 Performance Requirements

**PERF-001: Response Time**

- Dashboard load: <2 seconds
- Prescription queue: <2 seconds
- Drug interaction check: <3 seconds
- API calls: <1 second average
- Real-time updates: <1 second latency

**PERF-002: Throughput**

- Support 50 concurrent users per pharmacy
- Process 1000 prescriptions/day per pharmacy
- 100 transactions/second during peak

**PERF-003: Resource Usage**

- Database queries optimized (indexes on tenantId, status, date)
- Caching for static data (formulary, suppliers)
- Lazy loading for large lists (pagination)

### 5.2 Safety Requirements

**SAFE-001: Medication Safety**

- Drug interaction check MUST complete before dispensing
- Critical interactions MUST block dispensing (override requires approval)
- Expired medications MUST be blocked from dispensing
- All safety overrides MUST be logged

**SAFE-002: Data Backup**

- Automated backups every 6 hours
- Backup retention: 30 days
- Point-in-time recovery capability

**SAFE-003: Disaster Recovery**

- RTO (Recovery Time Objective): <4 hours
- RPO (Recovery Point Objective): <1 hour

### 5.3 Security Requirements

**SEC-001: Authentication**

- JWT-based authentication
- Password hashing: bcrypt (minimum 10 rounds)
- Session timeout: 8 hours (pharmacist), 12 hours (admin)
- Account lockout: 5 failed attempts, 15 min cooldown
- Password complexity: Min 8 characters, uppercase, lowercase, number, special char

**SEC-002: Authorization**

- Role-based access control (RBAC)
- Permissions enforced at API and UI layers
- Admin-only routes protected

**SEC-003: Data Encryption**

- At rest: AES-256 encryption
- In transit: HTTPS/TLS 1.2+
- Database: Encrypted backups

**SEC-004: Audit Logging**

- Log all critical actions (dispensing, inventory changes, overrides)
- Logs immutable (append-only, no deletion)
- Log fields: User, action, entity, timestamp, metadata

**SEC-005: Multi-Tenant Isolation**

- All queries filter by tenantId
- Cross-tenant access strictly prohibited
- Tenant data cannot be accessed by other tenants

**SEC-006: Input Validation**

- Frontend: Immediate user feedback
- API: Prevent SQL injection, XSS attacks
- Database: Constraints as final safeguard

### 5.4 Software Quality Attributes

**QUAL-001: Reliability**

- System uptime: 99.5% (max 3.6 hours downtime/month)
- Error handling: Graceful degradation
- Failover: Database replicas for high availability

**QUAL-002: Maintainability**

- Code quality: TypeScript strict mode, ESLint
- Documentation: Inline comments, API docs (OpenAPI/Swagger)
- Testing: Unit (>70% coverage), integration, end-to-end

**QUAL-003: Usability**

- Intuitive UI (new user onboarding <30 min)
- Error messages: Clear, actionable (no technical jargon)
- Keyboard shortcuts for frequent actions
- Help tooltips and context-sensitive help

**QUAL-004: Scalability**

- Horizontal scaling (add more servers)
- Database read replicas for reporting
- Support 100+ tenants without redesign
- Caching for frequently accessed data

**QUAL-005: Portability**

- Cross-browser compatibility (Chrome, Firefox, Safari, Edge)
- Mobile-responsive design
- No proprietary dependencies (use open-source where possible)

---

## 6. Database Requirements

### 6.1 Database Schema

**Tables:**

- Tenant, User, Member, Organization (core entities)
- Prescription, PrescriptionItem (prescription management)
- Formulary, PharmacyInventory, InventoryTxn (inventory)
- Dispensing (audit trail)
- Supplier, PurchaseOrder (procurement - future phase)
- AuditLog, Notification (system)

**Relationships:**

- Prescription 1:N PrescriptionItem
- PrescriptionItem N:1 Formulary
- PrescriptionItem 1:N Dispensing
- PharmacyInventory 1:N InventoryTxn
- PharmacyInventory N:1 Formulary

**Indexes:**

- tenantId (all tenant-scoped tables)
- status, prescribedAt (Prescription)
- formularyId, batchNo (PharmacyInventory)
- dispensedAt (Dispensing)

### 6.2 Data Integrity

**Constraints:**

- Primary keys: UUID (not auto-increment for multi-tenant security)
- Foreign keys: Cascade delete for tenants, restrict for prescriptions
- Unique constraints: tenantId + mrn, organizationId + formularyId + batchNo
- Check constraints: quantity >= 0, expiresOn > today

**Transactions:**

- Dispensing: Atomic (update prescription, create dispensing records, update inventory)
- Receiving: Atomic (update PO, create inventory, create transactions)
- ACID compliance required

### 6.3 Data Retention

**Retention Policies:**

- Prescriptions: 7 years (regulatory requirement)
- Dispensing records: 7 years
- Inventory transactions: 7 years
- Audit logs: 7 years
- User data: As long as account active + 1 year after deletion

**Archival:**

- Data older than 2 years moved to archive database (performance)
- Archive accessible for reporting and audits

---

## 7. Security Requirements

### 7.1 Authentication and Authorization

**AUTH-SEC-001: Multi-Factor Authentication** (Future)

- Optional 2FA via SMS or authenticator app
- Required for admin accounts

**AUTH-SEC-002: Password Policy**

- Minimum 8 characters
- Mix of uppercase, lowercase, numbers, special characters
- Cannot reuse last 3 passwords
- Password reset via email with 1-hour expiry token

**AUTH-SEC-003: License Verification**

- Pharmacist license number validated during onboarding
- License expiry tracked, alerts 90 days before
- Cannot dispense if license expired

### 7.2 Data Protection

**DATA-SEC-001: Encryption**

- Patient data encrypted at rest (AES-256)
- SSL/TLS for all network traffic
- Encrypted database backups

**DATA-SEC-002: Data Privacy**

- Comply with Nigeria Data Protection Regulation (NDPR)
- Patient consent for data processing
- Right to access (patient can request their data)
- Right to erasure (with regulatory exceptions)

**DATA-SEC-003: Audit Trail**

- All access to patient data logged
- Log: Who, what, when, why
- Logs protected from tampering

### 7.3 Network Security

**NET-SEC-001: Firewall**

- Database not directly accessible from internet
- Only application servers can connect to database
- Rate limiting on API endpoints (prevent DDoS)

**NET-SEC-002: CORS Policy**

- Whitelist allowed origins (VardMD domains only)
- No cross-origin requests from unknown domains

---

## 8. System Models

### 8.1 Context Diagram

```
┌─────────────────────────────────────────────────────────┐
│                  External Systems                       │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Provider Portal ──► [Prescriptions]                    │
│  EMR Portal ──────► [Medication Orders]                 │
│  Drug DB API ────► [Interaction Checks]                 │
│  Suppliers ──────► [Email: POs, Shipments]              │
│  Regulatory ◄──── [Audit Reports]                       │
│                                                         │
└───────────────┬─────────────────────────────────────────┘
                │
                ▼
        ┌───────────────────┐
        │  PHARMACY PORTAL  │
        ├───────────────────┤
        │ - Auth & RBAC     │
        │ - Prescription Rx │
        │ - Inventory Mgmt  │
        │ - Dispensing      │
        │ - Reporting       │
        └───────────────────┘
                │
                ▼
        ┌───────────────────┐
        │  Database (PG)    │
        │  Shared Data      │
        └───────────────────┘
```

### 8.2 Use Case Diagram

```
                   Pharmacist
                      │
        ┌─────────────┼─────────────┐
        │             │             │
    Review Rx    Dispense Med   Check Stock
        │             │             │
        └─────────────┼─────────────┘
                      │
                   System
                      │
        ┌─────────────┼─────────────┐
        │             │             │
  Check Interact  Update Inv   Generate Report
        │             │             │
        └─────────────┼─────────────┘
                      │
                Pharmacy Admin
```

### 8.3 State Diagram (Prescription Status)

```
    [New Prescription]
           │
           ▼
      ┌─────────┐
      │ pending │
      └────┬────┘
           │ Review
           ▼
      ┌──────────┐
      │validated │
      └────┬─────┘
           │ Prepare
           ▼
   ┌─────────────────┐
   │ready_for_dispense│
   └────────┬─────────┘
           │ Dispense
           ▼
      ┌──────────┐
      │dispensed │
      └────┬─────┘
           │ Finalize
           ▼
      ┌─────────┐
      │completed│
      └─────────┘

   Failure Paths:
   pending ──► queried ──► rejected
```

---

## 9. Validation and Verification

### 9.1 Validation Methods

**Unit Testing:**

- Backend services: Jest framework
- Coverage target: >70%
- Focus: Business logic, calculations, validations

**Integration Testing:**

- API endpoint testing: Supertest
- Database integration: Test database
- External API mocking: Nock

**End-to-End Testing:**

- User workflows: Playwright or Cypress
- Critical paths: Prescription processing, dispensing
- Cross-browser testing

**User Acceptance Testing:**

- Pilot pharmacies test in production-like environment
- Real prescriptions (sanitized data)
- Feedback via structured questionnaire

### 9.2 Acceptance Criteria

**System deemed acceptable if:**

- All P0 (critical) requirements implemented and tested
- No critical bugs in production
- Performance targets met (dashboard <2s, interaction check <3s)
- Security audit passed
- Regulatory compliance verified
- User satisfaction >4/5 stars

---

## 10. Appendices

### Appendix A: Medical Abbreviations

| Abbreviation | Meaning           |
| ------------ | ----------------- |
| TID          | Three times daily |
| BID          | Twice daily       |
| QID          | Four times daily  |
| PRN          | As needed         |
| PO           | By mouth (oral)   |
| IV           | Intravenous       |
| IM           | Intramuscular     |
| STAT         | Immediately       |
| OD           | Right eye         |
| OS           | Left eye          |
| OU           | Both eyes         |

### Appendix B: Error Codes

| Code     | Description               | User Action                   |
| -------- | ------------------------- | ----------------------------- |
| RX-001   | Prescription not found    | Verify Rx number              |
| RX-002   | Cannot dispense (expired) | Contact prescriber for new Rx |
| INV-001  | Insufficient stock        | Partial fill or backorder     |
| INV-002  | Drug expired              | Remove from inventory         |
| AUTH-001 | Invalid credentials       | Check email/password          |
| AUTH-002 | Account locked            | Wait 15 min or contact admin  |
| API-001  | Drug database unavailable | Manual review required        |

### Appendix C: Database Enums

- UserRole: `PHARMACY`, `PHARMACY_ADMIN`
- PrescriptionStatus: `pending`, `validated`, `queried`, `ready_for_dispense`, `dispensed`, `completed`, `rejected`
- PrescriptionPriority: `routine`, `urgent`, `stat`
- PaymentStatus: `pending`, `paid`, `partial`, `canceled`, `refunded`
- InventoryTxnType: `RECEIPT`, `DISPENSING`, `ADJUSTMENT`, `RETURN`

---

## Document Approval

| Role                   | Name             | Signature  | Date     |
| ---------------------- | ---------------- | ---------- | -------- |
| Product Owner          | ****\_\_\_\_**** | ****\_**** | \_\_\_\_ |
| Technical Lead         | ****\_\_\_\_**** | ****\_**** | \_\_\_\_ |
| Pharmacist Stakeholder | ****\_\_\_\_**** | ****\_**** | \_\_\_\_ |
| QA Lead                | ****\_\_\_\_**** | ****\_**** | \_\_\_\_ |
| Compliance Officer     | ****\_\_\_\_**** | ****\_**** | \_\_\_\_ |

---

**Document Status:** ✅ Approved for Development  
**Next Review Date:** February 4, 2026  
**Version Control:** Tracked in Git repository

---

**END OF SOFTWARE REQUIREMENTS SPECIFICATION**
