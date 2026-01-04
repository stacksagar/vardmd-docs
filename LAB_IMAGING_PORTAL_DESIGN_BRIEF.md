# VardMD Lab & Imaging Portal - Design Brief

**Project:** VardMD Healthcare Platform  
**Module:** Lab & Imaging Portal (Technician & Admin)  
**Version:** 1.0  
**Date:** January 4, 2026  
**Target Roles:** `LAB_IMAGING`, `LAB_ADMIN`

---

## 📋 Table of Contents

1. [Executive Summary](#executive-summary)
2. [System Architecture](#system-architecture)
3. [Database Schema](#database-schema)
4. [API Specifications](#api-specifications)
5. [Frontend Components](#frontend-components)
6. [User Workflows](#user-workflows)
7. [Security & Authorization](#security--authorization)
8. [Implementation Roadmap](#implementation-roadmap)

---

## 1. Executive Summary

### Purpose

The Lab & Imaging Portal is the operational hub for diagnostic services where lab technicians and radiologists:

- Receive test orders from doctors (blood tests, X-rays, MRIs, etc.)
- Process physical samples with barcode tracking
- Upload and validate test results
- Release reports to healthcare providers
- Manage equipment calibration and maintenance
- Track turnaround times and quality metrics

### Business Impact

**Without this system:**

- Paper orders get lost → delayed results → patient harm
- No sample tracking → specimens mixed up → wrong diagnoses
- Manual result reporting → transcription errors → medication mistakes
- Missing equipment maintenance → machine failures → service disruptions
- No audit trail → medical licensing violations → fines

**With this system:**

- 100% digital tracking → zero lost orders
- Barcode sample tracking → zero mix-ups
- Automated result delivery → instant, accurate reporting
- Equipment maintenance alerts → maximize uptime
- Complete audit logs → regulatory compliance

### User Roles

| Role          | Permissions                                         | Use Cases                                   |
| ------------- | --------------------------------------------------- | ------------------------------------------- |
| `LAB_IMAGING` | Process orders, upload results, track samples       | Lab technicians, radiologists, pathologists |
| `LAB_ADMIN`   | Full access + manage test catalog, equipment, staff | Lab managers, department heads              |

---

## 2. System Architecture

### Tech Stack

**Backend:**

- Runtime: Node.js + TypeScript
- Framework: Express.js
- Database: PostgreSQL
- ORM: Prisma
- Authentication: JWT tokens
- File Storage: DigitalOcean Spaces (X-ray images, PDF reports)

**Frontend:**

- Framework: React + TypeScript
- UI Library: shadcn/ui (Tailwind CSS)
- State Management: React hooks
- File Upload: React Dropzone

### Multi-Tenant Architecture

Every API request and database query MUST include tenant filtering:

```typescript
// Backend - All queries must filter by tenantId
prisma.labOrder.findMany({
  where: {
    tenantId: req.user.tenantId, // REQUIRED
  },
});
```

### Data Flow

```
Doctor orders test → Provider Portal
                        ↓
                [Lab Order created]
                        ↓
                Lab Portal ← You implement this
                        ↓
        [Technician processes & uploads results]
                        ↓
        [Results validated & released]
                        ↓
        Provider Portal ← Results appear
```

---

## 3. Database Schema

### Core Tables (from Prisma Schema)

#### 1. LabOrder Table

```prisma
model LabOrder {
  id                String             @id @default(uuid())
  encounterId       String             @map("encounter_id")
  orderingProviderId String            @map("ordering_provider_id")
  testName          String             @map("test_name")
  urgency           String?            // "routine", "urgent", "stat"
  status            String             @default("pending") // "pending", "in-progress", "completed", "rejected"
  createdAt         DateTime           @default(now()) @map("created_at")
  completedAt       DateTime?          @map("completed_at")
  labResults        LabResult[]
  encounter         Encounter          @relation(fields: [encounterId], references: [id])
  orderingProvider  User               @relation(fields: [orderingProviderId], references: [id])
}
```

**Key Fields:**

- `encounterId`: Links to patient visit
- `orderingProviderId`: Doctor who ordered test
- `testName`: E.g., "Complete Blood Count", "Chest X-Ray"
- `urgency`: Priority level (STAT = immediate)
- `status`: Workflow state

#### 2. LabResult Table

```prisma
model LabResult {
  id              String     @id @default(uuid())
  labOrderId      String     @map("lab_order_id")
  performedBy     String?    @map("performed_by")
  resultData      Json?      @map("result_data") // Structured test values
  notes           String?
  attachmentUrls  String[]   @map("attachment_urls") // X-ray images, PDF reports
  status          String     @default("draft") // "draft", "validated", "released"
  validatedBy     String?    @map("validated_by")
  validatedAt     DateTime?  @map("validated_at")
  releasedAt      DateTime?  @map("released_at")
  createdAt       DateTime   @default(now()) @map("created_at")
  labOrder        LabOrder   @relation(fields: [labOrderId], references: [id])
  performer       User?      @relation("PerformedBy", fields: [performedBy], references: [id])
  validator       User?      @relation("ValidatedBy", fields: [validatedBy], references: [id])
}
```

**Key Features:**

- `resultData`: JSON field storing test values (e.g., `{"hemoglobin": 12.5, "wbc": 7200}`)
- `attachmentUrls`: URLs to uploaded files (X-rays, scanned reports)
- `status`: Multi-stage validation (draft → validated → released)
- `validatedBy`: Senior tech/pathologist who verified results

#### 3. LabTestCatalog Table (Test Menu)

```prisma
model LabTestCatalog {
  id                  String       @id @default(uuid())
  organizationId      String       @map("organization_id")
  testCode            String       @map("test_code")
  testName            String       @map("test_name")
  category            String?      // "Hematology", "Radiology", "Biochemistry"
  cost                Decimal?     @db.Decimal(10, 2)
  turnaroundTime      String?      @map("turnaround_time") // "4 hours", "24 hours"
  sampleType          String?      @map("sample_type") // "Blood", "Urine", "Tissue"
  isActive            Boolean      @default(true) @map("is_active")
  createdAt           DateTime     @default(now()) @map("created_at")
  organization        Organization @relation(fields: [organizationId], references: [id])

  @@unique([organizationId, testCode])
}
```

**Purpose:**

- Defines what tests the lab can perform
- Pricing for billing
- Expected turnaround time for scheduling
- Sample requirements for collection

#### 4. LabEquipment Table

```prisma
model LabEquipment {
  id                    String              @id @default(uuid())
  organizationId        String              @map("organization_id")
  name                  String
  model                 String?
  serialNumber          String?             @map("serial_number")
  lastCalibrationDate   DateTime?           @map("last_calibration_date")
  nextCalibrationDue    DateTime?           @map("next_calibration_due")
  status                String              @default("active") // "active", "maintenance", "retired"
  maintenanceLogs       LabMaintenanceLog[]
  organization          Organization        @relation(fields: [organizationId], references: [id])
}
```

**Purpose:**

- Track expensive lab equipment ($50k-$1M+)
- Calibration schedule (regulatory requirement)
- Maintenance history
- Downtime tracking

#### 5. LabSample Table (Sample Tracking)

```prisma
model LabSample {
  id              String     @id @default(uuid())
  labOrderId      String     @map("lab_order_id")
  sampleBarcode   String     @unique @map("sample_barcode")
  sampleType      String     @map("sample_type") // "Blood", "Urine"
  collectedAt     DateTime?  @map("collected_at")
  receivedAt      DateTime?  @map("received_at")
  status          String     @default("collected") // "collected", "received", "processing", "processed", "rejected"
  rejectionReason String?    @map("rejection_reason")
  location        String?    // "Lab Station A", "Hematology Analyzer"
  labOrder        LabOrder   @relation(fields: [labOrderId], references: [id])
}
```

**Purpose:**

- Track physical specimens through lab
- Barcode scanning for identification
- Prevent sample mix-ups
- Document rejections (clotted blood, insufficient quantity)

### Relationships Diagram

```
User (Doctor)
  └── LabOrder
        ├── LabResult
        │     ├── User (Performer)
        │     └── User (Validator)
        └── LabSample

Organization
  ├── LabTestCatalog (Test Menu)
  └── LabEquipment
        └── LabMaintenanceLog
```

---

## 4. API Specifications

### Base URL

```
/api/v1/lab
```

### API Groups (35 Total)

#### 4.1 Dashboard Statistics (4 APIs)

**GET /dashboard**

Returns lab dashboard summary.

**Response:**

```json
{
  "pendingTests": 18,
  "testsInProgress": 6,
  "awaitingValidation": 9,
  "readyForRelease": 4,
  "criticalResults": 2,
  "todaysVolume": 45,
  "avgTurnaroundTime": "3.2 hours"
}
```

**GET /dashboard/orders-today**

**Response:**

```json
{
  "count": 45,
  "urgent": 12,
  "stat": 3,
  "routine": 30
}
```

**GET /dashboard/pending-count**

**Response:**

```json
{
  "pending": 18,
  "inProgress": 6,
  "awaitingValidation": 9
}
```

**GET /dashboard/critical-alerts**

**Response:**

```json
{
  "alerts": [
    {
      "orderId": "LAB-2025-001",
      "patientName": "John Doe",
      "testName": "Blood Glucose",
      "criticalValue": "40 mg/dL",
      "normalRange": "70-100 mg/dL",
      "severity": "critical"
    }
  ]
}
```

#### 4.2 Orders Queue Management (9 APIs)

**GET /orders**

Get all lab orders with filters.

**Query Parameters:**

- `status`: pending, in-progress, completed
- `urgency`: routine, urgent, stat
- `page`, `limit`

**Response:**

```json
{
  "data": [
    {
      "id": "ord_001",
      "orderId": "LAB-2025-001",
      "patient": {
        "id": "pat_001",
        "name": "Chioma Okeke",
        "mrn": "MRN-10001",
        "age": 34
      },
      "provider": {
        "id": "prov_001",
        "name": "Dr. Amaka Okafor"
      },
      "testName": "Complete Blood Count",
      "category": "Hematology",
      "urgency": "routine",
      "status": "pending",
      "orderedAt": "2026-01-04T09:00:00Z",
      "expectedTAT": "4 hours",
      "sampleRequired": "2mL blood in EDTA tube"
    }
  ],
  "pagination": {
    "total": 50,
    "page": 1,
    "limit": 20
  }
}
```

**PATCH /orders/{id}/accept**

Accept incoming order.

**Response:**

```json
{
  "id": "ord_001",
  "status": "in-progress",
  "acceptedBy": "user_123",
  "acceptedAt": "2026-01-04T09:15:00Z"
}
```

**PATCH /orders/{id}/reject**

Reject order (cannot fulfill).

**Request Body:**

```json
{
  "reason": "Equipment unavailable - MRI scanner under maintenance"
}
```

**PATCH /orders/{id}/in-progress**

Mark as processing.

**Backend Logic:**

1. Verify order status is "pending" or "accepted"
2. Update status to "in-progress"
3. Record start time
4. Assign to current user

**PATCH /orders/{id}/complete**

Mark order as complete.

**Backend Logic:**

1. Verify results have been uploaded and validated
2. Update order status to "completed"
3. Record completion time
4. Calculate actual turnaround time

#### 4.3 Results Management (6 APIs)

**POST /orders/{id}/results**

Upload test results.

**Request Body:**

```json
{
  "resultData": {
    "hemoglobin": 12.5,
    "wbc": 7200,
    "platelets": 250000
  },
  "notes": "All values within normal range",
  "attachments": ["file_id_1", "file_id_2"]
}
```

**Response:**

```json
{
  "id": "result_001",
  "status": "draft",
  "performedBy": "user_123",
  "createdAt": "2026-01-04T10:30:00Z"
}
```

**Backend Logic:**

1. Create LabResult record
2. Store result data as JSON
3. Link attachment URLs from file upload service
4. Set status to "draft"
5. Do NOT notify provider yet (needs validation first)

**PATCH /orders/{id}/results**

Update results (validation stage).

**Request Body:**

```json
{
  "status": "validated",
  "validationNotes": "Results verified, values consistent with patient history"
}
```

**Backend Logic:**

1. Verify user has validator role (senior tech/pathologist)
2. Update result status to "validated"
3. Record validator and timestamp
4. Check for critical values → auto-flag if present

**POST /orders/{id}/flag**

Flag critical result.

**Request Body:**

```json
{
  "criticalValue": "Blood Glucose: 40 mg/dL",
  "normalRange": "70-100 mg/dL",
  "severity": "critical",
  "action": "Immediate physician notification required"
}
```

**Backend Logic:**

1. Create critical result alert
2. Send urgent notification to ordering provider
3. Log in audit trail
4. Display in provider's critical alerts

**POST /orders/{id}/results/notify**

Release results to provider.

**Backend Logic:**

1. Verify result status is "validated"
2. Update result status to "released"
3. Send notification to ordering provider
4. Make results visible in Provider Portal
5. If flagged as critical, send SMS/urgent alert

**GET /orders/{id}/results/download**

Download result PDF.

**Response:** PDF file with formatted test results

#### 4.4 Sample Tracking (5 APIs)

**GET /samples**

Get all samples.

**Response:**

```json
{
  "data": [
    {
      "id": "sample_001",
      "barcode": "SAMPLE-12345",
      "orderId": "LAB-2025-001",
      "patientName": "John Doe",
      "sampleType": "Blood (EDTA)",
      "collectedAt": "2026-01-04T09:30:00Z",
      "receivedAt": "2026-01-04T09:45:00Z",
      "status": "processing",
      "location": "Hematology Analyzer",
      "condition": "acceptable"
    }
  ]
}
```

**PATCH /samples/{id}/status**

Update sample status.

**Request Body:**

```json
{
  "status": "processed",
  "notes": "Analysis complete"
}
```

**PATCH /samples/{id}/location**

Track sample location.

**Request Body:**

```json
{
  "location": "Hematology Analyzer Bay 2"
}
```

**POST /samples/{id}/reject**

Reject unsuitable sample.

**Request Body:**

```json
{
  "reason": "Sample clotted - cannot process",
  "action": "Recollection required"
}
```

**Backend Logic:**

1. Update sample status to "rejected"
2. Record rejection reason
3. Notify ordering provider
4. Create new order for recollection
5. Link original order for audit trail

#### 4.5 Test Catalog Management (6 APIs - ADMIN only)

**GET /test-catalog**

Get available tests.

**Response:**

```json
{
  "data": [
    {
      "id": "test_001",
      "testCode": "CBC-001",
      "testName": "Complete Blood Count",
      "category": "Hematology",
      "cost": 3500,
      "turnaroundTime": "4 hours",
      "sampleType": "Blood (EDTA)",
      "isActive": true
    }
  ]
}
```

**POST /test-catalog**

Add new test type.

**Request Body:**

```json
{
  "testCode": "XRAY-CHEST",
  "testName": "Chest X-Ray",
  "category": "Radiology",
  "cost": 8000,
  "turnaroundTime": "2 hours",
  "sampleType": "N/A - Imaging"
}
```

**PATCH /test-catalog/{id}/pricing**

Update test pricing.

**Request Body:**

```json
{
  "cost": 4000
}
```

#### 4.6 Equipment Management (5 APIs - ADMIN)

**GET /equipment**

List lab equipment.

**Response:**

```json
{
  "data": [
    {
      "id": "eq_001",
      "name": "Hematology Analyzer",
      "model": "Sysmex XN-1000",
      "serialNumber": "SN-123456",
      "status": "active",
      "lastCalibration": "2026-01-01",
      "nextCalibrationDue": "2026-04-01",
      "daysUntilDue": 87
    }
  ]
}
```

**POST /equipment/{id}/maintenance**

Log maintenance event.

**Request Body:**

```json
{
  "maintenanceType": "calibration",
  "performedBy": "BioMed Tech John",
  "notes": "Annual calibration completed, all parameters within spec",
  "nextDueDate": "2027-01-04"
}
```

#### 4.7 Reports (5 APIs)

**GET /reports/daily**

Daily test report.

**Response:**

```json
{
  "date": "2026-01-04",
  "totalTests": 45,
  "completed": 38,
  "pending": 7,
  "avgTurnaroundTime": "3.2 hours",
  "byCategory": {
    "Hematology": 20,
    "Radiology": 15,
    "Biochemistry": 10
  }
}
```

**GET /reports/turnaround-time**

TAT metrics.

**Response:**

```json
{
  "avgTAT": "3.2 hours",
  "targetTAT": "4 hours",
  "performance": "92% within target",
  "byTest": [
    {
      "testName": "CBC",
      "avgTAT": "2.5 hours",
      "target": "4 hours"
    }
  ]
}
```

---

## 5. Frontend Components

### Component Structure

```
src/components/lab-imaging/
├── LabImagingDashboard.tsx        (673 lines - Technician view)
├── LabAdminDashboard.tsx          (945 lines - Admin view)
├── PerformUpload.tsx              (Results upload)
├── ResultValidation.tsx           (Validation workflow)
├── Release.tsx                    (Release results)
├── LabImagingSettings.tsx         (Settings)
└── modals/
    └── LabAdminModals.tsx         (Admin modals)
```

### Key Frontend Interfaces

```typescript
// Lab Order interface
export interface LabOrder {
  id: string;
  orderId: string; // LAB-2025-001
  patientName: string;
  patientMrn: string;
  providerName: string;
  testName: string;
  category: string; // "Hematology", "Radiology"
  status: "pending" | "in-progress" | "result-ready" | "released";
  urgency: "routine" | "urgent" | "stat";
  orderedAt: string;
  expectedTAT: string;
}

// Lab Result interface
export interface LabResult {
  id: string;
  orderId: string;
  performedBy: string;
  resultData: Record<string, any>;
  notes?: string;
  attachments?: Array<{
    id: string;
    name: string;
    url: string;
    type: string; // "image/jpeg", "application/pdf"
  }>;
  status: "draft" | "validated" | "released";
  validatedBy?: string;
  validatedAt?: string;
}
```

---

## 6. User Workflows

### Workflow 1: Process Lab Test (Complete Cycle)

**Actors:** Doctor, Lab Technician, Pathologist, Patient

**Steps:**

1. **Doctor orders test** (Provider Portal - outside scope)

   - Creates LabOrder record
   - Status: `pending`

2. **Technician accepts order** (Your implementation)

   ```
   GET /orders/pending
   → Display in queue

   Technician clicks "Accept"
   → PATCH /orders/{id}/accept
   → Status: pending → in-progress
   ```

3. **Sample collection** (Nurse/Phlebotomist)

   ```
   Nurse draws blood → labels with barcode
   → POST /samples
   → Sample tracked in system
   ```

4. **Lab processing**

   ```
   Technician receives sample
   → PATCH /samples/{id}/status (status: received)

   Runs test on analyzer
   → Machine outputs results

   Technician uploads results
   → POST /orders/{id}/results
   → Attach images/PDFs if applicable
   ```

5. **Validation** (Senior tech/pathologist)

   ```
   Validator reviews results
   → Checks accuracy
   → If critical values found:
     → POST /orders/{id}/flag

   → PATCH /orders/{id}/results (status: validated)
   ```

6. **Release**
   ```
   POST /orders/{id}/results/notify
   → Provider receives notification
   → Results visible in Provider Portal
   → If critical → SMS alert sent
   ```

### Workflow 2: Critical Result Handling

**Scenario:** Dangerously low blood glucose detected

**Steps:**

1. Technician uploads result: Glucose = 40 mg/dL (normal: 70-100)
2. System auto-detects critical value
3. POST /orders/{id}/flag
4. Validator confirms criticality
5. POST /orders/{id}/results/notify
6. System sends:
   - Urgent notification to doctor
   - SMS alert
   - Phone call reminder (future)
7. Result flagged in red in Provider Portal
8. Doctor sees alert immediately, takes action

### Workflow 3: Sample Rejection

**Scenario:** Blood sample clotted, cannot process

**Steps:**

1. Technician receives sample
2. Inspects → finds clots
3. POST /samples/{id}/reject
   - Reason: "Sample clotted"
   - Action: "Recollection required"
4. System:
   - Notifies ordering provider
   - Creates new lab order (linked to original)
   - Logs rejection for quality metrics
5. Nurse recollects blood from patient
6. Process continues with new sample

---

## 7. Security & Authorization

### Role-Based Access Control

```typescript
// Backend middleware
export const requireLab = (req, res, next) => {
  if (!["LAB_IMAGING", "LAB_ADMIN"].includes(req.user.role)) {
    return res.status(403).json({ error: "Access denied" });
  }
  next();
};

export const requireLabAdmin = (req, res, next) => {
  if (req.user.role !== "LAB_ADMIN") {
    return res.status(403).json({ error: "Admin access required" });
  }
  next();
};

// Route protection
router.get("/orders", requireLab, getOrders);
router.post("/test-catalog", requireLabAdmin, addTest);
```

### Tenant Isolation

```typescript
// Every query MUST filter by tenant
const orders = await prisma.labOrder.findMany({
  where: {
    tenantId: req.user.tenantId, // REQUIRED
  },
});
```

---

## 8. Implementation Roadmap

### Phase 1: Core Workflow (Week 1)

**Backend:**

- [ ] GET /dashboard
- [ ] GET /orders (all + filters)
- [ ] GET /orders/{id}
- [ ] PATCH /orders/{id}/accept
- [ ] PATCH /orders/{id}/in-progress

**Frontend:**

- [ ] Dashboard stats cards
- [ ] Orders queue table
- [ ] Order detail view
- [ ] Accept/Start buttons

### Phase 2: Results & Validation (Week 2)

**Backend:**

- [ ] POST /orders/{id}/results
- [ ] PATCH /orders/{id}/results
- [ ] POST /orders/{id}/flag
- [ ] POST /orders/{id}/results/notify
- [ ] Sample tracking APIs

**Frontend:**

- [ ] Results upload form
- [ ] Validation workflow
- [ ] Critical flagging
- [ ] Release workflow

### Phase 3: Admin Features (Week 3)

**Backend:**

- [ ] GET /test-catalog
- [ ] POST /test-catalog
- [ ] GET /equipment
- [ ] POST /equipment/{id}/maintenance
- [ ] Reports APIs

**Frontend:**

- [ ] LabAdminDashboard
- [ ] Test catalog management
- [ ] Equipment tracking
- [ ] Reports generation

---

## Quick Start Guide

### Create Test Data

```sql
-- Add lab test to catalog
INSERT INTO lab_test_catalog (id, organization_id, test_code, test_name, category, cost, turnaround_time) VALUES
  ('test-1', 'org-1', 'CBC-001', 'Complete Blood Count', 'Hematology', 3500, '4 hours');

-- Create test order
INSERT INTO lab_orders (id, encounter_id, ordering_provider_id, test_name, urgency, status) VALUES
  ('order-1', 'encounter-1', 'provider-1', 'Complete Blood Count', 'routine', 'pending');
```

---

**END OF DESIGN BRIEF**
