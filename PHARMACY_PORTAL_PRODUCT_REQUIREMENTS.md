# VardMD Pharmacy Portal - Product Requirements Document (PRD)

**Product:** VardMD Pharmacy Portal  
**Version:** 1.0  
**Date:** January 4, 2026  
**Status:** Draft for Approval  
**Owner:** Product Team

---

## Executive Summary

### Vision

Build a pharmacy management system that eliminates medication errors, prevents stockouts, and ensures regulatory compliance while providing pharmacists with an intuitive, efficient workflow.

### Problem Statement

**Current State:**

- Paper prescriptions are illegible, easily lost, and prone to fraud
- No automated drug interaction checking leads to preventable adverse events
- Manual inventory tracking causes frequent stockouts of critical medications
- No real-time visibility into pharmacy operations
- Compliance documentation is manual and time-consuming

**Impact:**

- 15-20% of prescriptions delayed due to illegible handwriting
- 5-10% medication errors due to missed drug interactions
- 30% increase in inventory costs from overstocking/stockouts
- 10+ hours/week spent on manual record-keeping

### Success Metrics

| Metric                       | Baseline | Target (6 months) |
| ---------------------------- | -------- | ----------------- |
| Prescription processing time | 15 min   | 5 min             |
| Medication errors            | 5-10%    | <1%               |
| Stockout incidents           | 15/month | <3/month          |
| Inventory accuracy           | 85%      | 98%               |
| Pharmacist satisfaction      | N/A      | >4.5/5.0          |
| Regulatory audit preparation | 40 hours | 2 hours           |

---

## Product Objectives

### Primary Objectives (Must Have)

1. **Patient Safety** - Prevent medication errors through automated interaction checking
2. **Operational Efficiency** - Reduce prescription processing time by 66%
3. **Inventory Optimization** - Maintain 98%+ inventory accuracy, minimize stockouts
4. **Regulatory Compliance** - Automated audit trails, 7-year data retention

### Secondary Objectives (Should Have)

5. **Financial Visibility** - Real-time profitability tracking
6. **Supplier Management** - Streamlined procurement workflow
7. **Analytics & Insights** - Data-driven decision making

---

## Target Users

### Primary Users

#### 1. Pharmacist (PHARMACY Role)

**Profile:**

- Age: 25-45 years
- Education: Bachelor of Pharmacy (B.Pharm) or higher
- License: Registered with Pharmacy Council of Nigeria (PCN)
- Tech savvy: Moderate (comfortable with smartphones, computers)
- Work environment: Hospital pharmacy, retail pharmacy
- Daily workflow: Process 30-50 prescriptions, counsel patients, manage inventory

**Pain Points:**

- ❌ Illegible handwritten prescriptions waste time
- ❌ Fear of dispensing wrong medication (liability)
- ❌ Manually checking drug interactions is error-prone
- ❌ Patients wait too long for medications
- ❌ Inventory checks interrupt workflow
- ❌ Paperwork takes time away from patient care

**Goals:**

- ✅ Dispense medications safely and accurately
- ✅ Complete clinical checks quickly
- ✅ Provide excellent patient counseling
- ✅ Maintain accurate inventory
- ✅ End shift on time (no overtime completing paperwork)

**User Scenario:**

> "As a pharmacist, I receive a prescription for Amoxicillin from Dr. Okafor. I need to verify the patient has no penicillin allergy, check for drug interactions with their current Warfarin medication, confirm we have stock, counsel the patient on dosage and side effects, process payment, and update inventory - all within 5 minutes so the next patient doesn't wait too long."

#### 2. Pharmacy Administrator (PHARMACY_ADMIN Role)

**Profile:**

- Age: 30-55 years
- Education: B.Pharm + MBA or management certification
- Experience: 5+ years in pharmacy operations
- Responsibilities: Inventory, finances, staff, suppliers, reports
- Works in: Office + pharmacy floor

**Pain Points:**

- ❌ No visibility into real-time inventory levels
- ❌ Expiring medications cause financial losses
- ❌ Supplier management is chaotic (emails, phone calls, sticky notes)
- ❌ Monthly reports take days to compile manually
- ❌ Cannot identify top-selling vs. slow-moving drugs
- ❌ Regulatory audits are stressful and time-consuming

