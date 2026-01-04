# VardMD Pharmacy Portal - Design Brief

**Project:** VardMD Healthcare Platform  
**Module:** Pharmacy Portal (Pharmacist & Admin)  
**Version:** 1.0  
**Date:** January 4, 2026  
**Target Roles:** `PHARMACY`, `PHARMACY_ADMIN`

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
9. [Testing Requirements](#testing-requirements)

---

## 1. Executive Summary

### Purpose

The Pharmacy Portal is a critical safety checkpoint in VardMD's healthcare workflow where pharmacists:

- Receive prescriptions from doctors (Provider Portal) and hospitals (EMR Portal)
- Verify medication safety through drug interaction checking
- Dispense medications to patients
- Manage drug inventory and supplier relationships
- Track expiring medications and stock levels

### Business Impact

**Without this system:**

- Paper prescriptions → lost, illegible, forged
- No drug interaction checking → preventable deaths
- Manual inventory → frequent stockouts
- No expiry tracking → legal liability from expired drugs

**With this system:**

- 100% digital prescription workflow
- Automated drug safety checks
- Real-time inventory management
- Complete audit trail for regulatory compliance

### User Roles

| Role             | Permissions                                                   | Use Cases                            |
| ---------------- | ------------------------------------------------------------- | ------------------------------------ |
| `PHARMACY`       | Process prescriptions, dispense medications, manage inventory | Pharmacists and pharmacy technicians |
| `PHARMACY_ADMIN` | Full access + financial reports + supplier management         | Pharmacy managers and administrators |

---

## 2. System Architecture

### Tech Stack

**Backend:**

- Runtime: Node.js + TypeScript
- Framework: Express.js
- Database: PostgreSQL
- ORM: Prisma
- Authentication: JWT tokens

**Frontend:**

- Framework: React + TypeScript
- UI Library: shadcn/ui (Tailwind CSS)
- State Management: React hooks
- HTTP Client: Axios/Fetch

### Multi-Tenant Architecture

Every API request and database query MUST include tenant filtering:

```typescript
// Backend - All queries must filter by tenantId
prisma.prescription.findMany({
  where: {
    tenantId: req.user.tenantId, // REQUIRED
  },
});

// Frontend - Tenant context from auth
const { user } = useAuth();
// user.tenantId automatically included in API calls
```

### Data Flow

```
Doctor prescribes → Provider Portal
                        ↓
                [Prescription created]
                        ↓
                Pharmacy Portal ← You implement this
                        ↓
        [Pharmacist reviews & dispenses]
                        ↓
        [Inventory auto-updated]
                        ↓
        [Patient receives medication]
```

---

## 3. Database Schema

### Core Tables (from Prisma Schema)

#### 1. Prescription Table

```prisma
model Prescription {
  id                String             @id @default(uuid())
  encounterId       String             @map("encounter_id")
  prescriberUserId  String             @map("prescriber_user_id")
  notes             String?
  createdAt         DateTime           @default(now()) @map("created_at")
  items             PrescriptionItem[]
  encounter         Encounter          @relation(fields: [encounterId], references: [id])
  prescriber        User               @relation(fields: [prescriberUserId], references: [id])
}
```

**Key Fields:**

- `encounterId`: Links to patient encounter (hospital visit)
- `prescriberUserId`: Doctor who wrote prescription
- `items`: Array of medications prescribed

#### 2. PrescriptionItem Table

```prisma
model PrescriptionItem {
  id             String         @id @default(uuid())
  prescriptionId String         @map("prescription_id")
  formularyId    String         @map("formulary_id")
  dose           String?
  frequency      String?        // "TID" (3x daily), "BID" (2x daily)
  duration       String?        // "7 days", "2 weeks"
  quantity       Decimal        @db.Decimal(12, 2)
  instruction    String?
  dispensings    Dispensing[]
  formulary      Formulary      @relation(fields: [formularyId], references: [id])
  prescription   Prescription   @relation(fields: [prescriptionId], references: [id])
}
```

**Key Fields:**

- `formularyId`: Links to drug catalog
- `quantity`: Amount to dispense
- `frequency`: How often to take (TID = 3x daily, BID = 2x daily)
- `dispensings`: Track when medication was given to patient

#### 3. Formulary Table (Drug Catalog)

```prisma
model Formulary {
  id                  String              @id @default(uuid())
  organizationId      String              @map("organization_id")
  code                String              // Drug code (e.g., "AMX500")
  name                String              // Drug name
  strength            String?             // "500mg", "10mg/ml"
  route               String?             // "Oral", "IV", "Topical"
  form                String?             // "Tablet", "Capsule", "Syrup"
  isActive            Boolean             @default(true)
  createdAt           DateTime            @default(now())
  updatedAt           DateTime            @updatedAt
  organization        Organization        @relation(fields: [organizationId], references: [id])
  pharmacyInventories PharmacyInventory[]
  prescriptionItems   PrescriptionItem[]
}
```

**Unique Constraint:** `organizationId + code` (each org has own formulary)

#### 4. PharmacyInventory Table

```prisma
model PharmacyInventory {
  id             String         @id @default(uuid())
  organizationId String         @map("organization_id")
  formularyId    String         @map("formulary_id")
  sku            String?
  batchNo        String         @map("batch_no")
  expiresOn      DateTime?      @map("expires_on")
  quantityOnHand Decimal        @map("quantity_on_hand") @db.Decimal(14, 2)
  reorderLevel   Decimal        @map("reorder_level") @db.Decimal(14, 2)
  createdAt      DateTime       @default(now())
  updatedAt      DateTime       @updatedAt
  dispensings    Dispensing[]
  inventoryTxns  InventoryTxn[]
  formulary      Formulary      @relation(fields: [formularyId], references: [id])
  organization   Organization   @relation(fields: [organizationId], references: [id])
}
```

**Key Features:**

- **Batch tracking:** Each drug shipment has unique batch number
- **Expiry tracking:** `expiresOn` triggers alerts
- **Auto-reorder:** When `quantityOnHand < reorderLevel`, alert admin

**Unique Constraint:** `organizationId + formularyId + batchNo`

#### 5. Dispensing Table (Audit Trail)

```prisma
model Dispensing {
  id                 String            @id @default(uuid())
  prescriptionItemId String            @map("prescription_item_id")
  inventoryId        String            @map("inventory_id")
  qtyDispensed       Decimal           @map("qty_dispensed") @db.Decimal(14, 2)
  dispensedBy        String            @map("dispensed_by")
  dispensedAt        DateTime          @map("dispensed_at")
  dispensedByUser    User              @relation(fields: [dispensedBy], references: [id])
  inventory          PharmacyInventory @relation(fields: [inventoryId], references: [id])
  prescriptionItem   PrescriptionItem  @relation(fields: [prescriptionItemId], references: [id])
}
```

**Purpose:**

- Permanent record of who dispensed what medication
- Required for drug recalls ("Which patients got Batch ABC123?")
- Regulatory compliance (controlled substance tracking)

#### 6. InventoryTxn Table (Stock Movements)

```prisma
model InventoryTxn {
  id                        String            @id @default(uuid())
  inventoryId               String            @map("inventory_id")
  txnType                   String            @map("txn_type") // "RECEIPT", "DISPENSING", "ADJUSTMENT", "RETURN"
  qty                       Decimal           @db.Decimal(14, 2)
  reason                    String?
  relatedPrescriptionItemId String?           @map("related_prescription_item_id")
  createdAt                 DateTime          @default(now())
  inventory                 PharmacyInventory @relation(fields: [inventoryId], references: [id])
  relatedPrescriptionItem   PrescriptionItem? @relation(fields: [relatedPrescriptionItemId], references: [id])
}
```

**Transaction Types:**

- `RECEIPT`: Stock arrived from supplier (+qty)
- `DISPENSING`: Medication given to patient (-qty)
- `ADJUSTMENT`: Stock count correction (damaged, stolen, etc.)
- `RETURN`: Patient returned medication (+qty)

### Relationships Diagram

```
User (Doctor)
  └── Prescription
        └── PrescriptionItem
              ├── Formulary (Drug Catalog)
              │     └── PharmacyInventory (Stock)
              │           └── InventoryTxn (Movement Log)
              └── Dispensing (Audit Trail)
                    └── User (Pharmacist)
```

---

## 4. API Specifications

### Base URL

```
/api/v1/pharmacy
```

### Authentication

All endpoints require JWT authentication:

```typescript
// Headers
Authorization: Bearer {JWT_TOKEN}
Content-Type: application/json

// Token payload includes:
{
  userId: string,
  tenantId: string,
  role: "PHARMACY" | "PHARMACY_ADMIN",
  email: string
}
```

### API Groups

#### 4.1 Dashboard Statistics (Priority: HIGH)

**GET /dashboard**

Returns pharmacy dashboard summary.

**Response:**

```json
{
  "pendingPrescriptions": 15,
  "dispensedToday": 42,
  "lowStockItems": 8,
  "inventoryValue": 8750000,
  "topMovingDrugs": [
    {
      "drugName": "Paracetamol 500mg",
      "dispensedCount": 156,
      "revenue": 234000
    }
  ],
  "recentActivity": [
    {
      "patientName": "Chioma Okeke",
      "action": "Prescription dispensed",
      "timestamp": "2026-01-04T14:30:00Z"
    }
  ]
}
```

**GET /dashboard/prescriptions-today**

Returns count of prescriptions received today.

**Response:**

```json
{
  "count": 42,
  "urgent": 12,
  "routine": 30
}
```

**GET /dashboard/pending-count**

Returns pending prescription count.

**Response:**

```json
{
  "pending": 15,
  "validated": 8,
  "readyForDispense": 7
}
```

**GET /dashboard/low-stock-alerts**

Returns low stock alert count.

**Response:**

```json
{
  "count": 8,
  "critical": 3,
  "expiringSoon": 2
}
```

#### 4.2 Prescription Queue Management (Priority: HIGH)

**GET /prescriptions**

Get all prescriptions with filters.

**Query Parameters:**

- `status` (optional): `pending`, `validated`, `ready_for_dispense`, `dispensed`, `completed`, `rejected`
- `priority` (optional): `routine`, `urgent`, `stat`
- `page` (default: 1)
- `limit` (default: 20)

**Response:**

```json
{
  "data": [
    {
      "id": "rx_001",
      "rxNumber": "RX-2025-001234",
      "patient": {
        "id": "pat_001",
        "name": "Chioma Okeke",
        "mrn": "MRN-10001",
        "age": 34,
        "allergies": ["Penicillin"]
      },
      "provider": {
        "id": "prov_001",
        "name": "Dr. Amaka Okafor"
      },
      "status": "pending",
      "priority": "routine",
      "prescribedAt": "2026-01-04T09:00:00Z",
      "totalAmount": 12500,
      "billingStatus": "pending",
      "items": [
        {
          "id": "item_001",
          "drugName": "Amoxicillin",
          "strength": "500mg",
          "form": "Capsules",
          "dosage": "1 capsule",
          "frequency": "TID",
          "duration": "7 days",
          "quantity": 21,
          "instructions": "Take with food"
        }
      ]
    }
  ],
  "pagination": {
    "total": 50,
    "page": 1,
    "limit": 20,
    "pages": 3
  }
}
```

**GET /prescriptions/{id}**

Get prescription details.

**Response:** Same as individual prescription object above, with additional fields:

```json
{
  "validatedBy": "user_123",
  "validatedAt": "2026-01-04T09:15:00Z",
  "pharmacistNotes": "Checked interactions - safe to dispense",
  "insurance": {
    "provider": "NHIS",
    "policyNumber": "NHIS-12345",
    "coverage": 70
  }
}
```

**PATCH /prescriptions/{id}/review**

Clinical review of prescription (check interactions, allergies).

**Request Body:**

```json
{
  "pharmacistNotes": "Checked patient allergies and current medications. No interactions found.",
  "interactionChecked": true
}
```

**Response:**

```json
{
  "id": "rx_001",
  "status": "validated",
  "validatedBy": "user_123",
  "validatedAt": "2026-01-04T09:15:00Z"
}
```

**PATCH /prescriptions/{id}/prepare**

Start preparing prescription.

**Request Body:**

```json
{
  "notes": "Preparing medication for patient pickup"
}
```

**Response:**

```json
{
  "id": "rx_001",
  "status": "ready_for_dispense"
}
```

**PATCH /prescriptions/{id}/dispense**

Dispense medication to patient.

**Request Body:**

```json
{
  "paymentMethod": "cash",
  "amountPaid": 3150,
  "insuranceCovered": 7350,
  "patientSignature": true,
  "counselingProvided": true,
  "counselingNotes": "Explained dosage, side effects, and storage instructions"
}
```

**Response:**

```json
{
  "id": "rx_001",
  "status": "dispensed",
  "dispensedAt": "2026-01-04T14:30:00Z",
  "dispensedBy": "user_123",
  "receiptNumber": "REC-2025-8892"
}
```

**Backend Logic:**

1. Validate prescription status is `ready_for_dispense`
2. Create `Dispensing` record for each medication
3. Update `PharmacyInventory.quantityOnHand` (subtract dispensed qty)
4. Create `InventoryTxn` records with type `DISPENSING`
5. Check if stock < reorder level → trigger alert
6. Update prescription status to `dispensed`

**PATCH /prescriptions/{id}/reject**

Reject prescription (cannot fulfill).

**Request Body:**

```json
{
  "reason": "Out of stock - Amoxicillin not available",
  "recommendedAction": "Contact prescriber for alternative medication"
}
```

**Response:**

```json
{
  "id": "rx_001",
  "status": "rejected",
  "rejectionReason": "Out of stock - Amoxicillin not available"
}
```

**POST /prescriptions/{id}/substitute**

Suggest substitute drug (generic alternative).

**Request Body:**

```json
{
  "originalDrugId": "formulary_001",
  "substituteDrugId": "formulary_002",
  "reason": "Original brand out of stock, generic equivalent available"
}
```

**Response:**

```json
{
  "substitution": {
    "original": {
      "name": "Augmentin 625mg",
      "price": 420
    },
    "substitute": {
      "name": "Amoxiclav 625mg",
      "price": 280,
      "savings": 140
    }
  },
  "requiresPrescriberApproval": true
}
```

#### 4.3 Drug Interaction Checking (Priority: HIGH - Safety Feature)

**POST /interactions/check**

Check drug interactions before dispensing.

**Request Body:**

```json
{
  "patientId": "pat_001",
  "newMedications": [
    {
      "drugCode": "ASA325",
      "drugName": "Aspirin 325mg"
    }
  ]
}
```

**Response:**

```json
{
  "safe": false,
  "alerts": [
    {
      "severity": "critical",
      "type": "drug-drug-interaction",
      "description": "Aspirin + Warfarin = Severe bleeding risk",
      "recommendation": "Contact prescribing physician immediately",
      "currentMedication": "Warfarin 5mg",
      "newMedication": "Aspirin 325mg"
    }
  ],
  "allergyAlerts": [],
  "dosageWarnings": []
}
```

**Backend Logic:**

1. Query patient's current medications (recent `Dispensing` records)
2. Check drug interaction database
3. Check patient allergies from `Patient` or `User` table
4. Validate dosage ranges
5. Return alerts categorized by severity

**GET /interactions/alerts**

Get active interaction alerts.

**Response:**

```json
{
  "alerts": [
    {
      "id": "alert_001",
      "patientName": "Kemi Adebisi",
      "severity": "critical",
      "description": "Drug interaction detected",
      "createdAt": "2026-01-04T10:00:00Z"
    }
  ]
}
```

**GET /interactions/history**

Get interaction check audit log.

**Response:**

```json
{
  "history": [
    {
      "id": "check_001",
      "prescriptionId": "rx_001",
      "checkedBy": "user_123",
      "checkedAt": "2026-01-04T09:15:00Z",
      "alerts": 0,
      "status": "safe"
    }
  ]
}
```

#### 4.4 Inventory Management (Priority: HIGH)

**GET /inventory**

Get current inventory with filters.

**Query Parameters:**

- `lowStock` (boolean): Only show low stock items
- `expiring` (boolean): Only show expiring items
- `category` (string): Filter by drug category
- `search` (string): Search drug name

**Response:**

```json
{
  "data": [
    {
      "id": "inv_001",
      "drug": {
        "id": "formulary_001",
        "name": "Paracetamol 500mg",
        "code": "PAR500",
        "category": "Pain Relief"
      },
      "batchNo": "PAR2025A",
      "quantityOnHand": 245,
      "reorderLevel": 100,
      "expiresOn": "2025-06-15",
      "unitPrice": 150,
      "costPrice": 105,
      "totalValue": 36750,
      "status": "adequate"
    },
    {
      "id": "inv_002",
      "drug": {
        "id": "formulary_002",
        "name": "Insulin Glargine 100IU/ml",
        "code": "INS100",
        "category": "Diabetes"
      },
      "batchNo": "INS2025A",
      "quantityOnHand": 5,
      "reorderLevel": 20,
      "expiresOn": "2025-12-15",
      "status": "critical"
    }
  ]
}
```

**POST /inventory**

Add stock to inventory (when shipment arrives).

**Request Body:**

```json
{
  "formularyId": "formulary_001",
  "batchNo": "PAR2025C",
  "quantity": 200,
  "expiresOn": "2025-09-20",
  "costPrice": 105,
  "supplier": "Lagos Pharmaceuticals",
  "purchaseOrderId": "po_001"
}
```

**Response:**

```json
{
  "id": "inv_003",
  "message": "Stock added successfully",
  "inventoryTxn": {
    "id": "txn_001",
    "type": "RECEIPT",
    "quantity": 200
  }
}
```

**Backend Logic:**

1. Create new `PharmacyInventory` record
2. Create `InventoryTxn` with type `RECEIPT`
3. If related to PO, update PO status

**PATCH /inventory/{id}**

Update stock levels (manual adjustment).

**Request Body:**

```json
{
  "quantityOnHand": 230,
  "reason": "Stock count adjustment - 15 units damaged"
}
```

**Response:**

```json
{
  "id": "inv_001",
  "quantityOnHand": 230,
  "inventoryTxn": {
    "id": "txn_002",
    "type": "ADJUSTMENT",
    "quantity": -15,
    "reason": "Stock count adjustment - 15 units damaged"
  }
}
```

**GET /inventory/low-stock**

Get low stock items (quantity < reorder level).

**Response:**

```json
{
  "alerts": [
    {
      "id": "inv_002",
      "drugName": "Insulin Glargine 100IU/ml",
      "currentStock": 5,
      "reorderLevel": 20,
      "deficit": 15,
      "priority": "critical",
      "supplier": "Nigerian Drug Company"
    }
  ]
}
```

**GET /inventory/expiring**

Get medications expiring soon (within 90 days).

**Response:**

```json
{
  "expiring": [
    {
      "id": "inv_003",
      "drugName": "Ciprofloxacin 500mg",
      "batchNo": "CIP2024C",
      "quantity": 45,
      "expiresOn": "2025-02-28",
      "daysUntilExpiry": 30,
      "action": "Discount pricing or return to supplier"
    }
  ]
}
```

**GET /inventory/{id}/history**

Get stock movement history for specific drug.

**Response:**

```json
{
  "history": [
    {
      "id": "txn_001",
      "type": "RECEIPT",
      "quantity": 200,
      "balanceAfter": 445,
      "reason": "Purchase order PO-2025-0156",
      "createdAt": "2026-01-04T08:00:00Z"
    },
    {
      "id": "txn_002",
      "type": "DISPENSING",
      "quantity": -10,
      "balanceAfter": 435,
      "prescriptionId": "rx_001",
      "dispensedBy": "user_123",
      "createdAt": "2026-01-04T14:30:00Z"
    }
  ]
}
```

#### 4.5 Drug Formulary (Priority: MEDIUM)

**GET /formulary**

Get drug catalog.

**Query Parameters:**

- `search` (string): Search drug name
- `category` (string): Filter by category
- `active` (boolean): Only active drugs

**Response:**

```json
{
  "data": [
    {
      "id": "formulary_001",
      "code": "PAR500",
      "name": "Paracetamol 500mg",
      "genericName": "Acetaminophen",
      "strength": "500mg",
      "form": "Tablet",
      "route": "Oral",
      "category": "Pain Relief",
      "manufacturer": "Emzor",
      "isActive": true
    }
  ]
}
```

**POST /formulary** (PHARMACY_ADMIN only)

Add drug to formulary.

**Request Body:**

```json
{
  "code": "AMX500",
  "name": "Amoxicillin 500mg",
  "genericName": "Amoxicillin",
  "strength": "500mg",
  "form": "Capsule",
  "route": "Oral",
  "category": "Antibiotics",
  "manufacturer": "GSK"
}
```

**GET /formulary/{id}/alternatives**

Get generic alternatives for brand-name drug.

**Response:**

```json
{
  "original": {
    "id": "formulary_001",
    "name": "Augmentin 625mg",
    "price": 420
  },
  "alternatives": [
    {
      "id": "formulary_002",
      "name": "Amoxiclav 625mg",
      "price": 280,
      "savings": 140,
      "savingsPercent": 33,
      "bioequivalent": true
    }
  ]
}
```

#### 4.6 Supplier Management (Priority: MEDIUM - PHARMACY_ADMIN only)

**GET /suppliers**

Get supplier list.

**Response:**

```json
{
  "data": [
    {
      "id": "supplier_001",
      "name": "Lagos Pharmaceuticals Ltd",
      "contactPerson": "Mr. Emeka Okafor",
      "phone": "+234 803 123 4567",
      "email": "supply@lagospharm.ng",
      "productsSupplied": 145,
      "lastDelivery": "2025-01-18",
      "status": "active"
    }
  ]
}
```

**POST /suppliers**

Add new supplier.

**Request Body:**

```json
{
  "name": "Nigerian Drug Company",
  "contactPerson": "Dr. Amina Kano",
  "phone": "+234 901 234 5678",
  "email": "orders@nigeriandrugco.com",
  "address": "123 Marina Street, Lagos"
}
```

#### 4.7 Purchase Orders (Priority: MEDIUM - PHARMACY_ADMIN only)

**GET /purchase-orders**

Get all purchase orders.

**Response:**

```json
{
  "data": [
    {
      "id": "po_001",
      "poNumber": "PO-2025-0156",
      "supplier": {
        "id": "supplier_001",
        "name": "Nigerian Drug Company"
      },
      "orderDate": "2026-01-20",
      "expectedDelivery": "2026-01-25",
      "status": "pending",
      "totalAmount": 51923,
      "items": [
        {
          "drugName": "Metformin 500mg",
          "quantity": 100,
          "unitPrice": 126,
          "totalPrice": 12600
        }
      ]
    }
  ]
}
```

**POST /purchase-orders**

Create purchase order.

**Request Body:**

```json
{
  "supplierId": "supplier_001",
  "expectedDelivery": "2026-01-25",
  "items": [
    {
      "formularyId": "formulary_001",
      "quantity": 100,
      "unitPrice": 126
    }
  ]
}
```

**PATCH /purchase-orders/{id}/receive**

Receive shipment and update inventory.

**Request Body:**

```json
{
  "receivedDate": "2026-01-24",
  "items": [
    {
      "formularyId": "formulary_001",
      "quantityReceived": 100,
      "batchNo": "MET2025D",
      "expiresOn": "2026-06-15"
    }
  ]
}
```

**Backend Logic:**

1. Update PO status to "received"
2. For each item, create/update `PharmacyInventory`
3. Create `InventoryTxn` records with type `RECEIPT`
4. Auto-clear low stock alerts if quantity now adequate

#### 4.8 Dispensing History (Priority: LOW)

**GET /dispensing**

Get dispensing history.

**Query Parameters:**

- `startDate`, `endDate`
- `patientId`
- `drugId`

**Response:**

```json
{
  "data": [
    {
      "id": "disp_001",
      "dispensingNumber": "DISP-2025-4521",
      "patient": {
        "id": "pat_001",
        "name": "Chioma Okeke"
      },
      "drug": {
        "id": "formulary_001",
        "name": "Amoxicillin 500mg"
      },
      "quantity": 21,
      "batchNo": "AMX2025B",
      "dispensedBy": "user_123",
      "dispensedAt": "2026-01-04T14:30:00Z",
      "amountPaid": 3150
    }
  ]
}
```

**GET /dispensing/by-patient/{patientId}**

Get patient medication history (last 6 months).

**Response:**

```json
{
  "patient": {
    "id": "pat_001",
    "name": "Chioma Okeke",
    "mrn": "MRN-10001"
  },
  "medications": [
    {
      "drugName": "Amoxicillin 500mg",
      "dispensedAt": "2026-01-04T14:30:00Z",
      "prescribedBy": "Dr. Amaka Okafor",
      "quantity": 21
    }
  ]
}
```

---

## 5. Frontend Components

### Component Structure

```
src/components/pharmacy/
├── PharmacyDashboard.tsx          (962 lines - Main pharmacist view)
├── PharmacyAdminDashboard.tsx     (1186 lines - Admin view)
├── RxQueue.tsx                     (Prescription queue)
├── InventoryDispense.tsx          (Dispensing workflow)
├── DrugAvailability.tsx           (Inventory browser)
├── PharmacyReports.tsx            (Reports & analytics)
├── PharmacySettings.tsx           (Settings)
├── WalkInSales.tsx                (OTC sales)
└── modals/
    └── PharmacyAdminModals.tsx    (Admin modals)
```

### Key Frontend Code Reference

**File:** `Frontend-VardMD/src/components/pharmacy/PharmacyDashboard.tsx`

**Key Interfaces:**

```typescript
// Lines 44-73: Prescription interface
export interface Prescription {
  id: string;
  rxNumber: string;
  patientName: string;
  patientMrn: string;
  patientId: string;
  providerName: string;
  providerId: string;
  consultationId?: string;
  encounterId?: string;
  tenantId: string;
  status:
    | "pending"
    | "validated"
    | "queried"
    | "ready_for_dispense"
    | "partial"
    | "dispensed"
    | "completed"
    | "rejected";
  priority: "routine" | "urgent" | "stat";
  prescribedAt: string;
  validatedAt?: string;
  dispensedAt?: string;
  notes?: string;
  pharmacistNotes?: string;
  allergies?: string[];
  insurance?: {
    provider: string;
    policyNumber: string;
    coverage: number;
  };
  billingStatus: "pending" | "covered" | "partial" | "paid" | "overdue";
  totalAmount: number;
  items: PrescriptionItem[];
}

// Lines 75-95: Prescription item interface
export interface PrescriptionItem {
  id: string;
  prescriptionId: string;
  drugCode: string;
  drugName: string;
  genericName?: string;
  strength: string;
  form: string;
  dosage: string;
  frequency: string;
  duration: string;
  quantity: number;
  quantityDispensed: number;
  instructions: string;
  substitutable: boolean;
  substitutedWith?: string;
  substitutionReason?: string;
  unitPrice: number;
  totalPrice: number;
  formularyId?: string;
}

// Lines 111-120: Inventory item interface
export interface InventoryItem {
  id: string;
  medicationName: string;
  genericName: string;
  stockQuantity: number;
  batchNumber: string;
  expiryDate: string;
  unitPrice: number;
  supplier: string;
}
```

**Dashboard Stats (Lines 168-173):**

```typescript
const dashboardStats = {
  pendingPrescriptions: 15, // ← GET /dashboard/pending-count
  dispensedToday: 42, // ← GET /dashboard/prescriptions-today
  lowStockItems: 8, // ← GET /dashboard/low-stock-alerts
  inventoryValue: 8750000, // ← GET /dashboard
};
```

**Inventory Alerts (Lines 211-249):**

```typescript
const inventoryAlerts = [
  {
    id: "1",
    drugName: "Insulin Glargine 100IU/ml",
    currentStock: 5,
    reorderLevel: 20,
    expiryDate: "2025-12-15",
    alertType: "critical", // critical | low | expiring
    batchNumber: "INS2025A",
  },
];
```

**Prescription Queue (Lines 256-329):**

Mock prescription showing required data structure for API responses.

**UI Sections (Lines 142-151):**

```typescript
const menuItems = [
  { id: "dashboard", label: "Dashboard", icon: Home },
  { id: "queue", label: "Rx Queue", icon: Pill },
  { id: "inventory", label: "Inventory", icon: Package },
  { id: "dispense", label: "Dispense", icon: CheckCircle },
  { id: "walk-in-sales", label: "Walk-In Sales", icon: ShoppingCart },
  { id: "nurse-requests", label: "Internal Chat", icon: MessageSquare },
  { id: "reports", label: "Reports", icon: BarChart3 },
  { id: "settings", label: "Settings", icon: Settings },
];
```

### UI/UX Requirements

**Design System:**

- Use shadcn/ui components (already in project)
- Tailwind CSS for styling
- Dark mode support (already implemented)
- Mobile responsive (use `useIsMobile()` hook)

**Color Coding (Safety-Critical):**

- Red/Destructive: Urgent prescriptions, critical stock, drug interactions
- Amber/Warning: Low stock, expiring medications
- Green/Success: Dispensed, adequate stock
- Blue/Primary: Normal workflow states

**Accessibility:**

- All interactive elements must have aria-labels
- Color is not the only indicator (use icons + text)
- Keyboard navigation support
- Screen reader compatible

---

## 6. User Workflows

### Workflow 1: Prescription Processing (Complete Cycle)

**Actors:** Doctor, Pharmacist, Patient

**Steps:**

1. **Doctor prescribes** (Provider Portal - outside your scope)

   - Creates `Prescription` + `PrescriptionItem` records
   - Status: `pending`

2. **Pharmacist reviews** (Your implementation)

   ```
   GET /prescriptions/pending
   → Display in queue with priority badges

   Pharmacist clicks "Review"
   → POST /interactions/check (safety check)
   → Display any alerts

   If safe:
     PATCH /prescriptions/{id}/review
     → Status: pending → validated

   If dangerous:
     PATCH /prescriptions/{id}/reject
     → Status: pending → rejected
     → Notify doctor
   ```

3. **Prepare medication**

   ```
   GET /inventory (check stock)
   → Verify drug available in stock

   Pharmacist clicks "Prepare"
   → PATCH /prescriptions/{id}/prepare
   → Status: validated → ready_for_dispense
   ```

4. **Patient arrives**

   ```
   Pharmacist verifies patient ID
   Provides counseling (dosage, side effects)

   Patient agrees & pays
   → PATCH /prescriptions/{id}/dispense
   → Creates Dispensing records
   → Updates inventory (-qty)
   → Status: ready_for_dispense → dispensed
   ```

5. **Auto-inventory check** (Backend automatic)
   ```
   After dispensing:
   → Check if quantityOnHand < reorderLevel
   → If yes, add to low stock alerts
   → Admin sees alert in dashboard
   ```

### Workflow 2: Generic Substitution

**Scenario:** Brand-name drug out of stock

**Steps:**

1. Pharmacist sees prescription for "Augmentin 625mg"
2. GET /inventory → Stock: 0 units
3. GET /formulary/{id}/alternatives → Show generics
4. Pharmacist sees "Amoxiclav 625mg" (₦280 vs ₦420)
5. POST /prescriptions/{id}/substitute
6. System suggests calling doctor for approval
7. After approval, dispense generic instead

### Workflow 3: Critical Drug Interaction Alert

**Scenario:** Patient already on Warfarin, doctor prescribes Aspirin

**Steps:**

1. Pharmacist reviews prescription
2. POST /interactions/check
   ```json
   {
     "patientId": "pat_001",
     "newMedications": [{ "drugCode": "ASA325", "drugName": "Aspirin" }]
   }
   ```
3. API returns:
   ```json
   {
     "safe": false,
     "alerts": [
       {
         "severity": "critical",
         "description": "Aspirin + Warfarin = Severe bleeding risk"
       }
     ]
   }
   ```
4. Pharmacist sees RED ALERT modal
5. MUST call doctor before dispensing
6. Doctor changes prescription to Paracetamol
7. New prescription arrives, passes safety check

### Workflow 4: Reorder Workflow (Admin)

**Scenario:** Metformin stock low

**Steps:**

1. System auto-detects: `quantityOnHand (8) < reorderLevel (30)`
2. GET /inventory/low-stock → Shows alert in admin dashboard
3. Admin clicks "Reorder"
4. POST /purchase-orders
   ```json
   {
     "supplierId": "supplier_001",
     "items": [
       { "formularyId": "metformin_id", "quantity": 100, "unitPrice": 126 }
     ]
   }
   ```
5. Supplier delivers 5 days later
6. PATCH /purchase-orders/{id}/receive
   - Creates PharmacyInventory record
   - Creates InventoryTxn (type: RECEIPT)
   - Updates stock: 8 → 108 units
   - Clears low stock alert

---

## 7. Security & Authorization

### Role-Based Access Control (RBAC)

**Backend Middleware:**

```typescript
// src/middlewares/auth.middleware.ts

export const requirePharmacy = (req, res, next) => {
  if (!["PHARMACY", "PHARMACY_ADMIN"].includes(req.user.role)) {
    return res.status(403).json({ error: "Access denied" });
  }
  next();
};

export const requirePharmacyAdmin = (req, res, next) => {
  if (req.user.role !== "PHARMACY_ADMIN") {
    return res.status(403).json({ error: "Admin access required" });
  }
  next();
};
```

**Route Protection:**

```typescript
// src/routes/pharmacy.routes.ts

router.get("/dashboard", requirePharmacy, getDashboard);
router.get("/prescriptions", requirePharmacy, getPrescriptions);
router.patch(
  "/prescriptions/:id/dispense",
  requirePharmacy,
  dispensePrescription
);

// Admin-only routes
router.post("/formulary", requirePharmacyAdmin, addDrug);
router.post("/suppliers", requirePharmacyAdmin, addSupplier);
router.post("/purchase-orders", requirePharmacyAdmin, createPurchaseOrder);
```

### Data Isolation

**Every query MUST filter by tenant:**

```typescript
// BAD - Security vulnerability
const prescriptions = await prisma.prescription.findMany();

// GOOD - Tenant-isolated
const prescriptions = await prisma.prescription.findMany({
  where: {
    tenantId: req.user.tenantId, // From JWT token
  },
});
```

### Audit Logging

**All critical actions must be logged:**

```typescript
// Create audit log for dispensing
await prisma.auditLog.create({
  data: {
    tenantId: req.user.tenantId,
    actorUserId: req.user.userId,
    action: "DISPENSE_MEDICATION",
    entity: "Prescription",
    entityId: prescriptionId,
    metadata: {
      prescriptionNumber: prescription.rxNumber,
      patientId: prescription.patientId,
      medications: prescription.items.map((item) => item.drugName),
    },
  },
});
```

**Audit these actions:**

- Prescription dispensing
- Inventory adjustments
- Drug interaction overrides
- Supplier management
- Purchase orders

---

## 8. Implementation Roadmap

### Phase 1: Core Prescription Workflow (Week 1)

**Priority: CRITICAL**

**Backend:**

- [ ] GET /dashboard (basic stats)
- [ ] GET /prescriptions (all + filters)
- [ ] GET /prescriptions/{id}
- [ ] PATCH /prescriptions/{id}/review
- [ ] PATCH /prescriptions/{id}/prepare
- [ ] PATCH /prescriptions/{id}/dispense
- [ ] PATCH /prescriptions/{id}/reject

**Frontend:**

- [ ] Dashboard stats cards
- [ ] Prescription queue table
- [ ] Prescription detail modal
- [ ] Status update buttons
- [ ] Basic notifications (toast)

**Database:**

- [ ] Ensure Prescription, PrescriptionItem, Dispensing tables exist
- [ ] Add indexes on tenantId, status, prescribedAt

**Testing:**

- [ ] Create test prescriptions in database
- [ ] Test status transitions
- [ ] Test tenant isolation

### Phase 2: Inventory & Safety (Week 2)

**Priority: HIGH**

**Backend:**

- [ ] GET /inventory
- [ ] GET /inventory/low-stock
- [ ] GET /inventory/expiring
- [ ] PATCH /inventory/{id} (adjust stock after dispensing)
- [ ] POST /interactions/check
- [ ] GET /interactions/alerts

**Frontend:**

- [ ] Inventory browser
- [ ] Low stock alerts card
- [ ] Expiring medications card
- [ ] Drug interaction modal (red alert)
- [ ] Stock level indicators

**Database:**

- [ ] PharmacyInventory table
- [ ] InventoryTxn table
- [ ] Trigger for auto-creating txns

**Testing:**

- [ ] Test inventory deduction on dispense
- [ ] Test low stock alert triggering
- [ ] Test drug interaction checking

### Phase 3: Admin Features (Week 3)

**Priority: MEDIUM**

**Backend:**

- [ ] GET /formulary
- [ ] POST /formulary (admin)
- [ ] GET /formulary/{id}/alternatives
- [ ] GET /suppliers
- [ ] POST /suppliers
- [ ] GET /purchase-orders
- [ ] POST /purchase-orders
- [ ] PATCH /purchase-orders/{id}/receive

**Frontend:**

- [ ] PharmacyAdminDashboard
- [ ] Drug catalog management
- [ ] Supplier management
- [ ] Purchase order creation
- [ ] Shipment receiving workflow

**Database:**

- [ ] Formulary table seeded with common drugs
- [ ] Organization supplier relationships

**Testing:**

- [ ] Test PO creation
- [ ] Test receiving shipment (inventory update)
- [ ] Test generic substitution

### Phase 4: Reports & Analytics (Week 4)

**Priority: LOW**

**Backend:**

- [ ] GET /reports/dispensing
- [ ] GET /reports/inventory
- [ ] GET /reports/financial
- [ ] GET /dispensing (history)
- [ ] GET /dispensing/by-patient/{id}

**Frontend:**

- [ ] Reports dashboard
- [ ] Charts (dispensing trends, inventory distribution)
- [ ] Export to CSV/PDF
- [ ] Patient medication history

**Testing:**

- [ ] Test report generation
- [ ] Test date range filters
- [ ] Test export functionality

---

## 9. Testing Requirements

### Unit Tests

**Backend:**

```typescript
// Example: prescriptions.service.test.ts

describe("PrescriptionService", () => {
  it("should dispense prescription and update inventory", async () => {
    const prescription = await createTestPrescription();
    const inventoryBefore = await getInventory(
      prescription.items[0].formularyId
    );

    await prescriptionService.dispense(prescription.id, {
      dispensedBy: "user_123",
      paymentMethod: "cash",
    });

    const inventoryAfter = await getInventory(
      prescription.items[0].formularyId
    );
    expect(inventoryAfter.quantityOnHand).toBe(
      inventoryBefore.quantityOnHand - prescription.items[0].quantity
    );
  });

  it("should reject dispensing if stock insufficient", async () => {
    const prescription = await createTestPrescription({ quantity: 100 });
    await setInventoryStock(prescription.items[0].formularyId, 5);

    await expect(
      prescriptionService.dispense(prescription.id, {})
    ).rejects.toThrow("Insufficient stock");
  });
});
```

**Frontend:**

```typescript
// Example: PharmacyDashboard.test.tsx

describe("PharmacyDashboard", () => {
  it("should display pending prescriptions count", async () => {
    mockAPI("/dashboard/pending-count", { count: 15 });

    render(<PharmacyDashboard />);

    expect(await screen.findByText("15")).toBeInTheDocument();
    expect(screen.getByText("Pending Prescriptions")).toBeInTheDocument();
  });

  it("should show critical alert for drug interaction", async () => {
    mockAPI("/interactions/check", {
      safe: false,
      alerts: [{ severity: "critical", description: "Warfarin + Aspirin" }],
    });

    const { result } = renderHook(() => useInteractionCheck());
    await act(() => result.current.checkInteractions("pat_001", ["ASA325"]));

    expect(screen.getByText(/critical/i)).toBeInTheDocument();
  });
});
```

### Integration Tests

**Test Scenarios:**

1. **Complete prescription workflow:**

   - Create prescription
   - Review (passes safety check)
   - Prepare
   - Dispense
   - Verify inventory updated
   - Verify dispensing record created

2. **Drug interaction prevention:**

   - Create patient with existing medication
   - Attempt to dispense interacting drug
   - Verify rejection

3. **Reorder workflow:**
   - Reduce inventory below reorder level
   - Verify alert appears
   - Create purchase order
   - Receive shipment
   - Verify inventory updated
   - Verify alert cleared

### Manual Testing Checklist

**Pharmacist Role:**

- [ ] Can view dashboard stats
- [ ] Can see prescription queue
- [ ] Can review prescriptions
- [ ] Can dispense medications
- [ ] Can reject prescriptions
- [ ] Can view inventory
- [ ] Cannot access admin features

**Admin Role:**

- [ ] All pharmacist features work
- [ ] Can add drugs to formulary
- [ ] Can manage suppliers
- [ ] Can create purchase orders
- [ ] Can receive shipments
- [ ] Can view financial reports

**Security:**

- [ ] Tenant isolation (User A cannot see User B's data)
- [ ] Role enforcement (Pharmacist cannot access admin routes)
- [ ] JWT expiration handled gracefully
- [ ] CORS configured correctly

---

## 10. Additional Resources

### Reference Files

**Frontend:**

- Main file: `/Frontend-VardMD/src/components/pharmacy/PharmacyDashboard.tsx` (962 lines)
- Admin file: `/Frontend-VardMD/src/components/pharmacy/PharmacyAdminDashboard.tsx` (1186 lines)

**Backend:**

- Database schema: `/Backend-VardMD/prisma/schema.prisma`
- Example routes: `/Backend-VardMD/src/routes/` (check other portals for patterns)

**Documentation:**

- API Guide: `/PHARMACY_API_GUIDE.md` (1019 lines)
- Database Schema: Previously created in vardmd-docs

### Common Patterns in VardMD

**API Response Format:**

```typescript
// Success
{
  data: T,
  message?: string
}

// Error
{
  error: string,
  details?: any
}

// Paginated
{
  data: T[],
  pagination: {
    total: number,
    page: number,
    limit: number,
    pages: number
  }
}
```

**Date Handling:**

- All dates stored as ISO 8601 strings in PostgreSQL
- Frontend displays in local timezone (Africa/Lagos default)

**Currency:**

- All amounts stored in kobo/cents (multiply by 100)
- Display as Naira with ₦ symbol

---

## Quick Start Guide

### Setup

1. **Backend:**

   ```bash
   cd Backend-VardMD
   npm install
   npx prisma generate
   npx prisma migrate dev
   npm run dev
   ```

2. **Frontend:**
   ```bash
   cd Frontend-VardMD
   npm install
   npm run dev
   ```

### Create Test Data

```sql
-- Create test tenant
INSERT INTO tenants (id, name, domain) VALUES
  ('test-tenant', 'Test Pharmacy', 'test.vardmd.com');

-- Create pharmacist user
INSERT INTO users (id, tenant_id, email, password_hash, first_name, last_name, role) VALUES
  ('pharmacist-1', 'test-tenant', 'pharmacist@test.com', 'hash', 'John', 'Doe', 'PHARMACY');

-- Create admin user
INSERT INTO users (id, tenant_id, email, password_hash, first_name, last_name, role) VALUES
  ('admin-1', 'test-tenant', 'admin@test.com', 'hash', 'Jane', 'Smith', 'PHARMACY_ADMIN');

-- Create test organization (hospital/pharmacy)
INSERT INTO organizations (id, tenant_id, org_type, legal_name, display_name) VALUES
  ('org-1', 'test-tenant', 'PHARMACY', 'Test Pharmacy Ltd', 'Test Pharmacy');

-- Add drugs to formulary
INSERT INTO formulary (id, organization_id, code, name, strength, form, route) VALUES
  ('form-1', 'org-1', 'PAR500', 'Paracetamol', '500mg', 'Tablet', 'Oral'),
  ('form-2', 'org-1', 'AMX500', 'Amoxicillin', '500mg', 'Capsule', 'Oral');

-- Add inventory
INSERT INTO pharmacy_inventory (id, organization_id, formulary_id, batch_no, quantity_on_hand, reorder_level) VALUES
  ('inv-1', 'org-1', 'form-1', 'PAR2025A', 245, 100),
  ('inv-2', 'org-1', 'form-2', 'AMX2025B', 89, 50);
```

---

**END OF DESIGN BRIEF**

---

**Questions?** Contact project lead or refer to:

- DATABASE_SCHEMA.md (created earlier)
- PHARMACY_API_GUIDE.md (comprehensive 1019-line guide)
- Frontend components (PharmacyDashboard.tsx, PharmacyAdminDashboard.tsx)
