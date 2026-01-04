# VardMD Lab & Imaging Portal - Functional Requirements Document

**Project:** VardMD Healthcare Platform  
**Module:** Lab & Imaging Portal  
**Version:** 1.0  
**Date:** January 4, 2026  
**Status:** Draft for Review

---

## 1. Introduction

### 1.1 Purpose

This document specifies the functional requirements for the VardMD Lab & Imaging Portal, enabling laboratory technicians and administrators to process diagnostic test orders, manage specimens, validate and release results, and ensure regulatory compliance.

### 1.2 Scope

**In Scope:**

- Test order queue management
- Sample tracking with barcode scanning
- Result upload and multi-stage validation
- Critical result flagging and notification
- Test catalog management
- Equipment calibration tracking
- Turnaround time reporting

**Out of Scope:**

- Laboratory Information System (LIS) hardware integration
- Automated analyzer interfacing (future phase)
- Insurance claim processing
- Patient result portal (separate module)

### 1.3 Definitions

| Term           | Definition                                        |
| -------------- | ------------------------------------------------- |
| TAT            | Turnaround Time (order to result delivery)        |
| STAT           | Urgent priority - immediate processing            |
| Sample         | Physical specimen (blood, urine, tissue)          |
| Validation     | Senior tech/pathologist verifying result accuracy |
| Release        | Final approval making results visible to provider |
| Critical Value | Result requiring urgent physician notification    |
| QC             | Quality Control (equipment calibration, testing)  |

---

## 2. Functional Requirements

### FR-1: Dashboard Display

**FR-1.1: Dashboard Statistics**

- **Priority:** HIGH
- **User Role:** LAB_IMAGING, LAB_ADMIN
- **Requirement:** System SHALL display real-time lab statistics:
  - Pending tests count
  - Tests in progress
  - Awaiting validation
  - Ready for release
  - Critical results count
  - Today's test volume
  - Average turnaround time

**Acceptance Criteria:**

- Statistics update every 5 minutes
- Tenant-isolated data
- Dashboard loads within 2 seconds
- Color-coded status indicators (red=STAT, orange=urgent, blue=routine)

---

### FR-2: Test Order Queue Management

**FR-2.1: Order Queue Display**

- **Priority:** CRITICAL
- **User Role:** LAB_IMAGING, LAB_ADMIN
- **Requirement:** System SHALL display all pending lab orders with:
  - Order number (LAB-YYYY-NNNNNN format)
  - Patient name and MRN
  - Ordering provider
  - Test name and category
  - Urgency (routine, urgent, STAT)
  - Current status
  - Time ordered
  - Expected turnaround time

**Business Rules:**

- STAT orders MUST appear at top with red indicator
- Urgent orders before routine
- Orders >24 hours old highlighted
- Auto-refresh every 60 seconds

**Acceptance Criteria:**

- User can sort by any column
- Filter by status, urgency, date, test type
- Search by patient name, MRN, order number
- Pagination (20, 50, 100 per page)

**FR-2.2: Order Status Workflow**

- **Priority:** CRITICAL
- **Requirement:** System SHALL enforce status transitions:
  ```
  pending → in-progress → result-ready → released
     ↓
  rejected (cannot fulfill)
  ```

**Status Definitions:**

- **pending**: New order awaiting technician acceptance
- **in-progress**: Technician accepted, processing underway
- **result-ready**: Results uploaded and validated
- **released**: Results delivered to provider
- **rejected**: Cannot fulfill (equipment unavailable, etc.)

**Business Rules:**

- Cannot skip statuses
- Cannot move backward
- Status changes logged with timestamp and user ID
- "Released" status triggers provider notification

**FR-2.3: Order Detail View**

- **Priority:** HIGH
- **Requirement:** System SHALL display complete order details:
  - Patient demographics, allergies, current medications
  - Ordering provider and date/time
  - Test details (name, category, sample required, TAT)
  - Clinical notes/indication
  - Previous results for same test (if available)

---

### FR-3: Sample Tracking

**FR-3.1: Sample Registration**

- **Priority:** HIGH
- **User Role:** LAB_IMAGING
- **Requirement:** System SHALL allow registering physical samples with:
  - Unique barcode identifier
  - Sample type (blood, urine, tissue)
  - Collection date/time
  - Collection location
  - Linked to lab order

**Acceptance Criteria:**