**Goals:**

- ✅ Optimize inventory (minimize waste, prevent stockouts)
- ✅ Maximize profitability (track margins, reduce costs)
- ✅ Streamline supplier relationships
- ✅ Generate reports instantly for management
- ✅ Ensure regulatory compliance effortlessly

**User Scenario:**

> "As an admin, I need to review yesterday's sales, identify which 5 drugs are low on stock, create purchase orders for those drugs, receive a shipment that arrived today, verify the expiry dates are acceptable, update inventory, and generate a financial report for the hospital director - all before the morning meeting at 10 AM."

### Secondary Users

- **Pharmacy Technicians** - Support pharmacists with inventory tasks
- **Hospital Management** - Review performance reports
- **Regulatory Auditors** - Access compliance documentation
- **Patients** - Indirect beneficiaries (faster service, fewer errors)

---

## User Stories & Prioritization

### Epic 1: Prescription Processing (Priority: CRITICAL)

**US-1 (P0): Process Prescription with Safety Check**

```
AS A pharmacist
I WANT TO see all prescription details and automated safety alerts
SO THAT I can dispense medications safely without manual checking

Acceptance Criteria:
- View patient allergies prominently
- Auto-check drug interactions in <3 seconds
- Block dispensing if critical interaction found
- Document override with approval workflow

Value: HIGH | Effort: MEDIUM | Risk: MEDIUM
```

**US-2 (P0): Dispense Medication Efficiently**

```
AS A pharmacist
I WANT TO complete the dispensing workflow in under 5 minutes
SO THAT patients don't wait and I can help more people

Acceptance Criteria:
- One-screen view with all information
- Auto-select correct batch (FIFO)
- Patient counseling checklist
- One-click print receipt and label

Value: HIGH | Effort: LOW | Risk: LOW
```

**US-3 (P1): Handle Generic Substitution**

```
AS A pharmacist
I WANT TO see generic alternatives when brand is out of stock
SO THAT I can still fulfill the prescription

Acceptance Criteria:
- Show alternatives with price comparison
- Quick call/SMS to prescriber for approval
- Log substitution for audit

Value: MEDIUM | Effort: MEDIUM | Risk: LOW
```

### Epic 2: Inventory Management (Priority: HIGH)

**US-4 (P0): Monitor Stock in Real-Time**

```
AS A pharmacy admin
I WANT TO see current stock levels and alerts on dashboard
SO THAT I never run out of critical medications

Acceptance Criteria:
- Real-time inventory count
- Low stock alerts (configurable threshold)
- Expiring medication alerts (90, 30, 7 days)
- One-click reorder from alert

Value: HIGH | Effort: MEDIUM | Risk: LOW
```

**US-5 (P1): Receive Shipment Accurately**

```
AS A pharmacy admin
I WANT TO verify shipments and update inventory automatically
SO THAT stock levels are always accurate

Acceptance Criteria:
- Match shipment to purchase order
- Scan barcodes for batch numbers
- Flag discrepancies (short ship, damaged)
- Auto-update inventory upon confirmation

Value: MEDIUM | Effort: MEDIUM | Risk: MEDIUM
```

**US-6 (P2): Track Expiring Medications**

```
AS A pharmacy admin
I WANT TO be alerted about expiring medications
SO THAT I can take action before they expire

Acceptance Criteria:
- Daily email summary of expiring items
- Recommended actions (discount, return, dispose)
- Block dispensing of expired drugs

Value: MEDIUM | Effort: LOW | Risk: LOW
```

### Epic 3: Financial & Reporting (Priority: MEDIUM)

**US-7 (P1): View Real-Time Financial Dashboard**

```
AS A pharmacy admin
I WANT TO see today's revenue and profit margins
SO THAT I can monitor business performance

Acceptance Criteria:
- Total revenue (prescriptions + OTC)
- Cost of goods sold
- Gross profit and margin %
- Top 5 revenue-generating drugs

Value: MEDIUM | Effort: LOW | Risk: LOW
```

