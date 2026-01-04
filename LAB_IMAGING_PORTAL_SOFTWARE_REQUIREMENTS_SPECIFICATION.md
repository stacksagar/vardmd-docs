# VardMD Lab & Imaging Portal - Software Requirements Specification (SRS)

**Project:** VardMD Lab & Imaging Portal  
**Version:** 1.0  
**Date:** January 4, 2026  
**Status:** Approved for Development

---

## 1. Introduction

### 1.1 Purpose

This Software Requirements Specification (SRS) describes the complete software requirements for the VardMD Lab & Imaging Portal, intended for developers, testers, project managers, and stakeholders.

### 1.2 Scope

The **VardMD Lab & Imaging Portal** is a web-based laboratory management system that enables:

- Lab technicians to process diagnostic test orders with barcode sample tracking
- Multi-stage result validation before release to providers
- Critical value flagging and urgent notifications
- Equipment calibration tracking for regulatory compliance
- Turnaround time analytics and quality metrics

**Benefits:**

- Reduce average TAT from 4-6 hours to 2-3 hours
- Eliminate specimen mix-ups (target: 0%)
- Ensure 100% result validation before release
- Automate regulatory compliance documentation

### 1.3 Definitions

| Term           | Definition                                                                     |
| -------------- | ------------------------------------------------------------------------------ |
| TAT            | Turnaround Time (order creation to result release)                             |
| STAT           | Immediate priority - process within 1 hour                                     |
| Sample         | Physical laboratory specimen (blood, urine, tissue)                            |
| Barcode        | Unique identifier for specimen tracking                                        |
| Validation     | Senior technician/pathologist verifying result accuracy                        |
| Critical Value | Result requiring urgent physician notification (e.g., dangerously low glucose) |
| QC             | Quality Control (equipment calibration, testing standards)                     |
| CLIA           | Clinical Laboratory Improvement Amendments (US regulatory standard)            |
| CAP            | College of American Pathologists (accreditation body)                          |

### 1.4 References

- VardMD Database Schema Documentation
- Lab & Imaging API Guide (603 lines)
- Lab & Imaging Portal Design Brief
- Lab & Imaging Portal Functional Requirements
- Lab & Imaging Portal Product Requirements
- CLIA Regulations
- CAP Laboratory Accreditation Standards

---

## 2. Overall Description

### 2.1 Product Perspective

```
┌─────────────────────────────────────────────┐
│        VardMD Healthcare Platform           │
├─────────────────────────────────────────────┤
│                                             │
│  Provider Portal ──► Lab Portal ──► Patient │
│         │                 │                 │
│         └──► EMR Portal ──┘                 │
│                                             │
│  Super Admin (Monitoring & Reports)         │
└─────────────────────────────────────────────┘
```

**System Interfaces:**

- **Input:** Test orders from Provider Portal and EMR Portal
- **Output:** Test results to Provider Portal, critical alerts to providers
- **External:** File storage (DigitalOcean Spaces for X-ray images, PDFs)

### 2.2 Product Functions

1. **Order Management:** Queue with priority handling (STAT, urgent, routine)
2. **Sample Tracking:** Barcode registration, location tracking, rejection workflow
3. **Result Management:** Upload, multi-stage validation, critical flagging, release
4. **Equipment Tracking:** Calibration schedule, maintenance logging, downtime tracking
5. **Analytics:** TAT reports, test volume, quality metrics

### 2.3 User Characteristics

| User Type                         | Technical Expertise | Usage Pattern                          |
| --------------------------------- | ------------------- | -------------------------------------- |
| **Lab Technician**                | Moderate            | Daily, 8-10 hours, 20-40 tests         |
| **Lab Administrator**             | Moderate to High    | Daily, focus on management and reports |
| **Senior Technician/Pathologist** | Moderate            | Result validation, 2-4 hours/day       |

### 2.4 Constraints

**Regulatory:**

- MUST comply with CLIA regulations (if applicable)
- MUST maintain 7-year record retention
- MUST track equipment calibration
- MUST document critical result notifications

**Technical:**

- MUST use PostgreSQL database
- MUST use Prisma ORM
- MUST support barcode scanners
- MUST support file uploads (images, PDFs)
- MUST be responsive (desktop, tablet, mobile)

**Business:**

- Multi-tenant architecture (data isolation per organization)
- 4-week development timeline for Phase 1
- Use existing VardMD infrastructure

### 2.5 Assumptions and Dependencies