- Barcode scanner support
- Manual entry option
- Validate barcode uniqueness
- Photo capture of sample label (optional)

**FR-3.2: Sample Status Tracking**

- **Priority:** HIGH
- **Requirement:** System SHALL track sample status:
  - **collected**: Sample obtained from patient
  - **received**: Sample arrived at lab
  - **processing**: Sample being analyzed
  - **processed**: Analysis complete
  - **rejected**: Unsuitable for testing

**FR-3.3: Sample Location Tracking**

- **Priority:** MEDIUM
- **Requirement:** System SHALL track sample location:
  - Lab reception
  - Specific workstation (e.g., "Hematology Station A")
  - Equipment (e.g., "Sysmex Analyzer Bay 2")
  - Storage (e.g., "Refrigerator Shelf B3")

**Purpose:** Prevent sample loss in busy labs processing 200+ specimens/day

**FR-3.4: Sample Rejection**

- **Priority:** HIGH
- **Requirement:** System SHALL allow rejecting unsuitable samples with:
  - Rejection reason (clotted, insufficient quantity, wrong tube, contaminated)
  - Automated notification to ordering provider
  - Create new order for recollection
  - Link to original order for audit

**Business Rules:**

- Rejection MUST include documented reason
- Provider notified within 15 minutes
- Original order marked as "sample rejected"
- New order created automatically with recollection instructions

---

### FR-4: Result Upload and Management

**FR-4.1: Result Upload (Perform Stage)**

- **Priority:** CRITICAL
- **User Role:** LAB_IMAGING
- **Requirement:** System SHALL allow uploading test results including:
  - Structured data (numeric values, e.g., Hemoglobin: 12.5 g/dL)
  - Free-text notes
  - File attachments (X-ray images, PDF reports, graphs)

**Data Entry Methods:**

- Manual entry (type values into form)
- Upload structured file (CSV from analyzer)
- Drag-and-drop image files
- Scan QR code from paper result

**Validation:**

- Required fields enforced
- Numeric values validated (reasonable ranges)
- File size limits (max 50MB per file)
- Supported formats: JPG, PNG, PDF, DICOM (medical imaging)

**Acceptance Criteria:**

- Form pre-populated with test parameters
- Auto-save draft every 30 seconds
- Multiple file upload supported
- Preview uploaded files before submission

**FR-4.2: Result Validation (Second Stage)**

- **Priority:** CRITICAL
- **User Role:** LAB_ADMIN or Senior Technician
- **Requirement:** System SHALL provide validation workflow:
  - Review uploaded results
  - Verify values are accurate and consistent
  - Add validation notes
  - Flag critical values
  - Approve or send back for correction

**Business Rules:**

- Only senior techs or pathologists can validate
- Validation MUST occur before results released
- Critical values auto-detected and highlighted
- Validator signs off electronically

**Acceptance Criteria:**

- Side-by-side view of patient history
- Critical value alerts in red
- Require validation notes for critical results
- Digital signature capture

**FR-4.3: Critical Value Flagging**

- **Priority:** CRITICAL (SAFETY)
- **Requirement:** System SHALL automatically flag critical values:
  - Compare result against normal reference ranges
  - Identify values requiring urgent physician notification
  - Display severity (critical, high, low)

**Examples of Critical Values:**

- Blood Glucose <50 or >400 mg/dL
- Hemoglobin <7 or >20 g/dL
- Potassium <2.5 or >6.0 mEq/L

**Actions When Critical:**

- Red alert banner
- Mandatory validation notes
- Urgent notification to provider
- SMS/phone call alert (future)
- Documented in audit log

**FR-4.4: Result Release (Final Stage)**

- **Priority:** CRITICAL
- **User Role:** LAB_ADMIN or Authorized Validator
- **Requirement:** System SHALL allow releasing results to providers:
  - Final approval step
  - Make results visible in Provider Portal
  - Send notification to ordering provider
  - If critical: Send urgent alert

**Business Rules:**

- Results cannot be released without validation
- Release action logged with timestamp and user
- Provider receives notification within 1 minute
- Critical results flagged prominently

---

### FR-5: Test Catalog Management

**FR-5.1: Test Catalog Database**