**US-8 (P2): Generate Compliance Reports**

```
AS A pharmacy admin
I WANT TO export dispensing records for regulatory audits
SO THAT I can show we comply with regulations

Acceptance Criteria:
- Filter by date range
- Export to CSV/PDF
- Include all required fields (patient, drug, pharmacist, date)

Value: MEDIUM | Effort: LOW | Risk: LOW
```

### Epic 4: Supplier & Procurement (Priority: LOW)

**US-9 (P2): Manage Suppliers**

```
AS A pharmacy admin
I WANT TO maintain a database of suppliers
SO THAT I can easily reorder from preferred vendors

Acceptance Criteria:
- Add/edit supplier contact info
- Track performance (on-time delivery, quality)
- View order history with each supplier

Value: LOW | Effort: LOW | Risk: LOW
```

**US-10 (P2): Create Purchase Orders**

```
AS A pharmacy admin
I WANT TO create purchase orders quickly
SO THAT I can restock efficiently

Acceptance Criteria:
- Select supplier from dropdown
- Auto-suggest quantities based on usage
- Calculate totals with tax
- Send PO via email to supplier

Value: LOW | Effort: MEDIUM | Risk: LOW
```

---

## Feature Prioritization

### MoSCoW Method

#### Must Have (Phase 1 - Week 1-2)

- [ ] Prescription queue view with filters
- [ ] Drug interaction and allergy checking
- [ ] Dispensing workflow with patient counseling
- [ ] Real-time inventory tracking
- [ ] Low stock and expiry alerts
- [ ] Dashboard statistics

#### Should Have (Phase 2 - Week 3)

- [ ] Generic substitution lookup
- [ ] Inventory receiving workflow
- [ ] Purchase order creation
- [ ] Basic financial reports
- [ ] Supplier management

#### Could Have (Phase 3 - Week 4)

- [ ] Walk-in OTC sales
- [ ] Advanced analytics (trends, charts)
- [ ] Barcode scanning
- [ ] Bulk inventory import/export

#### Won't Have (Future Phases)

- Point-of-sale hardware integration
- Insurance claim processing API
- Patient mobile app notifications
- Delivery tracking
- Multi-location inventory sync

---

## Technical Requirements Summary

### Architecture

- **Multi-tenant SaaS** - Each pharmacy isolated by tenantId
- **Real-time updates** - WebSocket for inventory changes
- **Offline capability** - Basic read-only access (future)

### Technology Stack

- Backend: Node.js, TypeScript, Express.js, PostgreSQL, Prisma
- Frontend: React, TypeScript, shadcn/ui, Tailwind CSS
- Infrastructure: DigitalOcean, Docker, Nginx

### Integrations

- Provider Portal (receives prescriptions)
- EMR Dashboard (receives hospital medication orders)
- Drug Interaction Database (third-party API)
- Payment Gateway (future)

### Security

- JWT authentication with role-based access
- Data encryption at rest (AES-256) and in transit (HTTPS/TLS)
- Audit logging for all critical actions
- 7-year data retention for compliance

### Performance

- Dashboard load time: <2 seconds
- Drug interaction check: <3 seconds
- Support: 50 concurrent users per pharmacy
- 99.5% uptime SLA

---

## Go-to-Market Strategy

### Launch Plan

#### Phase 1: Pilot (Month 1-2)

- **Target:** 1-2 hospital pharmacies in Lagos
- **Goal:** Validate workflows, collect feedback
- **Success Criteria:** Positive NPS score, <5 critical bugs

#### Phase 2: Early Adopters (Month 3-4)

- **Target:** 5-10 pharmacies (hospitals + retail)
- **Goal:** Prove scalability, refine features
- **Success Criteria:** 90% feature adoption, <3% churn

#### Phase 3: General Availability (Month 5-6)

- **Target:** Open to all VardMD pharmacies
- **Goal:** Achieve 50+ pharmacy customers
- **Success Criteria:** 4.5+ star rating, 10% MoM growth

### Pricing Model (Future Consideration)