**Assumptions:**

- Internet connectivity available during lab operating hours
- Barcode scanners available for sample tracking
- Lab staff have basic computer literacy

**Dependencies:**

- VardMD authentication system
- Provider Portal operational (source of test orders)
- DigitalOcean Spaces for file storage
- PostgreSQL database availability

---

## 3. Specific Functional Requirements

### 3.1 User Authentication

**FR-AUTH-001: User Login**

- **Description:** Authenticate users via email and password
- **Input:** Email, password
- **Processing:** Validate credentials, generate JWT token
- **Output:** JWT token (8-hour expiry for technician, 12-hour for admin)
- **Priority:** CRITICAL

**FR-AUTH-002: Role-Based Access Control**

- **Description:** Enforce role-based permissions
- **Roles:**
  - `LAB_IMAGING`: Process orders, upload results, track samples
  - `LAB_ADMIN`: All LAB_IMAGING + manage test catalog, equipment, reports
- **Priority:** CRITICAL

### 3.2 Dashboard

**FR-DASH-001: Dashboard Statistics**

- **Description:** Display real-time lab statistics
- **Output:**
  - Pending tests count
  - Tests in progress
  - Awaiting validation
  - Ready for release
  - Critical results count
  - Today's test volume
  - Average TAT
- **Update Frequency:** Every 5 minutes
- **Priority:** HIGH

### 3.3 Test Order Queue

**FR-ORDER-001: Order Queue Display**

- **Description:** Display all test orders in prioritized queue
- **Display Fields:**
  - Order number, patient name/MRN, provider, test name, category
  - Urgency, status, time ordered, expected TAT
- **Sorting:** STAT > Urgent > Routine, then by date
- **Filtering:** Status, urgency, date range, test type
- **Pagination:** 20, 50, 100 items per page
- **Priority:** CRITICAL

**FR-ORDER-002: Order Status Workflow**

- **Description:** Enforce status transitions
- **Valid Transitions:**
  ```
  pending → in-progress → result-ready → released
     ↓
  rejected
  ```
- **Rules:**
  - Cannot skip statuses
  - Cannot move backward
  - Status changes logged
- **Priority:** CRITICAL

**FR-ORDER-003: Accept Order**

- **Description:** Technician accepts incoming order
- **Processing:**
  - Verify order status is "pending"
  - Assign to current user
  - Update status to "in-progress"
  - Record start time
- **Priority:** CRITICAL

**FR-ORDER-004: Reject Order**

- **Description:** Technician rejects order (cannot fulfill)
- **Input:** Rejection reason
- **Processing:**
  - Update status to "rejected"
  - Log reason
  - Notify ordering provider
- **Examples:** Equipment unavailable, test discontinued
- **Priority:** HIGH

### 3.4 Sample Tracking

**FR-SAMPLE-001: Sample Registration**

- **Description:** Register physical sample with barcode
- **Input:**
  - Barcode (scanned or manual)
  - Sample type (blood, urine, tissue)
  - Collection date/time
- **Processing:**
  - Validate barcode uniqueness
  - Link to lab order
  - Set status to "received"
- **Priority:** CRITICAL

**FR-SAMPLE-002: Sample Status Tracking**

- **Description:** Track sample through workflow
- **Status Values:**
  - `collected`: Obtained from patient
  - `received`: Arrived at lab
  - `processing`: Being analyzed
  - `processed`: Analysis complete
  - `rejected`: Unsuitable for testing
- **Priority:** HIGH

**FR-SAMPLE-003: Sample Location Tracking**

- **Description:** Track sample location in lab
- **Locations:**
  - Lab reception
  - Workstation (e.g., "Hematology Station A")
  - Equipment (e.g., "Sysmex Analyzer Bay 2")
  - Storage (e.g., "Refrigerator Shelf B3")
- **Update Method:** Scan barcode at each location
- **Priority:** MEDIUM

**FR-SAMPLE-004: Sample Rejection**

- **Description:** Reject unsuitable samples
- **Input:**
  - Rejection reason (clotted, insufficient, wrong tube, contaminated)
  - Photos (optional)
- **Processing:**
  - Update sample status to "rejected"
  - Log reason
  - Notify ordering provider
  - Create new order for recollection
  - Link to original order
- **Priority:** HIGH

### 3.5 Result Management

**FR-RESULT-001: Upload Results**