- **Priority:** HIGH
- **User Role:** LAB_ADMIN
- **Requirement:** System SHALL maintain test catalog with:
  - Test code (unique identifier)
  - Test name (e.g., "Complete Blood Count")
  - Category (Hematology, Radiology, Biochemistry, Microbiology)
  - Cost (patient charge)
  - Expected turnaround time
  - Sample type required (blood, urine, etc.)
  - Sample collection instructions
  - Active/inactive status

**Purpose:** Define what tests lab can perform

**FR-5.2: Test Catalog CRUD**

- **Priority:** MEDIUM
- **User Role:** LAB_ADMIN
- **Requirement:** System SHALL allow:
  - Add new test types (when equipment acquired)
  - Edit test details and pricing
  - Deactivate tests (equipment broken, test discontinued)
  - View test history

**Business Rules:**

- Cannot delete tests (maintain historical data)
- Price changes logged with effective date
- Inactive tests not available for new orders
- Test code unique per organization

---

### FR-6: Equipment Management

**FR-6.1: Equipment Inventory**

- **Priority:** MEDIUM
- **User Role:** LAB_ADMIN
- **Requirement:** System SHALL track lab equipment:
  - Equipment name and model
  - Serial number
  - Purchase date and cost
  - Current status (active, maintenance, retired)
  - Location

**FR-6.2: Calibration Tracking**

- **Priority:** HIGH (REGULATORY)
- **Requirement:** System SHALL track calibration schedule:
  - Last calibration date
  - Next calibration due date
  - Calibration frequency (monthly, quarterly, annual)
  - Automated alerts 30 days before due

**Business Rules:**

- Uncalibrated equipment MUST be flagged
- Tests performed on uncalibrated equipment flagged for review
- Calibration events logged in maintenance history

**FR-6.3: Maintenance Logging**

- **Priority:** MEDIUM
- **Requirement:** System SHALL log maintenance events:
  - Maintenance type (calibration, repair, replacement)
  - Performed by (name, company)
  - Date and duration
  - Notes and parts replaced
  - Next maintenance due

**Purpose:** Regulatory compliance, budget planning, downtime tracking

---

### FR-7: Reporting and Analytics

**FR-7.1: Daily Test Report**

- **Priority:** MEDIUM
- **User Role:** LAB_ADMIN
- **Requirement:** System SHALL generate daily test report:
  - Total tests ordered
  - Tests completed
  - Tests pending
  - Average turnaround time
  - Breakdown by category
  - Breakdown by urgency

**FR-7.2: Turnaround Time Report**

- **Priority:** HIGH
- **Requirement:** System SHALL calculate TAT metrics:
  - Average TAT per test type
  - Target TAT vs. actual
  - % tests completed within target
  - Identify bottlenecks

**TAT Calculation:**

```
TAT = Time Released - Time Ordered
```

**Acceptance Criteria:**

- Historical trends (weekly, monthly)
- Export to Excel
- Charts and graphs

**FR-7.3: Critical Result Report**

- **Priority:** HIGH
- **Requirement:** System SHALL report on critical values:
  - Number of critical results
  - Average notification time
  - Provider response time (future)
  - Critical result types distribution

**Purpose:** Quality assurance, regulatory compliance

---

## 3. Business Rules

### BR-1: Order Processing Rules

**BR-1.1:** STAT orders MUST be processed within 1 hour of receipt.

**BR-1.2:** Urgent orders MUST be processed within 4 hours.

**BR-1.3:** Routine orders SHOULD be processed within specified TAT (typically 24-48 hours).

**BR-1.4:** Orders >72 hours old without results MUST trigger investigation alert.

**BR-1.5:** Results MUST be validated before release (cannot skip validation stage).

**BR-1.6:** Critical results MUST be communicated to provider within 30 minutes of validation.

---

### BR-2: Sample Handling Rules

**BR-2.1:** Samples MUST be uniquely identified with barcode or handwritten label.

**BR-2.2:** Rejected samples MUST have documented reason.

**BR-2.3:** Sample storage temperature and conditions MUST be logged.

**BR-2.4:** Samples older than retention period MUST be disposed per biohazard protocol.

---

### BR-3: Quality Control Rules

**BR-3.1:** Equipment MUST be calibrated per manufacturer schedule.

**BR-3.2:** Tests performed on uncalibrated equipment MUST be flagged for review.

**BR-3.3:** QC checks MUST be performed daily before patient testing.

**BR-3.4:** Failed QC MUST halt testing until issue resolved.

---