- **Included in VardMD subscription** - No separate charge initially
- **Pro Plan** - Advanced analytics and multi-location support (future)

### Training & Support

- Video tutorials for key workflows (15 min each)
- PDF quick reference guides
- In-app tooltips and guided tours
- Help desk support (email + WhatsApp)
- Monthly webinars for best practices

---

## Success Metrics & KPIs

### Business Metrics

| Metric                      | Measurement                         | Target                  |
| --------------------------- | ----------------------------------- | ----------------------- |
| Pharmacy adoption rate      | % of VardMD pharmacies using portal | 80% within 6 months     |
| Daily active users          | Pharmacists logging in daily        | 70% retention           |
| Prescription volume         | Prescriptions processed via system  | 10,000/month by month 6 |
| Customer satisfaction (NPS) | Net Promoter Score                  | >50                     |

### Operational Metrics

| Metric                       | Measurement                           | Target       |
| ---------------------------- | ------------------------------------- | ------------ |
| Prescription processing time | Average time from receipt to dispense | <5 minutes   |
| Medication error rate        | % of prescriptions with errors        | <1%          |
| Inventory accuracy           | Physical count vs. system count       | >98%         |
| Stockout incidents           | Times critical drug unavailable       | <3 per month |

### Financial Metrics

| Metric                   | Measurement                        | Target |
| ------------------------ | ---------------------------------- | ------ |
| Inventory turnover ratio | (COGS / Avg Inventory) per year    | >6x    |
| Gross profit margin      | (Revenue - COGS) / Revenue         | >30%   |
| Inventory shrinkage      | Unexplained loss / Total inventory | <2%    |

### Technical Metrics

| Metric         | Measurement                           | Target     |
| -------------- | ------------------------------------- | ---------- |
| System uptime  | % time system available               | >99.5%     |
| Page load time | Average dashboard load time           | <2 seconds |
| Error rate     | % of API calls that fail              | <0.1%      |
| Data accuracy  | % of transactions processed correctly | >99.9%     |

---

## Risks & Mitigation

### High Risk

**Risk 1: Drug Interaction Database Unavailable**

- **Impact:** HIGH - Safety feature fails
- **Probability:** MEDIUM
- **Mitigation:**
  - Cache common interactions locally
  - Fallback to manual pharmacist review
  - Display clear warning if API down
  - SLA with database provider

**Risk 2: Pharmacists Resistant to Change**

- **Impact:** HIGH - Low adoption
- **Probability:** MEDIUM
- **Mitigation:**
  - Involve pharmacists in design (user testing)
  - Emphasize time savings and safety benefits
  - Provide comprehensive training
  - Gradual rollout, not big-bang

**Risk 3: Data Entry Errors Corrupt Inventory**

- **Impact:** HIGH - Inaccurate stock levels
- **Probability:** LOW
- **Mitigation:**
  - Real-time validation
  - Reconciliation checks (monthly stock count)
  - Audit trail for all changes
  - Barcode scanning reduces manual entry

### Medium Risk

**Risk 4: Supplier Integration Delays**

- **Impact:** MEDIUM - Manual PO process
- **Probability:** HIGH
- **Mitigation:**
  - Start with email-based POs (no API needed)
  - Phase in API integrations supplier-by-supplier
  - Focus on top 3 suppliers first

**Risk 5: Regulatory Compliance Gaps**

- **Impact:** MEDIUM - Legal issues
- **Probability:** LOW
- **Mitigation:**
  - Consult with Pharmacy Council of Nigeria
  - Review during development
  - Compliance officer approval before launch
  - Regular audits

---

## Development Roadmap

### Phase 1: MVP (Weeks 1-2)

**Goal:** Core prescription workflow functional

**Deliverables:**

- Dashboard with stats
- Prescription queue
- Drug interaction checking
- Basic dispensing workflow
- Real-time inventory
- Low stock alerts

**Success Criteria:**

- Pharmacist can process routine prescription end-to-end
- Inventory updates automatically
- Critical interactions blocked

### Phase 2: Enhanced Features (Week 3)

**Goal:** Advanced workflows and admin features