- **Description:** Upload test results (structured data and files)
- **Input:**
  - Result values (e.g., `{"hemoglobin": 12.5, "wbc": 7200}`)
  - Notes (free text)
  - File attachments (JPG, PNG, PDF, DICOM)
- **Validation:**
  - Required fields enforced
  - Numeric values within reasonable ranges
  - File size < 50MB per file
- **Processing:**
  - Create LabResult record
  - Store result data as JSON
  - Upload files to DigitalOcean Spaces
  - Set status to "draft"
- **Priority:** CRITICAL

**FR-RESULT-002: Validate Results**

- **Description:** Senior technician/pathologist validates results
- **Authorization:** Only LAB_ADMIN or designated validators
- **Input:**
  - Validation notes
  - Approval/rejection decision
- **Processing:**
  - Update result status to "validated"
  - Record validator and timestamp
  - Auto-detect critical values → flag if present
- **Priority:** CRITICAL

**FR-RESULT-003: Critical Value Detection**

- **Description:** Automatically detect critical values
- **Detection Rules:**
  - Compare against reference ranges
  - Flag values outside critical thresholds
- **Examples:**
  - Blood Glucose <50 or >400 mg/dL
  - Hemoglobin <7 or >20 g/dL
  - Potassium <2.5 or >6.0 mEq/L
- **Actions:**
  - Red alert banner
  - Mandatory validation notes
  - Urgent notification flag
- **Priority:** CRITICAL (SAFETY)

**FR-RESULT-004: Release Results**

- **Description:** Final approval and notification
- **Authorization:** LAB_ADMIN or authorized validator
- **Processing:**
  - Verify result status is "validated"
  - Update status to "released"
  - Make results visible in Provider Portal
  - Send notification to ordering provider
  - If critical: Send urgent alert (SMS, email, app notification)
- **SLA:** Notification within 1 minute of release
- **Priority:** CRITICAL

**FR-RESULT-005: Download Results**

- **Description:** Generate PDF report
- **Output:** Formatted PDF with:
  - Patient demographics
  - Test name and date
  - Result values
  - Reference ranges
  - Performer and validator signatures
  - Attached images (if applicable)
- **Priority:** MEDIUM

### 3.6 Test Catalog Management

**FR-CATALOG-001: Test Catalog Database**

- **Description:** Maintain comprehensive test catalog (LAB_ADMIN only)
- **Data Fields:**
  - Test code, test name, category
  - Cost, turnaround time
  - Sample type, collection instructions
  - Active/inactive status
- **Multi-Tenant:** Each organization has own catalog
- **Priority:** HIGH

**FR-CATALOG-002: Add/Edit Tests**

- **Description:** CRUD operations on test catalog
- **Operations:**
  - Add new test types
  - Edit test details and pricing
  - Deactivate tests (soft delete)
- **Business Rules:**
  - Cannot delete (maintain history)
  - Price changes logged with effective date
  - Test code unique per organization
- **Priority:** MEDIUM

### 3.7 Equipment Management

**FR-EQUIP-001: Equipment Inventory**

- **Description:** Track lab equipment
- **Data Fields:**
  - Name, model, serial number
  - Purchase date, status (active, maintenance, retired)
  - Location
- **Priority:** MEDIUM

**FR-EQUIP-002: Calibration Tracking**

- **Description:** Track calibration schedule (REGULATORY)
- **Data Fields:**
  - Last calibration date
  - Next calibration due date
  - Calibration frequency
- **Alerts:** 30 days before due
- **Business Rules:**
  - Uncalibrated equipment flagged
  - Tests on uncalibrated equipment flagged for review
- **Priority:** HIGH

**FR-EQUIP-003: Maintenance Logging**

- **Description:** Log maintenance events
- **Input:**
  - Maintenance type (calibration, repair, replacement)
  - Performed by, date, notes
  - Parts replaced
  - Next maintenance due
- **Purpose:** Regulatory compliance, budget planning
- **Priority:** MEDIUM

### 3.8 Reporting

**FR-REPORT-001: Daily Test Report**

- **Description:** Generate daily test summary
- **Metrics:**
  - Total tests ordered
  - Tests completed
  - Tests pending
  - Average TAT
  - Breakdown by category and urgency
- **Priority:** MEDIUM

**FR-REPORT-002: Turnaround Time Report**

- **Description:** Calculate TAT metrics
- **Metrics:**
  - Average TAT per test type
  - Target TAT vs. actual
  - % tests completed within target