### BR-4: Data Retention Rules

**BR-4.1:** Test results MUST be retained for 7 years minimum.

**BR-4.2:** Equipment maintenance logs MUST be retained for equipment lifespan + 3 years.

**BR-4.3:** Critical result notifications MUST be logged permanently.

---

## 4. Use Cases

### UC-1: Process Routine Lab Test

**Primary Actor:** Lab Technician  
**Goal:** Complete CBC test and release results

**Main Success Scenario:**

1. Technician logs into dashboard
2. Sees new order "CBC - John Doe - Routine"
3. Clicks "Accept" → Status: in-progress
4. Verifies sample received and acceptable
5. Runs blood through hematology analyzer
6. Machine outputs: Hb 12.5, WBC 7200, Platelets 250k
7. Technician enters values into system
8. Clicks "Submit Results" → Status: result-ready
9. Senior tech Biodun reviews results
10. All values within normal range
11. Clicks "Validate" → Status: validated
12. Pathologist Dr. Kemi final review
13. Clicks "Release" → System notifies Dr. Okafor
14. Results appear in Provider Portal
15. Dr. Okafor reviews and discusses with patient

**Alternative Flow 9a: Critical Value Detected**

1. Senior tech notices Hb 6.5 (critical low)
2. System auto-flags as critical
3. Senior tech adds note: "Severe anemia - transfusion may be needed"
4. Clicks "Validate Critical Result"
5. System sends URGENT notification to Dr. Okafor
6. Dr. Okafor receives SMS alert immediately
7. Dr. Okafor calls patient for immediate follow-up

---

### UC-2: Reject Unsuitable Sample

**Primary Actor:** Lab Technician  
**Goal:** Reject clotted blood sample and request recollection

**Main Scenario:**

1. Technician receives blood sample for CBC
2. Scans barcode → Links to Order LAB-2025-001
3. Visually inspects sample → Sees clots
4. Clicks "Reject Sample"
5. Selects reason: "Sample clotted - cannot process"
6. System asks: "Request recollection?"
7. Technician clicks "Yes"
8. System:
   - Marks original order as "sample rejected"
   - Creates new order LAB-2025-001-R2 (recollection)
   - Notifies Dr. Okafor
   - Notifies phlebotomy team
9. Nurse receives alert: "Recollect blood for John Doe (CBC)"
10. Nurse recollects blood (fresh sample)
11. Process continues with new sample

---

## 5. Non-Functional Requirements

### NFR-1: Performance

- Dashboard load time: <2 seconds
- Result upload: <5 seconds for 10MB file
- Support 30 concurrent users per lab
- Process 500 orders/day per lab

### NFR-2: Availability

- 99.5% uptime
- Automated backups every 6 hours
- RTO: <4 hours, RPO: <1 hour

### NFR-3: Security

- JWT authentication
- Role-based access control
- File encryption at rest (AES-256)
- Audit logging for all critical actions

### NFR-4: Usability

- Mobile responsive
- WCAG 2.1 Level AA accessibility
- Barcode scanner support
- Keyboard shortcuts

### NFR-5: Compliance

- 7-year data retention
- CLIA compliance (Clinical Laboratory Improvement Amendments)
- CAP compliance (College of American Pathologists)
- Nigerian medical device regulations

---

## 6. Acceptance Criteria

### Test Order Management

- [ ] Queue displays all pending orders
- [ ] STAT orders appear at top with red indicator
- [ ] User can accept, reject, or start processing
- [ ] Status transitions enforced
- [ ] Auto-refresh every 60 seconds

### Sample Tracking

- [ ] Barcode scanning supported
- [ ] Sample status tracked
- [ ] Location tracking functional
- [ ] Rejection with reason documented
- [ ] Recollection workflow automated

### Result Management

- [ ] Upload structured and file-based results
- [ ] Validation workflow with approval
- [ ] Critical values auto-flagged
- [ ] Release notification sent within 1 minute
- [ ] Audit trail complete

### Equipment Management

- [ ] Equipment inventory maintained
- [ ] Calibration schedule tracked
- [ ] Alerts 30 days before calibration due
- [ ] Maintenance events logged

### Reporting

- [ ] Daily test report generated
- [ ] TAT metrics calculated correctly
- [ ] Export to Excel supported

---

**END OF FUNCTIONAL REQUIREMENTS DOCUMENT**