**Deliverables:**

- Generic substitution
- Inventory receiving
- Purchase order management
- Supplier database
- Financial reports

**Success Criteria:**

- Admin can reorder stock
- Shipment receiving updates inventory correctly
- Financial data accurate

### Phase 3: Analytics & Polish (Week 4)

**Goal:** Reporting and user experience refinement

**Deliverables:**

- Advanced reports (dispensing, inventory, financial)
- Charts and trends
- Export functionality
- Walk-in OTC sales
- Mobile responsive improvements

**Success Criteria:**

- All reports generate correctly
- UI/UX polished
- No critical bugs
- Performance targets met

### Post-Launch (Ongoing)

- Bug fixes and minor improvements
- User feedback incorporation
- Performance optimization
- Additional integrations (insurance, etc.)

---

## Open Questions & Decisions Needed

### Questions for Stakeholders

**Q1:** Should system allow dispensing with active drug interaction alert if prescriber approves?

- **Options:** Require override workflow vs. Block completely
- **Decision Needed By:** Week 1
- **Impact:** Safety vs. flexibility

**Q2:** How to handle partial prescription fills when stock insufficient?

- **Options:** Dispense partial + backorder vs. Reject entire prescription
- **Decision Needed By:** Week 1
- **Impact:** Patient satisfaction vs. workflow complexity

**Q3:** Should we support walk-in sales (OTC) in Phase 1 or defer to Phase 3?

- **Options:** Include in MVP vs. Defer to later phase
- **Decision Needed By:** Week 1
- **Impact:** Scope and timeline

**Q4:** What level of barcode scanning support needed?

- **Options:** Full barcode scanning vs. Manual entry only
- **Decision Needed By:** Week 2
- **Impact:** Hardware requirements, user experience

**Q5:** How to handle controlled substance regulations (Codeine, Tramadol)?

- **Options:** Phase 1 basic logging vs. Full DEA-style reporting
- **Decision Needed By:** Week 2
- **Impact:** Compliance, development effort

---

## Appendix

### Competitive Analysis

**Direct Competitors:**

- mPharma (Ghana) - Inventory management focus
- Drugstoc (Nigeria) - Procurement platform
- HealthPlus Pharmacy (Nigeria) - ERP system

**VardMD Advantages:**

- Integrated with full EMR ecosystem
- Real-time prescription flow from doctors
- Multi-tenant SaaS (competitors are single-tenant)
- Modern UI/UX (competitors have dated interfaces)

**Gaps to Address:**

- Competitors have insurance integration (we don't yet)
- Competitors have delivery tracking (future for us)

### Regulatory Requirements (Nigeria)

**Pharmacy Council of Nigeria (PCN) Requirements:**

- Licensed pharmacist must verify prescriptions
- Controlled substance register maintained
- Patient counseling mandatory
- Records retained 7 years minimum
- Physical layout inspections (not software-related)

**Data Protection:**

- Nigeria Data Protection Regulation (NDPR) compliance
- Patient consent for data processing
- Right to access and erasure

---

## Conclusion

The VardMD Pharmacy Portal addresses critical pain points in pharmacy operations: medication errors, inventory inefficiencies, and compliance burdens. By automating safety checks, streamlining workflows, and providing real-time visibility, we will:

1. **Improve patient safety** through automated drug interaction checking
2. **Increase operational efficiency** by reducing prescription processing time by 66%
3. **Optimize inventory** to minimize waste and prevent stockouts
4. **Ensure compliance** with automated audit trails

Success will be measured by adoption rate (80% within 6 months), error reduction (<1%), and pharmacist satisfaction (>4.5/5.0).

---

**Document Status:** ✅ Ready for Stakeholder Review  
**Next Steps:**

1. Review with pharmacy stakeholders
2. Obtain approval from product owner
3. Finalize open questions
4. Hand off to development team (Hariss & Mohammed)

**Approval:**

- [ ] Product Owner
- [ ] Clinical Lead (Pharmacist)
- [ ] Technical Lead
- [ ] Compliance Officer

---

**END OF PRODUCT REQUIREMENTS DOCUMENT**