- **Calculation:** `TAT = Time Released - Time Ordered`
- **Charts:** Historical trends (weekly, monthly)
- **Export:** Excel
- **Priority:** HIGH

**FR-REPORT-003: Critical Result Report**

- **Description:** Report on critical values
- **Metrics:**
  - Number of critical results
  - Average notification time
  - Critical result types distribution
- **Purpose:** Quality assurance
- **Priority:** HIGH

---

## 4. External Interface Requirements

### 4.1 User Interfaces

**UI-001: Responsive Design**

- Desktop ≥1920px, tablet 768-1024px, mobile 375-767px
- Navigation: Desktop (sidebar), Mobile (hamburger)
- Critical actions within 3 clicks

**UI-002: Design System**

- shadcn/ui components
- Tailwind CSS
- Dark mode support
- Color coding:
  - Red: STAT, critical values, errors
  - Orange: Urgent, warnings
  - Green: Success, completed
  - Blue: Info, routine

**UI-003: Accessibility**

- WCAG 2.1 Level AA
- Screen reader compatible
- Keyboard navigation
- Min contrast ratio 4.5:1

### 4.2 Hardware Interfaces

**HW-001: Barcode Scanner**

- USB barcode scanners (keyboard wedge mode)
- Scan sample barcodes during registration
- Scan location barcodes during tracking

**HW-002: Camera** (Future)

- Capture photos of sample labels
- Scan barcodes using phone camera

### 4.3 Software Interfaces

**SI-001: Provider Portal**

- Shared PostgreSQL database
- Data Flow: Provider creates LabOrder → Lab receives
- Real-time: WebSocket notifications

**SI-002: File Storage Service**

- DigitalOcean Spaces
- Upload: X-ray images, PDF reports
- Access: Secure URLs with expiry

**SI-003: Notification Service**

- Email: SendGrid
- SMS: Twilio (future)
- Push notifications: FCM (future)

### 4.4 Communication Interfaces

**COM-001: HTTPS/TLS**

- All communication over HTTPS
- TLS 1.2 or higher

**COM-002: WebSocket**

- Real-time notifications (new orders, critical results)
- Fallback to polling

**COM-003: REST API**

- JSON request/response
- Standard HTTP methods
- RESTful URLs

---

## 5. Non-Functional Requirements

### 5.1 Performance

**PERF-001: Response Time**

- Dashboard load: <2 seconds
- Order queue: <2 seconds
- File upload (10MB): <5 seconds
- API calls: <1 second average

**PERF-002: Throughput**

- Support 30 concurrent users per lab
- Process 500 orders/day per lab
- 50 transactions/second during peak

### 5.2 Safety

**SAFE-001: Patient Safety**

- Zero specimen mix-ups (barcode validation)
- Critical values auto-flagged
- Multi-stage validation enforced

**SAFE-002: Data Backup**

- Automated backups every 6 hours
- 30-day retention
- Point-in-time recovery

**SAFE-003: Disaster Recovery**

- RTO: <4 hours
- RPO: <1 hour

### 5.3 Security

**SEC-001: Authentication**

- JWT-based
- Password hashing: bcrypt (10+ rounds)
- Session timeout: 8 hours (tech), 12 hours (admin)
- Account lockout: 5 failed attempts, 15 min cooldown

**SEC-002: Authorization**

- Role-based access control (RBAC)
- Admin-only routes protected
- File upload: virus scanning

**SEC-003: Data Encryption**

- At rest: AES-256
- In transit: HTTPS/TLS 1.2+
- Encrypted backups

**SEC-004: Audit Logging**

- Log all critical actions
- Immutable logs (append-only)
- Log fields: User, action, entity, timestamp

**SEC-005: Multi-Tenant Isolation**

- All queries filter by tenantId
- Cross-tenant access prohibited

### 5.4 Software Quality

**QUAL-001: Reliability**

- Uptime: 99.5%
- Graceful error handling
- Database replicas

**QUAL-002: Maintainability**

- TypeScript strict mode
- Unit test coverage >70%
- API documentation (OpenAPI/Swagger)

**QUAL-003: Usability**

- Intuitive UI
- Clear error messages
- Keyboard shortcuts
- Help tooltips

**QUAL-004: Scalability**

- Horizontal scaling
- Database read replicas
- Support 50+ organizations
- Caching for static data

---

## 6. Database Requirements

### 6.1 Database Schema

**Tables:**

- LabOrder, LabResult, LabSample (core workflow)
- LabTestCatalog (test menu)
- LabEquipment, LabMaintenanceLog (equipment tracking)
- User, Organization, Tenant (core entities)

