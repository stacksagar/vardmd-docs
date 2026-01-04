# VardMD Pharmacy Portal - Functional Requirements Document

**Project:** VardMD Healthcare Platform  
**Module:** Pharmacy Portal  
**Version:** 1.0  
**Date:** January 4, 2026  
**Status:** Draft for Review  
**Document Owner:** Product Team

---

## Document Control

| Version | Date       | Author      | Changes       |
| ------- | ---------- | ----------- | ------------- |
| 1.0     | 2026-01-04 | VardMD Team | Initial draft |

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Stakeholders](#2-stakeholders)
3. [System Overview](#3-system-overview)
4. [Functional Requirements](#4-functional-requirements)
5. [User Stories](#5-user-stories)
6. [Business Rules](#6-business-rules)
7. [Use Cases](#7-use-cases)
8. [Non-Functional Requirements](#8-non-functional-requirements)
9. [Acceptance Criteria](#9-acceptance-criteria)
10. [Constraints & Assumptions](#10-constraints--assumptions)

---

## 1. Introduction

### 1.1 Purpose

This document specifies the functional requirements for the VardMD Pharmacy Portal, a critical component of the VardMD healthcare platform that enables pharmacists to safely process prescriptions, manage medication inventory, and ensure patient safety through automated drug interaction checking.

### 1.2 Scope

**In Scope:**

- Prescription queue management and processing
- Drug interaction and allergy checking
- Medication dispensing workflow
- Inventory management with batch tracking
- Expiry date monitoring and alerts
- Supplier relationship management
- Purchase order creation and receiving
- Financial tracking and reporting
- Audit trail for regulatory compliance

**Out of Scope:**

- Point of Sale (POS) hardware integration
- Insurance claim processing (external system)
- Controlled substance DEA reporting (future phase)
- Patient mobile app integration
- Delivery logistics and tracking

### 1.3 Definitions

| Term       | Definition                                                          |
| ---------- | ------------------------------------------------------------------- |
| Rx         | Prescription (medical abbreviation for "recipe")                    |
| TID        | Ter In Die - Three times daily                                      |
| BID        | Bis In Die - Twice daily                                            |
| QID        | Quater In Die - Four times daily                                    |
| PRN        | Pro Re Nata - As needed                                             |
| Formulary  | Catalog of approved medications available in the pharmacy           |
| Generic    | Non-brand name medication with same active ingredient               |
| Batch      | A specific production lot of medication with unique tracking number |
| Dispensing | The act of providing medication to a patient                        |
| OTC        | Over-The-Counter (no prescription required)                         |
| STAT       | Immediately (urgent priority)                                       |
| PO         | Purchase Order                                                      |
| MRN        | Medical Record Number (unique patient identifier)                   |

### 1.4 References

- VardMD Database Schema Documentation
- VardMD Pharmacy API Guide
- Pharmacy Portal Design Brief
- Nigerian Pharmacy Council Regulations (PCN)
- WHO Guidelines for Safe Medication Practices

---

## 2. Stakeholders

### 2.1 Primary Stakeholders

| Stakeholder             | Role                  | Interest                                           |
| ----------------------- | --------------------- | -------------------------------------------------- |
| Pharmacists             | System Users          | Process prescriptions safely and efficiently       |
| Pharmacy Administrators | System Administrators | Manage inventory, suppliers, and financial reports |
| Patients                | End Beneficiaries     | Receive correct medications safely                 |
| Prescribing Doctors     | Data Providers        | Ensure prescriptions are fulfilled accurately      |
| Hospital Management     | Decision Makers       | Monitor pharmacy performance and profitability     |
| Regulatory Bodies       | Compliance Auditors   | Ensure adherence to pharmacy regulations           |

### 2.2 Secondary Stakeholders

- Pharmacy Technicians
- Suppliers/Vendors
- Insurance Companies
- IT Support Team
- Quality Assurance Team

---

## 3. System Overview

### 3.1 System Context

The Pharmacy Portal is part of the larger VardMD ecosystem and interfaces with:

**Upstream Systems:**

- **Provider Portal** - Receives prescriptions from doctors
- **EMR Dashboard** - Receives medication orders from inpatient settings
- **Patient Portal** - Receives prescription refill requests

**Downstream Systems:**

- **Billing System** - Sends payment transactions
- **Audit System** - Sends compliance logs
- **Notification Service** - Sends alerts to patients and staff

**External Systems:**

- **Drug Interaction Database** - Checks medication safety
- **Supplier Portals** - Places orders and receives confirmations

### 3.2 User Roles

| Role               | Access Level  | Primary Functions                                                                             |
| ------------------ | ------------- | --------------------------------------------------------------------------------------------- |
| **PHARMACY**       | Standard User | Review prescriptions, dispense medications, check inventory, handle walk-in sales             |
| **PHARMACY_ADMIN** | Administrator | All PHARMACY functions + manage formulary, suppliers, purchase orders, view financial reports |

---

## 4. Functional Requirements

### FR-1: Dashboard and Overview

#### FR-1.1: Dashboard Statistics Display

**Priority:** HIGH  
**User Role:** PHARMACY, PHARMACY_ADMIN

**Requirement:**
The system SHALL display real-time pharmacy statistics on the dashboard including:

- Count of pending prescriptions
- Count of prescriptions dispensed today
- Count of low stock inventory items
- Total inventory value in Nigerian Naira (₦)
- Today's revenue
- Number of active alerts

**Acceptance Criteria:**

- Statistics update automatically every 5 minutes
- All counts are tenant-isolated (only show current pharmacy's data)
- Dashboard loads within 2 seconds

#### FR-1.2: Top Moving Drugs Display

**Priority:** MEDIUM  
**User Role:** PHARMACY_ADMIN

**Requirement:**
The system SHALL display the top 5 most dispensed medications in the current month with:

- Drug name
- Quantity dispensed
- Revenue generated
- Current stock level
- Trend indicator (up/down/stable)

#### FR-1.3: Recent Activity Feed

**Priority:** LOW  
**User Role:** PHARMACY, PHARMACY_ADMIN

**Requirement:**
The system SHALL display the last 10 pharmacy activities including:

- Patient name
- Action performed (prescription dispensed, payment received, etc.)
- Timestamp (relative time, e.g., "5 minutes ago")
- Priority indicator for urgent items

---

### FR-2: Prescription Queue Management

#### FR-2.1: Prescription Queue View

**Priority:** CRITICAL  
**User Role:** PHARMACY, PHARMACY_ADMIN

**Requirement:**
The system SHALL display all prescriptions in a queue with the following information:

- Prescription number (Rx-YYYY-NNNNNN format)
- Patient name and MRN
- Prescribing provider name
- Date/time prescribed
- Current status (pending, validated, ready_for_dispense, dispensed, completed, rejected)
- Priority level (routine, urgent, STAT)
- Billing status (pending, covered, partial, paid, overdue)

**Business Rules:**

- STAT prescriptions MUST appear at the top of the queue with red indicator
- Urgent prescriptions MUST appear before routine prescriptions
- Prescriptions older than 72 hours MUST show age indicator
- Queue MUST auto-refresh every 60 seconds

**Acceptance Criteria:**

- User can sort by any column
- User can filter by status, priority, date range
- User can search by patient name, MRN, or Rx number
- Pagination supports 20, 50, 100 items per page

#### FR-2.2: Prescription Status Workflow

**Priority:** CRITICAL  
**User Role:** PHARMACY

**Requirement:**
The system SHALL enforce the following prescription status workflow:

```
pending → validated → ready_for_dispense → dispensed → completed
   ↓
queried (if issues found)
   ↓
rejected (if cannot fulfill)
```

**Status Definitions:**

- **pending**: New prescription awaiting pharmacist review
- **validated**: Pharmacist reviewed, passed safety checks, ready to prepare
- **queried**: Issues found, awaiting clarification from prescriber
- **ready_for_dispense**: Medication prepared, awaiting patient pickup
- **dispensed**: Medication given to patient
- **completed**: All documentation finalized, payment processed
- **rejected**: Cannot fulfill (out of stock, dangerous interaction, etc.)

**Business Rules:**

- Status transitions MUST follow the defined workflow
- Backward transitions (e.g., dispensed → validated) are NOT allowed
- Status changes MUST be logged with timestamp and user ID
- Status "dispensed" MUST trigger automatic inventory deduction

#### FR-2.3: Prescription Detail View

**Priority:** HIGH  
**User Role:** PHARMACY, PHARMACY_ADMIN

**Requirement:**
The system SHALL display complete prescription details including:

**Patient Information:**

- Full name, age, gender
- MRN (Medical Record Number)
- Known allergies (highlighted in red if any)
- Current medications (from dispensing history)
- Insurance information (provider, policy number, coverage %)

**Prescription Information:**

- Prescribed by (doctor name and license number)
- Date/time prescribed
- Consultation ID (link to consultation record)
- Diagnosis/indication

**Medications (for each item):**

- Drug name (brand and generic)
- Strength (e.g., 500mg)
- Dosage form (tablet, capsule, syrup, etc.)
- Dosage (e.g., "1 tablet")
- Frequency (e.g., "TID" - 3 times daily)
- Duration (e.g., "7 days")
- Total quantity
- Unit price and total price
- Special instructions (e.g., "Take with food")

**Acceptance Criteria:**

- All fields display correctly formatted data
- Allergies display in red warning box if present
- Links to related records (patient, consultation) are clickable
- Print button generates printable prescription label

---

### FR-3: Drug Interaction and Safety Checking

#### FR-3.1: Automatic Drug Interaction Check

**Priority:** CRITICAL (SAFETY FEATURE)  
**User Role:** PHARMACY

**Requirement:**
The system SHALL automatically check for drug interactions when a pharmacist reviews a prescription. The check MUST include:

**Drug-Drug Interactions:**

- Compare new medications against patient's current medications
- Identify interactions with severity levels:
  - **Critical**: Life-threatening, absolutely contraindicated
  - **Major**: Serious, requires intervention
  - **Moderate**: Monitor patient closely
  - **Minor**: Generally acceptable

**Drug-Allergy Interactions:**

- Compare medications against patient's documented allergies
- Flag exact matches (e.g., Penicillin allergy + Amoxicillin)
- Flag cross-allergies (e.g., Penicillin allergy + Cephalosporins)

**Dosage Validation:**

- Check if prescribed dose exceeds maximum safe dose
- Check if dose is below therapeutic minimum
- Consider patient age, weight, renal/hepatic function if available

**Duplicate Therapy:**

- Identify if patient is already on same medication
- Identify if patient is on therapeutic duplicate (different drug, same action)

**Business Rules:**

- CRITICAL interactions MUST block dispensing until override authorized
- Override MUST require:
  - Documented reason
  - Contact with prescribing physician
  - Senior pharmacist approval
  - Patient informed consent
- All interactions MUST be logged for audit

**Acceptance Criteria:**

- Check completes within 3 seconds
- Clear, actionable alerts displayed to pharmacist
- Alerts include:
  - Severity level (color-coded)
  - Plain language description
  - Clinical recommendation
  - References to drug monographs
- User can view detailed interaction monograph

#### FR-3.2: Allergy Alert System

**Priority:** CRITICAL  
**User Role:** PHARMACY

**Requirement:**
The system SHALL display prominent allergy warnings when:

- Patient has documented drug allergies
- Prescribed medication matches or cross-reacts with known allergy
- Patient allergy status is unknown/not documented

**Display Requirements:**

- Red banner across prescription detail screen
- Specific allergen highlighted
- Reaction history if documented (e.g., "Anaphylaxis to Penicillin - 2023")
- Recommended alternatives if available

**Business Rules:**

- Allergy alerts CANNOT be dismissed without acknowledgment
- Dispensing with active allergy alert REQUIRES:
  - Override reason documented
  - Prescriber contacted and confirmed
  - Patient counseled on risks
  - Written informed consent

#### FR-3.3: Interaction Alert History

**Priority:** MEDIUM  
**User Role:** PHARMACY_ADMIN

**Requirement:**
The system SHALL maintain a complete log of all drug interaction checks including:

- Date/time of check
- Prescription ID
- Pharmacist who performed check
- Medications checked
- Interactions found (severity, description)
- Actions taken (dispensed, rejected, overridden)
- Override justification if applicable

**Acceptance Criteria:**

- History searchable by date range, pharmacist, severity
- Export to CSV/PDF for regulatory audits
- Minimum retention period: 7 years

---

### FR-4: Medication Dispensing

#### FR-4.1: Dispensing Workflow

**Priority:** CRITICAL  
**User Role:** PHARMACY

**Requirement:**
The system SHALL provide a structured dispensing workflow:

**Step 1: Patient Verification**

- Display patient photo (if available)
- Request patient ID verification
- Confirm patient details match prescription

**Step 2: Safety Check**

- Run drug interaction check
- Review allergy alerts
- Verify dose appropriateness

**Step 3: Medication Preparation**

- Display medications to dispense with quantities
- Show batch number to use (FIFO - First In, First Out)
- Verify expiry date is at least 30 days away
- Generate medication labels

**Step 4: Patient Counseling**

- Display mandatory counseling points:
  - How to take medication (dosage, frequency)
  - When to take (with food, before bed, etc.)
  - Common side effects
  - What to do if dose missed
  - Storage instructions
  - When to follow up with doctor
- Checkbox to confirm counseling provided

**Step 5: Payment Processing**

- Display total amount
- Display insurance coverage (if applicable)
- Calculate patient co-pay
- Record payment method (cash, card, insurance, subscription)
- Generate receipt

**Step 6: Finalization**

- Request patient signature (digital or physical)
- Print prescription label
- Update prescription status to "dispensed"
- Auto-deduct from inventory
- Create dispensing record for audit

**Business Rules:**

- Steps MUST be completed in order
- Patient counseling CANNOT be skipped
- Payment MUST be confirmed before finalization
- System MUST check stock availability before starting

**Acceptance Criteria:**

- Workflow completes in under 5 minutes for routine prescriptions
- All steps logged with timestamps
- Clear error messages if stock insufficient
- Undo option available before finalization

#### FR-4.2: Batch Selection (FIFO)

**Priority:** HIGH  
**User Role:** PHARMACY

**Requirement:**
The system SHALL automatically select medication batches using FIFO (First In, First Out) principle:

- Medications with earliest expiry dates dispensed first
- Batches expiring within 30 days show warning
- Batches expiring within 7 days CANNOT be dispensed
- User can override FIFO if necessary (e.g., customer preference) with documented reason

#### FR-4.3: Partial Dispensing

**Priority:** MEDIUM  
**User Role:** PHARMACY

**Requirement:**
The system SHALL support partial dispensing when full quantity is not available:

- Allow pharmacist to dispense partial quantity
- Create partial dispensing record
- Calculate adjusted payment
- Automatically create backorder for remaining quantity
- Notify patient when backorder fulfilled
- Track partial vs. complete dispensing in reports

**Example:**

- Prescribed: Metformin 500mg x 30 tablets
- Available: 15 tablets
- Action: Dispense 15 now, backorder 15
- Patient pays for 15 tablets
- System creates alert when remaining 15 received

---

### FR-5: Inventory Management

#### FR-5.1: Real-Time Inventory Tracking

**Priority:** CRITICAL  
**User Role:** PHARMACY, PHARMACY_ADMIN

**Requirement:**
The system SHALL track inventory in real-time with the following data:

**For Each Drug:**

- Drug name (brand and generic)
- Drug code (unique identifier)
- Category (antibiotics, pain relief, cardiovascular, etc.)
- Current quantity on hand
- Reorder level (minimum before alert)
- Maximum stock level (prevent overordering)
- Average monthly usage
- Unit of measure (tablets, vials, bottles)

**For Each Batch:**

- Batch number (from manufacturer)
- Quantity in batch
- Expiry date
- Cost price per unit
- Selling price per unit
- Supplier
- Date received
- Storage location (shelf/bin)

**Business Rules:**

- Inventory MUST update automatically when:
  - Medication dispensed (subtract quantity)
  - Stock received (add quantity)
  - Stock adjusted (damaged, stolen, expired)
  - Stock transferred (between locations)
- All changes MUST create audit log entry
- Negative inventory NOT allowed

**Acceptance Criteria:**

- Inventory count accurate to ±1% when audited
- Real-time updates visible within 1 second
- Batch tracking enabled for all medications
- Export to Excel/CSV supported

#### FR-5.2: Low Stock Alerts

**Priority:** HIGH  
**User Role:** PHARMACY_ADMIN

**Requirement:**
The system SHALL automatically generate low stock alerts when:

- Quantity on hand falls below reorder level
- Quantity falls below 5 days of average usage
- Only one batch remaining and it's partially used

**Alert Display:**

- Drug name
- Current quantity
- Reorder level
- Deficit (reorder level - current quantity)
- Suggested reorder quantity
- Preferred supplier
- Estimated delivery time
- Priority (critical if quantity < 2 days usage)

**Alert Actions:**

- Quick action button to create purchase order
- Snooze alert (with reason)
- Mark as ordered (specify PO number)
- Override reorder level if seasonal/temporary

**Notification:**

- Dashboard badge showing count
- Daily email summary to admin at 8:00 AM
- SMS alert for critical items (optional)

#### FR-5.3: Expiry Date Management

**Priority:** HIGH  
**User Role:** PHARMACY, PHARMACY_ADMIN

**Requirement:**
The system SHALL monitor expiry dates and alert when:

- Medication expires within 90 days (warning)
- Medication expires within 30 days (urgent)
- Medication expires within 7 days (critical - cannot dispense)
- Medication is expired (auto-flag for disposal)

**Alert Information:**

- Drug name and batch number
- Current quantity
- Days until expiry
- Total value (cost price × quantity)
- Recommended action:
  - 90 days: Discount pricing to sell faster
  - 30 days: Return to supplier if within policy
  - 7 days: Remove from dispensing inventory
  - Expired: Move to disposal/destruction area

**Business Rules:**

- System MUST prevent dispensing of expired medications
- Expired medications auto-flagged for proper disposal
- Expiry tracking MUST account for:
  - Manufacturer expiry date (printed on package)
  - After opening expiry (e.g., eye drops expire 28 days after opening)
- Monthly expiry report generated automatically

**Acceptance Criteria:**

- Expiry alerts visible on dashboard
- Medications expiring soon highlighted in yellow/red
- Batch selection algorithm avoids near-expiry batches first
- Disposal workflow documented for audit

#### FR-5.4: Stock Adjustment

**Priority:** MEDIUM  
**User Role:** PHARMACY_ADMIN

**Requirement:**
The system SHALL allow manual stock adjustments with proper documentation:

**Adjustment Types:**

- **Damaged goods**: Items damaged in storage/handling
- **Expired**: Items that expired before being sold
- **Stolen/Lost**: Unexplained inventory shortage
- **Returned**: Patient returned unused medication
- **Supplier credit**: Incorrect shipment, returned to supplier
- **Stock count correction**: Physical count differs from system

**Required Information:**

- Drug and batch being adjusted
- Adjustment quantity (+ or -)
- Adjustment type (from list above)
- Reason (free text explanation)
- Supporting documentation (photo upload optional)
- Authorized by (must be admin)

**Business Rules:**

- All adjustments MUST be documented
- Adjustments > 10 units or > ₦10,000 value require approver signature
- Adjustment creates inventory transaction record
- Adjustments affect financial reports (COGS, shrinkage)

**Acceptance Criteria:**

- Adjustment history viewable by drug
- Monthly adjustment report generated
- Excessive adjustments trigger investigation (>5% monthly shrinkage)

#### FR-5.5: Inventory Valuation

**Priority:** MEDIUM  
**User Role:** PHARMACY_ADMIN

**Requirement:**
The system SHALL calculate total inventory value using:

- **Cost Price**: What pharmacy paid supplier
- **Selling Price**: What pharmacy charges patients
- **Potential Profit**: (Selling Price - Cost Price) × Quantity

**Calculations:**

- Total inventory at cost = Σ(Quantity × Cost Price)
- Total inventory at retail = Σ(Quantity × Selling Price)
- Potential gross profit = Total retail - Total cost

**Acceptance Criteria:**

- Values update in real-time
- Breakdown by drug category
- Historical trend graph (monthly comparison)
- Export to Excel for accounting

---

### FR-6: Drug Formulary Management

#### FR-6.1: Formulary (Drug Catalog)

**Priority:** HIGH  
**User Role:** PHARMACY_ADMIN

**Requirement:**
The system SHALL maintain a comprehensive drug formulary with:

**Drug Information:**

- Drug code (unique identifier)
- Brand name
- Generic name (INN - International Nonproprietary Name)
- Active ingredient(s)
- Strength (e.g., 500mg, 10mg/ml)
- Dosage form (tablet, capsule, syrup, injection, cream, etc.)
- Route of administration (oral, IV, topical, etc.)
- Manufacturer
- Drug category/class

**Clinical Information:**

- Therapeutic category
- Indications (what it treats)
- Contraindications (when NOT to use)
- Common side effects
- Storage requirements (temperature, light protection)
- Pregnancy category
- Controlled substance schedule (if applicable)

**Business Information:**

- Standard cost price
- Standard selling price
- Profit margin
- Reimbursement code (for insurance)
- Prescription required (yes/no)
- Substitutable (generic allowed?)
- Active/inactive status

**Business Rules:**

- Each organization has own formulary (multi-tenant)
- Drug code MUST be unique within organization
- Generic equivalents MUST be linked to brand drugs
- Price changes MUST be logged with effective date

**Acceptance Criteria:**

- Add/edit/deactivate drugs (not delete - maintain history)
- Search by any field (name, code, category)
- Bulk import from CSV/Excel
- Export formulary to Excel

#### FR-6.2: Generic Substitution Lookup

**Priority:** HIGH  
**User Role:** PHARMACY

**Requirement:**
The system SHALL provide generic substitution recommendations when:

- Brand-name drug is out of stock
- Generic equivalent is significantly cheaper
- Pharmacist requests alternatives

**Display:**

- Original drug (brand name, strength, price)
- Generic alternatives with:
  - Generic name and strength
  - Manufacturer
  - Price
  - Savings amount and percentage
  - Bioequivalent status (therapeutically equivalent?)
  - Availability in stock

**Business Rules:**

- Substitution requires prescriber approval unless:
  - Prescription marked "substitution allowed"
  - Country regulations allow pharmacist substitution
- Substitution MUST be documented in dispensing record
- Patient MUST be informed of substitution

**Acceptance Criteria:**

- Alternatives ranked by savings (highest first)
- Only show in-stock alternatives
- One-click to call prescriber for approval
- Log prescriber approval in system

---

### FR-7: Supplier and Purchase Order Management

#### FR-7.1: Supplier Management

**Priority:** MEDIUM  
**User Role:** PHARMACY_ADMIN

**Requirement:**
The system SHALL maintain supplier database with:

**Supplier Information:**

- Company name
- Contact person
- Phone, email, website
- Physical address
- Tax ID/Registration number

**Business Terms:**

- Payment terms (Net 30, Net 60, etc.)
- Minimum order value
- Volume discounts
- Average delivery time (in days)
- Return policy

**Performance Tracking:**

- Total orders placed (count and value)
- On-time delivery rate
- Quality issues (count)
- Average response time
- Overall rating (1-5 stars)

**Business Rules:**

- Supplier status: Active, Inactive, Blacklisted
- Inactive suppliers not available for new POs
- Blacklisted suppliers show warning if selected

**Acceptance Criteria:**

- Add/edit/deactivate suppliers
- Search suppliers by name, product supplied
- View order history with supplier
- Export supplier list to Excel

#### FR-7.2: Purchase Order Creation

**Priority:** MEDIUM  
**User Role:** PHARMACY_ADMIN

**Requirement:**
The system SHALL allow creating purchase orders with:

**Header Information:**

- PO number (auto-generated: PO-YYYY-NNNNNN)
- Supplier selected from dropdown
- Order date (auto-filled, today)
- Expected delivery date
- Delivery address (default to pharmacy)
- Payment terms (from supplier defaults)
- Notes/special instructions

**Line Items:**

- Drug (search from formulary)
- Quantity to order
- Unit price (from supplier's last price or manual)
- Total price (auto-calculated)
- Suggested quantity based on:
  - Current stock
  - Reorder level
  - Average monthly usage

**Totals:**

- Subtotal
- VAT (7.5% in Nigeria, configurable)
- Shipping charges (if applicable)
- Discounts (if applicable)
- Grand total

**Business Rules:**

- Minimum order validation against supplier's minimum
- Price mismatch alert if price changed >10% from last order
- Credit limit check (if credit terms)
- Manager approval required for orders >₦100,000

**Acceptance Criteria:**

- Save as draft (not sent to supplier)
- Submit (sends to supplier via email)
- Print/export to PDF
- Track status: Draft, Submitted, Confirmed, Partially Received, Fully Received, Cancelled

#### FR-7.3: Shipment Receiving

**Priority:** MEDIUM  
**User Role:** PHARMACY_ADMIN

**Requirement:**
The system SHALL provide shipment receiving workflow:

**Step 1: Match to Purchase Order**

- Enter PO number or select from list
- Display expected items

**Step 2: Verify Items**
For each item:

- Expected drug and quantity
- Actual drug received (confirm match)
- Actual quantity received
- Batch number (scan or type)
- Expiry date
- Condition (good/damaged)
- Variance (expected - actual)

**Step 3: Discrepancies**
If variance exists:

- Short shipment (less than expected): Create backorder
- Overshipped (more than expected): Accept or reject excess
- Wrong item: Reject and request replacement
- Damaged: Photograph, reject, request replacement

**Step 4: Finalize**

- Update inventory for received items
- Create inventory transactions (type: RECEIPT)
- Update PO status
- Generate goods received note (GRN)
- Trigger payment if terms are immediate

**Business Rules:**

- Partial receiving allowed (receive 50 out of 100 ordered)
- Cannot receive more than ordered without approval
- Expiry date must be at least 6 months away
- Damaged goods NOT added to inventory

**Acceptance Criteria:**

- Mobile-friendly interface (use on warehouse floor)
- Barcode scanning supported (batch number, drug code)
- Photo upload for damaged goods
- Print GRN for record-keeping

---

### FR-8: Walk-In Sales (Over-The-Counter)

#### FR-8.1: OTC Sales Processing

**Priority:** MEDIUM  
**User Role:** PHARMACY

**Requirement:**
The system SHALL support walk-in sales for over-the-counter medications:

**Customer Information (Optional):**

- Name (for receipt)
- Phone (for follow-up if needed)
- Not required for OTC sales

**Sale Items:**

- Search drug from formulary
- Filter to show only OTC items (prescription not required)
- Add multiple items to cart
- Adjust quantities
- Apply discounts (if authorized)

**Pharmacist Counseling:**

- System prompts counseling for certain OTC drugs:
  - Painkillers (dosage limits, liver/kidney warnings)
  - Cough syrup (drowsiness, alcohol content)
  - Antibiotics sold OTC (resistance warnings)
- Checkbox to confirm counseling provided

**Payment:**

- Calculate total
- Apply tax if applicable
- Accept payment (cash, card)
- Generate receipt

**Inventory Update:**

- Auto-deduct from inventory
- Create transaction record (type: OTC_SALE)

**Business Rules:**

- No prescription required for OTC items
- Some OTC items have quantity limits (e.g., max 20 paracetamol tablets)
- Controlled OTC drugs still require ID verification (logged)

**Acceptance Criteria:**

- Quick sale workflow (<2 minutes)
- Supports barcode scanning
- Receipt prints automatically
- Daily OTC sales report available

---

### FR-9: Reporting and Analytics

#### FR-9.1: Dispensing Analytics Report

**Priority:** LOW  
**User Role:** PHARMACY_ADMIN

**Requirement:**
The system SHALL generate dispensing analytics report with:

**Metrics:**

- Total prescriptions dispensed (by day, week, month)
- Average prescriptions per day
- Average items per prescription
- Breakdown by:
  - Drug category
  - Prescribing provider
  - Payment method (cash, insurance, subscription)
  - Priority (routine, urgent, STAT)

**Trends:**

- Daily/weekly/monthly comparison
- Peak hours/days
- Seasonal variations

**Charts:**

- Line chart: Prescriptions over time
- Pie chart: Distribution by category
- Bar chart: Top 10 prescribed drugs

**Acceptance Criteria:**

- Date range selection (last 7 days, last 30 days, custom)
- Export to PDF/Excel
- Schedule automated email delivery (daily/weekly summary)

#### FR-9.2: Inventory Report

**Priority:** MEDIUM  
**User Role:** PHARMACY_ADMIN

**Requirement:**
The system SHALL generate inventory reports:

**Stock Levels Report:**

- All drugs with current quantities
- Value at cost and retail
- Highlight low stock and overstocked items
- Days supply remaining (based on average usage)

**Movement Report:**

- For selected date range, show:
  - Opening stock
  - Receipts (purchases)
  - Dispensed/sold
  - Adjustments
  - Closing stock
- For each drug or aggregated

**Slow-Moving Items:**

- Drugs with <5 units sold in last 90 days
- Total value tied up
- Recommendation to discount or return

**Fast-Moving Items:**

- Drugs with >50 units sold in last 30 days
- Stockout risk
- Recommendation to increase stock levels

**Acceptance Criteria:**

- Filter by category, supplier, date range
- Export to Excel
- Print-friendly format

#### FR-9.3: Financial Performance Report

**Priority:** MEDIUM  
**User Role:** PHARMACY_ADMIN

**Requirement:**
The system SHALL generate financial reports:

**Revenue Report:**

- Total sales (prescriptions + OTC)
- Breakdown by:
  - Payment method
  - Drug category
  - Insurance vs. cash
- Average transaction value

**Cost of Goods Sold (COGS):**

- Cost of medications dispensed
- Gross profit (Revenue - COGS)
- Gross profit margin %

**Inventory Financials:**

- Inventory value (cost)
- Inventory turnover ratio
- Days inventory on hand
- Shrinkage (unexplained loss)

**Acceptance Criteria:**

- Monthly and year-to-date views
- Comparison to previous period
- Export to Excel for accounting software

#### FR-9.4: Supplier Performance Report

**Priority:** LOW  
**User Role:** PHARMACY_ADMIN

**Requirement:**
The system SHALL track supplier performance:

**Metrics per Supplier:**

- Number of orders placed
- Total value of orders
- On-time delivery rate (%)
- Average delivery time (days)
- Quality issues (count)
- Return rate (%)

**Ranking:**

- Suppliers ranked by:
  - Reliability (on-time delivery)
  - Cost-effectiveness (best prices)
  - Quality (fewest issues)

**Acceptance Criteria:**

- Compare multiple suppliers
- Historical trend (quarterly view)
- Identify underperforming suppliers

#### FR-9.5: Audit and Compliance Reports

**Priority:** HIGH  
**User Role:** PHARMACY_ADMIN

**Requirement:**
The system SHALL generate regulatory compliance reports:

**Dispensing Audit Trail:**

- All medications dispensed with:
  - Patient name and MRN
  - Drug name, strength, quantity
  - Batch number
  - Dispensed by (pharmacist name and license)
  - Date/time
  - Prescriber
- Filterable and exportable for regulatory inspection

**Drug Interaction Overrides:**

- All cases where interaction alert was overridden
- Justification documented
- Approver name
- Patient outcome (if known)

**Controlled Substances Log:**

- All controlled drug transactions
- Includes:
  - Receipts (from supplier)
  - Dispensing (to patients)
  - Adjustments/losses
- Running balance

**Business Rules:**

- Audit logs MUST be tamper-proof (no deletion)
- Minimum retention: 7 years
- Exportable in standard formats (CSV, PDF)

---

## 5. User Stories

### Epic 1: Prescription Management

**US-1.1: View Pending Prescriptions**  
**As a** pharmacist  
**I want to** view all pending prescriptions in a queue  
**So that** I can prioritize and process them efficiently

**Acceptance Criteria:**

- Given I am logged in as a pharmacist
- When I navigate to the Rx Queue
- Then I see all pending prescriptions sorted by priority (STAT > Urgent > Routine)
- And I can see patient name, Rx number, prescribing doctor, and time prescribed
- And STAT prescriptions are highlighted in red

---

**US-1.2: Review Prescription for Safety**  
**As a** pharmacist  
**I want to** check for drug interactions and allergies before dispensing  
**So that** I can ensure patient safety

**Acceptance Criteria:**

- Given I select a prescription to review
- When I click "Review"
- Then the system automatically checks for:
  - Drug-drug interactions
  - Drug-allergy interactions
  - Duplicate therapy
  - Dosage appropriateness
- And if a critical interaction is found
- Then I see a red alert with description and recommendation
- And I cannot dispense without documenting override reason

---

**US-1.3: Dispense Medication to Patient**  
**As a** pharmacist  
**I want to** dispense medications and record the transaction  
**So that** inventory is updated and patient receives correct medications

**Acceptance Criteria:**

- Given a prescription is validated and ready
- When patient arrives at pharmacy
- Then I verify patient identity
- And I prepare the correct medications with appropriate batch numbers
- And I counsel the patient on usage, side effects, and storage
- And patient confirms understanding and signs
- And I process payment
- Then prescription status updates to "dispensed"
- And inventory automatically decreases
- And dispensing record is created for audit

---

**US-1.4: Handle Generic Substitution**  
**As a** pharmacist  
**I want to** substitute a brand-name drug with a generic equivalent  
**So that** I can fulfill prescriptions when brand is out of stock or save patient money

**Acceptance Criteria:**

- Given a prescription for a brand-name drug
- When I search for alternatives
- Then I see a list of generic equivalents with:
  - Name, strength, price, savings
  - Current stock availability
  - Bioequivalence status
- And I can select a generic
- Then I call prescriber for approval (or note if substitution allowed)
- And I document the substitution reason
- And I inform patient of the change

---

### Epic 2: Inventory Management

**US-2.1: Monitor Low Stock Items**  
**As a** pharmacy administrator  
**I want to** receive alerts when medications are running low  
**So that** I can reorder before running out

**Acceptance Criteria:**

- Given inventory quantities are updated in real-time
- When a drug's quantity falls below reorder level
- Then I see an alert in the dashboard
- And the alert shows:
  - Drug name
  - Current quantity
  - Reorder level
  - Suggested order quantity
  - Preferred supplier
- And I can directly create a purchase order from the alert

---

**US-2.2: Track Expiring Medications**  
**As a** pharmacist  
**I want to** see medications that are expiring soon  
**So that** I can take action (sell, return, or dispose)

**Acceptance Criteria:**

- Given medications have expiry dates
- When I view expiring medications
- Then I see medications expiring within:
  - 7 days (critical - cannot dispense)
  - 30 days (urgent - return to supplier if possible)
  - 90 days (warning - discount pricing)
- And I see quantity and total value
- And system prevents dispensing of expired medications

---

**US-2.3: Receive and Add Stock**  
**As a** pharmacy administrator  
**I want to** receive shipments and update inventory  
**So that** stock levels are accurate

**Acceptance Criteria:**

- Given a purchase order was placed
- When shipment arrives
- Then I select the PO
- And I verify each item received:
  - Confirm drug matches
  - Enter actual quantity
  - Scan/enter batch number
  - Enter expiry date
  - Note any damaged items
- And inventory updates automatically
- And low stock alerts clear if quantity now adequate

---

### Epic 3: Financial and Reporting

**US-3.1: View Daily Sales Summary**  
**As a** pharmacy administrator  
**I want to** view daily sales and revenue  
**So that** I can monitor business performance

**Acceptance Criteria:**

- Given I view the dashboard
- When I select today's date
- Then I see:
  - Total prescriptions dispensed
  - Total OTC sales
  - Total revenue
  - Breakdown by payment method
  - Top 5 selling drugs
- And data updates in real-time

---

**US-3.2: Generate Monthly Financial Report**  
**As a** pharmacy administrator  
**I want to** generate a monthly financial report  
**So that** I can analyze profitability and costs

**Acceptance Criteria:**

- Given I select "Financial Report" for a month
- When I generate the report
- Then I see:
  - Total revenue
  - Cost of goods sold
  - Gross profit and margin %
  - Inventory value
  - Top revenue-generating drugs
  - Comparison to previous month
- And I can export to Excel/PDF

---

## 6. Business Rules

### BR-1: Prescription Processing Rules

**BR-1.1:** A prescription MUST have a valid prescriber (doctor) ID and license number.

**BR-1.2:** A prescription cannot be dispensed more than 365 days after being written (expired prescription).

**BR-1.3:** STAT prescriptions MUST be processed within 30 minutes of receipt.

**BR-1.4:** Controlled substances require:

- Valid prescription (no refills allowed)
- Patient ID verification
- Pharmacist license verification
- Electronic logging to regulatory authority

**BR-1.5:** Prescription refills:

- Original prescription must specify number of refills allowed
- Refill cannot exceed original quantities
- Refills must be within validity period (usually 1 year)

**BR-1.6:** Prescription modifications:

- Only prescriber can modify prescription
- Pharmacist can suggest changes but cannot alter prescription
- Exception: Generic substitution allowed per country regulations

---

### BR-2: Inventory Rules

**BR-2.1:** Inventory cannot go negative (system must prevent over-dispensing).

**BR-2.2:** Batch selection follows FIFO (First In, First Out) to minimize expiry waste.

**BR-2.3:** Medications expiring within 7 days MUST NOT be dispensed to patients.

**BR-2.4:** Expired medications MUST be segregated and disposed of according to regulations.

**BR-2.5:** Stock adjustments >10 units or >₦10,000 require admin approval.

**BR-2.6:** Physical stock count MUST be conducted quarterly and reconciled with system.

**BR-2.7:** Reorder level calculation:

```
Reorder Level = (Average Daily Usage × Lead Time Days) + Safety Stock
```

---

### BR-3: Pricing Rules

**BR-3.1:** Selling price MUST be at least cost price (no selling at a loss without approval).

**BR-3.2:** Price changes take effect immediately for new sales but do not affect pending prescriptions.

**BR-3.3:** Discount limitations:

- Standard discount: Up to 10% (pharmacist authorized)
- High discount: 10-25% (admin approval required)
- Above 25%: Owner approval + documented reason

**BR-3.4:** Insurance billing:

- Patient pays co-pay at point of sale
- Pharmacy submits claim to insurance
- Outstanding balance tracked in accounts receivable

---

### BR-4: Safety and Compliance Rules

**BR-4.1:** Critical drug interactions MUST be reviewed by a licensed pharmacist (cannot be delegated to technicians).

**BR-4.2:** Patient counseling is MANDATORY before dispensing (cannot be skipped).

**BR-4.3:** All dispensing activities MUST be logged with:

- Patient identifier
- Medication details
- Batch number
- Pharmacist identifier
- Date/time

**BR-4.4:** Audit logs MUST be immutable (no deletion, modification allowed only with logged reason).

**BR-4.5:** Data retention requirements:

- Prescription records: 7 years
- Dispensing records: 7 years
- Controlled substance logs: 10 years
- Financial records: As per tax law (typically 7 years)

**BR-4.6:** Pharmacist licensing:

- Only licensed pharmacists can dispense prescription medications
- License verification required during onboarding
- License expiry tracked, alerts sent 90 days before expiry

---

### BR-5: Multi-Tenant Rules

**BR-5.1:** All data MUST be isolated by tenantId (each pharmacy sees only their data).

**BR-5.2:** Cross-tenant data access is STRICTLY PROHIBITED except for:

- Super admin for support purposes (with audit log)
- Aggregate anonymous statistics

**BR-5.3:** User can only belong to one tenant at a time.

**BR-5.4:** Tenant cannot be deleted if active prescriptions or inventory exist (must be archived).

---

## 7. Use Cases

### UC-1: Process Urgent Prescription

**Primary Actor:** Pharmacist  
**Goal:** Safely dispense urgent prescription to patient  
**Preconditions:**

- Pharmacist is logged in
- Prescription exists in system with "urgent" priority

**Main Success Scenario:**

1. Pharmacist sees urgent prescription notification in queue
2. Pharmacist opens prescription details
3. System displays patient info, medications, allergies
4. Pharmacist clicks "Review"
5. System runs drug interaction check
6. System shows "No critical interactions found"
7. Pharmacist verifies stock availability
8. Pharmacist clicks "Prepare"
9. System generates medication label with batch number
10. Patient arrives at pharmacy
11. Pharmacist verifies patient ID
12. Pharmacist counsels patient on:
    - Dosage and frequency
    - Side effects
    - Storage instructions
13. Patient signs acknowledgment
14. Pharmacist processes payment
15. System updates:
    - Prescription status → "dispensed"
    - Inventory reduced
    - Dispensing record created
16. System prints receipt
17. Patient leaves with medication

**Alternative Flows:**

**3a. Critical Drug Interaction Found**

1. System displays red alert: "Aspirin + Warfarin = Bleeding risk"
2. Pharmacist calls prescribing doctor
3. Doctor changes prescription to Paracetamol
4. New prescription enters queue
5. Resume at step 4

**7a. Drug Out of Stock**

1. System shows "Insufficient stock: 0 available, 21 needed"
2. Pharmacist searches for generic alternatives
3. System shows alternatives with pricing
4. Pharmacist calls doctor for substitution approval
5. Doctor approves
6. Pharmacist prepares generic drug instead
7. Resume at step 9

**10a. Patient Cannot Pay**

1. Pharmacist checks insurance coverage
2. System shows 70% coverage, patient owes 30%
3. Patient pays 30% co-pay
4. System records insurance claim
5. Resume at step 14

**Postconditions:**

- Prescription status is "dispensed"
- Inventory updated
- Payment recorded
- Audit trail created

---

### UC-2: Receive Supplier Shipment

**Primary Actor:** Pharmacy Administrator  
**Goal:** Receive and verify supplier shipment, update inventory  
**Preconditions:**

- Purchase order was created and sent to supplier
- Shipment has arrived at pharmacy

**Main Success Scenario:**

1. Admin navigates to "Purchase Orders"
2. Admin selects PO matching delivery note
3. System displays expected items
4. For each item, admin:
   - Scans barcode or enters drug code
   - System confirms match to PO
   - Admin enters actual quantity received
   - Admin scans/enters batch number
   - Admin enters expiry date
5. System calculates:
   - Total items received: 100
   - Total items expected: 100
   - Variance: 0
6. Admin clicks "Finalize Receipt"
7. System:
   - Updates inventory (+100 units per item)
   - Creates inventory transactions (type: RECEIPT)
   - Updates PO status to "Fully Received"
   - Clears low stock alerts if quantity now adequate
8. System generates Goods Received Note (GRN)
9. Admin prints GRN for record-keeping
10. Admin signs delivery note for courier

**Alternative Flows:**

**4a. Quantity Discrepancy**

1. Expected: 100 units
2. Received: 85 units
3. System flags 15 unit shortage
4. Admin documents reason: "Short shipped by supplier"
5. System creates backorder for 15 units
6. System notifies admin to contact supplier
7. Resume at step 5

**4b. Wrong Item Received**

1. Expected: Paracetamol 500mg
2. Received: Paracetamol 1000mg (different strength)
3. Admin marks item as "Rejected"
4. Admin photographs incorrect item
5. System notifies supplier of error
6. System creates replacement order
7. Resume at step 4 for next item

**4c. Damaged Goods**

1. Admin inspects package, finds 20 damaged bottles
2. Admin marks 20 units as "Damaged"
3. Admin uploads photos of damage
4. System adjusts received quantity (100 - 20 = 80)
5. System creates supplier claim for damaged goods
6. Resume at step 5

**4d. Near-Expiry Received**

1. Admin enters expiry date: March 15, 2026 (2 months away)
2. System alerts: "Expiry within 6 months - Reject?"
3. Admin contacts supplier
4. Supplier agrees to replace with longer-dated stock
5. Admin rejects shipment
6. Resume at step 4 for next item

**Postconditions:**

- Inventory updated with received quantities
- Purchase order status updated
- GRN created for accounting
- Alerts cleared or backorders created

---

### UC-3: Override Critical Drug Interaction

**Primary Actor:** Senior Pharmacist  
**Goal:** Dispense medication despite critical interaction warning  
**Preconditions:**

- Prescription exists in system
- Pharmacist has reviewed prescription
- Critical drug interaction detected

**Main Success Scenario:**

1. Pharmacist reviews prescription for Mrs. Kemi
2. System detects: Patient currently on Warfarin
3. New prescription includes Aspirin
4. System displays CRITICAL ALERT:
   - "Warfarin + Aspirin = Severe bleeding risk"
   - "Combined blood thinning may cause internal bleeding"
   - "CONTRAINDICATED - Do not dispense"
5. Pharmacist calls prescribing doctor (Dr. Okafor)
6. Dr. Okafor reviews patient file
7. Dr. Okafor explains: "Patient has specific condition requiring both drugs"
8. Dr. Okafor agrees to monitor patient closely
9. Dr. Okafor provides authorization code
10. Pharmacist clicks "Override Alert"
11. System requires:
    - Override reason
    - Prescriber approval (authorization code)
    - Senior pharmacist approval
12. Pharmacist enters:
    - Reason: "Prescriber approved dual therapy for specific indication"
    - Auth code: "DOK-2026-4567"
    - Senior pharmacist ID: "Senior Pharm. Tunde"
13. Senior Pharmacist reviews and approves
14. System logs override with all details
15. Pharmacist counsels patient extensively on:
    - Bleeding risks
    - Warning signs (unusual bruising, blood in stool/urine)
    - When to seek emergency care
16. Patient signs enhanced consent form
17. System allows dispensing
18. Pharmacist dispenses both medications
19. System creates follow-up reminder (7 days)

**Alternative Flows:**

**8a. Doctor Disagrees with Dual Therapy**

1. Dr. Okafor reviews and says: "No, this is an error. Stop the Aspirin."
2. Dr. Okafor cancels Aspirin prescription
3. Dr. Okafor prescribes Paracetamol instead
4. New prescription enters queue
5. Use case ends

**13a. Senior Pharmacist Denies Override**

1. Senior Pharmacist reviews interaction
2. Senior Pharmacist determines risk > benefit
3. Senior Pharmacist denies override
4. Pharmacist must contact doctor again
5. Resume at step 5

**Postconditions:**

- Medication dispensed with documented override
- Detailed audit log created
- Patient counseled and consented
- Follow-up scheduled

---

## 8. Non-Functional Requirements

### NFR-1: Performance

**NFR-1.1:** Dashboard and prescription queue MUST load within 2 seconds under normal load.

**NFR-1.2:** Drug interaction check MUST complete within 3 seconds.

**NFR-1.3:** System MUST support 50 concurrent users (pharmacists + admins) without degradation.

**NFR-1.4:** Real-time inventory updates MUST be visible within 1 second of transaction.

**NFR-1.5:** System MUST handle 1000 prescriptions per day per pharmacy.

---

### NFR-2: Availability

**NFR-2.1:** System uptime MUST be 99.5% (allowing 3.6 hours downtime/month).

**NFR-2.2:** Planned maintenance MUST be scheduled during off-peak hours (12 AM - 6 AM).

**NFR-2.3:** System MUST have automated backups every 6 hours.

**NFR-2.4:** Disaster recovery RTO (Recovery Time Objective) MUST be < 4 hours.

**NFR-2.5:** Disaster recovery RPO (Recovery Point Objective) MUST be < 1 hour (maximum 1 hour of data loss).

---

### NFR-3: Security

**NFR-3.1:** All API endpoints MUST require authentication (JWT token).

**NFR-3.2:** Passwords MUST be hashed using bcrypt with minimum 10 rounds.

**NFR-3.3:** Session timeout MUST be 8 hours for pharmacist, 12 hours for admin.

**NFR-3.4:** All sensitive data (patient info, prescriptions) MUST be encrypted at rest (AES-256).

**NFR-3.5:** Data in transit MUST use HTTPS/TLS 1.2 or higher.

**NFR-3.6:** Role-based access control (RBAC) MUST be enforced at API and UI levels.

**NFR-3.7:** Audit logs MUST be tamper-proof (append-only, no deletion).

**NFR-3.8:** Failed login attempts > 5 MUST lock account for 15 minutes.

**NFR-3.9:** Pharmacist license verification MUST occur before granting dispensing permissions.

---

### NFR-4: Usability

**NFR-4.1:** System MUST be usable on desktop, tablet, and mobile devices (responsive design).

**NFR-4.2:** Critical actions (dispense, reject) MUST have confirmation dialogs.

**NFR-4.3:** Error messages MUST be clear and actionable (not technical jargon).

**NFR-4.4:** System MUST support keyboard shortcuts for frequent actions.

**NFR-4.5:** User interface MUST follow VardMD design system (shadcn/ui components).

**NFR-4.6:** System MUST support dark mode (already implemented in frontend).

**NFR-4.7:** Accessibility MUST meet WCAG 2.1 Level AA standards:

- Screen reader compatible
- Keyboard navigable
- Adequate color contrast
- Text resize support

**NFR-4.8:** New user onboarding MUST include guided tutorial for key workflows.

---

### NFR-5: Data Integrity

**NFR-5.1:** Database transactions MUST be ACID-compliant (Atomicity, Consistency, Isolation, Durability).

**NFR-5.2:** Inventory updates MUST use database transactions to prevent race conditions.

**NFR-5.3:** Concurrent prescription processing MUST use optimistic or pessimistic locking.

**NFR-5.4:** Data validation MUST occur at:

- Frontend (immediate user feedback)
- API layer (prevent malicious input)
- Database constraints (final safeguard)

**NFR-5.5:** Cascading deletes MUST be carefully controlled:

- Tenant deletion → cascade to all related data
- User deletion → reassign prescriptions, not delete history
- Drug deletion → soft delete (mark inactive, keep history)

---

### NFR-6: Scalability

**NFR-6.1:** System architecture MUST support horizontal scaling (add more servers).

**NFR-6.2:** Database MUST support read replicas for reporting queries.

**NFR-6.3:** System MUST support 100+ tenants (pharmacies) without redesign.

**NFR-6.4:** Caching strategy MUST be implemented for frequently accessed data (formulary, suppliers).

---

### NFR-7: Maintainability

**NFR-7.1:** Code MUST follow TypeScript best practices (strict mode, type safety).

**NFR-7.2:** API MUST be documented using OpenAPI/Swagger.

**NFR-7.3:** Unit test coverage MUST be >70% for backend services.

**NFR-7.4:** Integration tests MUST cover critical workflows (prescription processing, inventory updates).

**NFR-7.5:** Code MUST be reviewed before merging (pull request approval required).

**NFR-7.6:** Database migrations MUST be versioned and reversible.

---

### NFR-8: Compliance

**NFR-8.1:** System MUST comply with Nigerian Pharmacy Council (PCN) regulations.

**NFR-8.2:** System MUST support regulatory audits:

- Generate audit reports
- Export data in standard formats
- Provide read-only access for auditors

**NFR-8.3:** Patient data MUST comply with data protection regulations:

- Consent management
- Right to access (patient can request their data)
- Right to erasure (with regulatory exceptions)

**NFR-8.4:** Controlled substance tracking MUST meet government requirements:

- Electronic logging
- Reporting to authorities
- Secure storage of records

---

## 9. Acceptance Criteria

### AC-1: Prescription Processing

**AC-1.1: Prescription Queue Display**

- [ ] Queue displays all pending prescriptions for current tenant only
- [ ] STAT prescriptions appear at top with red badge
- [ ] Urgent prescriptions appear before routine
- [ ] Queue auto-refreshes every 60 seconds
- [ ] User can filter by status, priority, date range
- [ ] User can search by patient name, MRN, Rx number

**AC-1.2: Drug Interaction Check**

- [ ] Check runs automatically when reviewing prescription
- [ ] Check completes within 3 seconds
- [ ] Critical interactions block dispensing
- [ ] Interactions display severity (critical, major, moderate, minor)
- [ ] User can view detailed interaction monograph
- [ ] Allergy alerts display in red banner
- [ ] Override requires documented reason + approval

**AC-1.3: Dispensing Workflow**

- [ ] Patient verification required before dispensing
- [ ] Counseling checklist must be completed
- [ ] Payment must be processed before finalization
- [ ] Inventory automatically decreases upon dispensing
- [ ] Dispensing record created with batch number
- [ ] Receipt prints automatically
- [ ] Prescription status updates to "dispensed"

---

### AC-2: Inventory Management

**AC-2.1: Real-Time Inventory**

- [ ] Inventory updates within 1 second of transaction
- [ ] Negative inventory prevented (validation)
- [ ] Batch tracking enabled for all medications
- [ ] FIFO batch selection implemented
- [ ] Expiry dates enforced (7-day cutoff)

**AC-2.2: Stock Alerts**

- [ ] Low stock alert triggers when quantity < reorder level
- [ ] Expiring medications alert at 90, 30, 7 days
- [ ] Alerts visible on dashboard with count badge
- [ ] User can click alert to view details
- [ ] One-click purchase order creation from alert

**AC-2.3: Stock Receiving**

- [ ] PO selection displays expected items
- [ ] Variance calculation (expected - actual)
- [ ] Damaged goods can be flagged with photo
- [ ] Inventory updates automatically upon finalization
- [ ] GRN generated for record-keeping

---

### AC-3: Reporting

**AC-3.1: Dispensing Report**

- [ ] Report displays for selected date range
- [ ] Metrics include total prescriptions, revenue, top drugs
- [ ] Charts display trends over time
- [ ] Export to PDF and Excel supported

**AC-3.2: Financial Report**

- [ ] Revenue, COGS, gross profit calculated correctly
- [ ] Breakdown by payment method available
- [ ] Comparison to previous period shown
- [ ] Export to Excel for accounting

**AC-3.3: Audit Reports**

- [ ] All dispensing activities logged
- [ ] Logs include patient, drug, pharmacist, date/time
- [ ] Logs are immutable (no deletion)
- [ ] Export to CSV for regulatory inspection

---

### AC-4: Security and Access Control

**AC-4.1: Authentication**

- [ ] Login requires valid email and password
- [ ] JWT token expires after 8 hours
- [ ] Failed login (5 attempts) locks account for 15 minutes
- [ ] Password reset via email supported

**AC-4.2: Authorization**

- [ ] PHARMACY role can process prescriptions
- [ ] PHARMACY_ADMIN role has all PHARMACY permissions + admin features
- [ ] Non-admin users cannot access supplier management
- [ ] Non-admin users cannot access financial reports

**AC-4.3: Tenant Isolation**

- [ ] All queries filter by tenantId
- [ ] User cannot access other tenant's data
- [ ] Cross-tenant access blocked (except super admin)

---

## 10. Constraints & Assumptions

### Constraints

**Technical Constraints:**

- Backend MUST use Node.js + TypeScript + Express.js
- Frontend MUST use React + TypeScript
- Database MUST be PostgreSQL
- ORM MUST be Prisma
- Authentication MUST use JWT tokens
- UI library MUST be shadcn/ui (consistency with existing VardMD components)

**Business Constraints:**

- Budget: Development cost MUST be within allocated budget
- Timeline: Phase 1 (core features) MUST be completed in 4 weeks
- Compliance: MUST meet Nigerian Pharmacy Council regulations
- Multi-tenancy: MUST support isolation for multiple pharmacies

**Resource Constraints:**

- Development team: 2 developers (Hariss and Mohammed)
- Testing: QA team available for 1 week
- Deployment: Existing VardMD infrastructure (DigitalOcean)

**Legal Constraints:**

- Data protection: MUST comply with Nigeria Data Protection Act
- Pharmacy regulations: MUST maintain records for 7+ years
- Controlled substances: MUST report to regulatory authorities

---

### Assumptions

**User Assumptions:**

- Pharmacists have basic computer literacy
- Pharmacists have smartphones for 2FA (if implemented)
- Pharmacists understand medical terminology (TID, BID, etc.)
- Internet connection is stable during operating hours

**Business Assumptions:**

- Drug interaction database is available and maintained by third party
- Suppliers can accept purchase orders via email (no API integration initially)
- Insurance claims processed offline (not part of this system initially)
- Pharmacies operate during regular business hours (8 AM - 8 PM)

**Technical Assumptions:**

- Existing VardMD authentication system can be extended for pharmacy roles
- Provider Portal is operational and sending prescriptions
- Database can handle projected transaction volume (1000 prescriptions/day)
- Barcode scanners are available for batch number scanning

**Data Assumptions:**

- Formulary will be pre-populated with common medications
- Patient allergy data is accurate and up-to-date
- Prescribing doctors provide complete prescription information
- Batch numbers from suppliers are unique and traceable

---

## Appendices

### Appendix A: Glossary

| Term          | Definition                                                                              |
| ------------- | --------------------------------------------------------------------------------------- |
| Bioequivalent | Generic drug that delivers same active ingredient in same amount and rate as brand drug |
| Co-pay        | Portion of cost patient pays (remainder covered by insurance)                           |
| FIFO          | First In, First Out - oldest stock used first                                           |
| Formulary     | List of approved medications                                                            |
| GRN           | Goods Received Note - receipt for shipment                                              |
| INN           | International Nonproprietary Name (generic drug name)                                   |
| Monograph     | Detailed drug information document                                                      |
| PO            | Purchase Order                                                                          |
| PRN           | As needed (not on fixed schedule)                                                       |

### Appendix B: Acronyms

| Acronym | Full Form                         |
| ------- | --------------------------------- |
| API     | Application Programming Interface |
| COGS    | Cost of Goods Sold                |
| JWT     | JSON Web Token                    |
| MRN     | Medical Record Number             |
| OTC     | Over-The-Counter                  |
| PCN     | Pharmacy Council of Nigeria       |
| RBAC    | Role-Based Access Control         |
| RTO     | Recovery Time Objective           |
| RPO     | Recovery Point Objective          |
| STAT    | Immediately (from Latin "statim") |

---

**END OF FUNCTIONAL REQUIREMENTS DOCUMENT**

---

**Prepared by:** VardMD Product Team  
**Review Date:** January 4, 2026  
**Next Review:** February 4, 2026  
**Approval Required:** Product Owner, Technical Lead, Pharmacy Stakeholder

**Status:** ✅ Ready for Development