**Relationships:**

- LabOrder 1:N LabResult
- LabOrder 1:N LabSample
- LabResult N:1 User (performer, validator)
- LabEquipment 1:N LabMaintenanceLog

**Indexes:**

- tenantId (all tenant-scoped tables)
- status, urgency, createdAt (LabOrder)
- sampleBarcode (LabSample - unique)
- nextCalibrationDue (LabEquipment)

### 6.2 Data Integrity

**Constraints:**

- Primary keys: UUID
- Foreign keys: Cascade for tenants, restrict for orders
- Unique: tenantId + orderNumber, sampleBarcode
- Check: TAT calculated fields, status enum

**Transactions:**

- Result upload + validation: atomic
- Sample registration: atomic
- ACID compliance

### 6.3 Data Retention

**Retention Policies:**

- Lab orders and results: 7 years
- Sample records: 7 years
- Equipment logs: Equipment lifespan + 3 years
- Audit logs: 7 years

---

## 7. System Models

### 7.1 State Diagram (Order Status)

```
    [New Order]
         │
         ▼
    ┌─────────┐
    │ pending │
    └────┬────┘
         │ Accept
         ▼
   ┌────────────┐
   │in-progress │
   └────┬───────┘
         │ Upload Results
         ▼
  ┌─────────────┐
  │result-ready │
  └──────┬──────┘
         │ Validate
         ▼
  ┌──────────┐
  │validated │
  └────┬─────┘
         │ Release
         ▼
  ┌──────────┐
  │ released │
  └──────────┘

Failure Path:
pending ──► rejected
```

---

## 8. Validation and Verification

### 8.1 Validation Methods

**Unit Testing:**

- Backend services: Jest
- Coverage >70%
- Focus: Business logic, TAT calculations, critical value detection

**Integration Testing:**

- API endpoints: Supertest
- Database integration
- File upload/download

**End-to-End Testing:**

- User workflows: Playwright
- Critical paths: Order processing, result upload, validation, release
- Cross-browser testing

**User Acceptance Testing:**

- Pilot labs test in production-like environment
- Real test orders (sanitized data)
- Feedback via questionnaire

### 8.2 Acceptance Criteria

**System deemed acceptable if:**

- All P0 (critical) requirements implemented
- No critical bugs
- Performance targets met (dashboard <2s, upload <5s)
- Security audit passed
- 100% result validation enforced
- Average TAT < 3 hours

---

## 9. Appendices

### Appendix A: Critical Value Reference Ranges

| Parameter     | Critical Low | Critical High |
| ------------- | ------------ | ------------- |
| Blood Glucose | <50 mg/dL    | >400 mg/dL    |
| Hemoglobin    | <7 g/dL      | >20 g/dL      |
| Potassium     | <2.5 mEq/L   | >6.0 mEq/L    |
| Sodium        | <120 mEq/L   | >160 mEq/L    |
| WBC Count     | <2,000/µL    | >30,000/µL    |

### Appendix B: Sample Rejection Reasons

- Clotted sample
- Insufficient quantity
- Wrong tube type
- Hemolyzed (for certain tests)
- Contaminated
- Unlabeled or mislabeled
- Expired sample (>24 hours old for certain tests)

### Appendix C: Error Codes

| Code    | Description           | Action                   |
| ------- | --------------------- | ------------------------ |
| LAB-001 | Order not found       | Verify order number      |
| LAB-002 | Duplicate barcode     | Use different barcode    |
| LAB-003 | File upload failed    | Retry or contact support |
| LAB-004 | Invalid result format | Check numeric values     |

---

## Document Approval

| Role               | Name             | Signature  | Date     |
| ------------------ | ---------------- | ---------- | -------- |
| Product Owner      | ****\_\_\_\_**** | ****\_**** | \_\_\_\_ |
| Lab Director       | ****\_\_\_\_**** | ****\_**** | \_\_\_\_ |
| Technical Lead     | ****\_\_\_\_**** | ****\_**** | \_\_\_\_ |
| QA Lead            | ****\_\_\_\_**** | ****\_**** | \_\_\_\_ |
| Compliance Officer | ****\_\_\_\_**** | ****\_**** | \_\_\_\_ |

---

**Document Status:** ✅ Approved for Development  
**Next Review Date:** February 4, 2026

---

**END OF SOFTWARE REQUIREMENTS SPECIFICATION**
