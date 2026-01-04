# VardMD Lab & Imaging Portal - Product Requirements Document (PRD)

**Product:** VardMD Lab & Imaging Portal  
**Version:** 1.0  
**Date:** January 4, 2026  
**Status:** Draft for Approval  
**Owner:** Product Team

---

## Executive Summary

### Vision

Build a diagnostic services management system that eliminates lost test orders, prevents specimen mix-ups, accelerates result delivery, and ensures regulatory compliance while providing lab staff with an intuitive, efficient workflow.

### Problem Statement

**Current State:**

- Paper test orders are lost or illegible
- No sample tracking leads to specimen mix-ups
- Manual result reporting requires transcription (error-prone)
- Equipment maintenance not tracked systematically
- Compliance documentation is manual and time-consuming

**Impact:**

- 20-30% of test orders delayed due to lost paperwork
- 5% sample mix-ups requiring recollection
- 4-6 hour average TAT (should be 2-3 hours)
- Equipment downtime due to missed calibrations
- 20+ hours/week spent on manual documentation

### Success Metrics

| Metric                       | Baseline  | Target (6 months) |
| ---------------------------- | --------- | ----------------- |
| Average TAT                  | 4-6 hours | 2-3 hours         |
| Lost orders                  | 20-30%    | <1%               |
| Sample mix-ups               | 5%        | 0%                |
| Equipment downtime           | 15%       | <5%               |
| Critical result notification | 60 min    | <15 min           |
| Staff satisfaction           | N/A       | >4.5/5.0          |

---

## Product Objectives

### Primary Objectives (Must Have)

1. **Patient Safety** - Zero specimen mix-ups through barcode tracking
2. **Speed** - Reduce average TAT by 50%
3. **Quality** - Multi-stage validation before result release
4. **Compliance** - Automated audit trails, equipment tracking

### Secondary Objectives (Should Have)

5. **Efficiency** - Real-time workload visibility
6. **Analytics** - Performance metrics and reporting
7. **Equipment Management** - Calibration tracking and maintenance

---

## Target Users

### Primary Users

#### 1. Lab Technician (LAB_IMAGING Role)

**Profile:**

- Age: 25-45 years
- Education: Medical Laboratory Science degree or diploma
- License: Registered with medical laboratory board
- Tech savvy: Moderate
- Work environment: Hospital or reference lab
- Daily workflow: Process 20-40 test orders

**Pain Points:**

- ❌ Paper orders illegible or lost
- ❌ Cannot track where samples are in the lab
- ❌ Fear of mixing up specimens (patient safety risk)
- ❌ Manual result entry prone to transcription errors
- ❌ No visibility into pending workload

**Goals:**

- ✅ Process tests accurately and safely
- ✅ Track samples throughout workflow
- ✅ Upload results quickly
- ✅ Meet turnaround time targets
- ✅ End shift without backlog

**User Scenario:**

> "As a lab technician, I receive an order for a CBC. I need to verify the sample is correctly labeled, run it on the analyzer, upload the numeric results, flag any critical values, and get it validated by a senior tech - all within the 4-hour TAT window so the doctor can treat the patient promptly."

#### 2. Lab Administrator (LAB_ADMIN Role)

**Profile:**

- Age: 30-55 years
- Education: Medical Laboratory Science + management
- Experience: 5+ years in lab operations
- Responsibilities: Equipment, staff, quality, reports
- Works in: Office + lab oversight

**Pain Points:**

- ❌ No real-time visibility into lab operations
- ❌ Equipment calibration missed → regulatory violations
- ❌ Cannot identify performance bottlenecks
- ❌ Manual reports take days to compile
- ❌ Critical results delayed without notification

**Goals:**

- ✅ Ensure equipment calibrated on schedule
- ✅ Monitor turnaround time metrics
- ✅ Balance workload across staff
- ✅ Generate compliance reports instantly
- ✅ Track quality indicators

**User Scenario:**

> "As a lab manager, I need to review yesterday's test volume, identify which equipment needs calibration this month, check that STAT orders met the 1-hour target, and generate a TAT report for the medical director - all before the 10 AM meeting."

---

## User Stories & Prioritization

### Epic 1: Test Order Management (Priority: CRITICAL)

**US-1 (P0): View and Accept Test Orders**

```
AS A lab technician
I WANT TO see all pending test orders in a prioritized queue
SO THAT I can accept and process them efficiently

Acceptance Criteria:
- STAT orders at top with red indicator
- Urgent orders before routine
- Show patient, test, urgency, time ordered
- One-click accept to claim order

Value: HIGH | Effort: LOW | Risk: LOW
```

**US-2 (P0): Track Order Status in Real-Time**

```
AS A lab technician
I WANT TO update order status as I process it
SO THAT everyone knows what stage each test is in

Acceptance Criteria:
- Status workflow: pending → in-progress → result-ready → released
- Auto-refresh dashboard
- Filter by status

Value: HIGH | Effort: LOW | Risk: LOW
```

### Epic 2: Sample Management (Priority: HIGH)

**US-3 (P0): Register Sample with Barcode**

```
AS A lab technician
I WANT TO scan sample barcodes to link them to orders
SO THAT samples are correctly identified throughout processing

Acceptance Criteria:
- Barcode scanner support
- Auto-link to order
- Prevent duplicate barcodes
- Photo capture of label

Value: HIGH | Effort: MEDIUM | Risk: MEDIUM
```

**US-4 (P1): Reject Unsuitable Samples**

```
AS A lab technician
I WANT TO reject clotted or insufficient samples
SO THAT I don't process specimens that will yield invalid results

Acceptance Criteria:
- Select rejection reason from list
- Auto-notify ordering provider
- Create recollection order
- Link to original order

Value: MEDIUM | Effort: LOW | Risk: LOW
```

### Epic 3: Result Management (Priority: CRITICAL)

**US-5 (P0): Upload Test Results**

```
AS A lab technician
I WANT TO upload numeric results and attach images
SO THAT I can document test outcomes

Acceptance Criteria:
- Form for structured data entry
- Drag-and-drop file upload
- Support JPG, PNG, PDF, DICOM
- Auto-save draft

Value: HIGH | Effort: MEDIUM | Risk: LOW
```

**US-6 (P0): Validate Results Before Release**

```
AS A senior lab technician
I WANT TO review and approve results
SO THAT errors are caught before reaching doctors

Acceptance Criteria:
- Side-by-side view of patient history
- Critical values auto-flagged
- Require validation notes
- Digital signature

Value: HIGH | Effort: MEDIUM | Risk: MEDIUM
```

**US-7 (P0): Flag and Notify Critical Results**

```
AS A lab technician
I WANT TO automatically flag dangerously abnormal values
SO THAT urgent cases are prioritized

Acceptance Criteria:
- Auto-detect critical values (Glucose <50, Hb <7, etc.)
- Red alert banner
- Mandatory validation notes
- Urgent notification to provider within 15 min

Value: CRITICAL | Effort: MEDIUM | Risk: HIGH
```

### Epic 4: Equipment & Compliance (Priority: MEDIUM)

**US-8 (P2): Track Equipment Calibration**

```
AS A lab administrator
I WANT TO be alerted when equipment calibration is due
SO THAT we maintain regulatory compliance

Acceptance Criteria:
- Equipment inventory with calibration schedule
- Alerts 30 days before due
- Log calibration events
- Flag tests on uncalibrated equipment

Value: MEDIUM | Effort: MEDIUM | Risk: LOW
```

**US-9 (P2): Generate TAT Reports**

```
AS A lab administrator
I WANT TO see turnaround time metrics
SO THAT I can identify bottlenecks and improve efficiency

Acceptance Criteria:
- Average TAT per test type
- Target vs. actual
- Historical trends
- Export to Excel

Value: MEDIUM | Effort: LOW | Risk: LOW
```

---

## Feature Prioritization

### MoSCoW Method

#### Must Have (Phase 1 - Week 1-2)

- [ ] Test order queue with priority sorting
- [ ] Order acceptance and status updates
- [ ] Sample registration with barcode scanning
- [ ] Result upload (numeric and file attachments)
- [ ] Multi-stage validation workflow
- [ ] Critical value flagging
- [ ] Result release and notification
- [ ] Dashboard statistics

#### Should Have (Phase 2 - Week 3)

- [ ] Sample location tracking
- [ ] Rejection with recollection workflow
- [ ] Test catalog management
- [ ] Equipment calibration tracking
- [ ] Basic TAT reporting

#### Could Have (Phase 3 - Week 4)

- [ ] Advanced analytics and charts
- [ ] Automated analyzer interface (import results)
- [ ] Quality control tracking
- [ ] Staff workload balancing

#### Won't Have (Future Phases)

- Full LIS integration
- Insurance billing interface
- Patient result portal
- Mobile app

---

## Go-to-Market Strategy

### Launch Plan

#### Phase 1: Pilot (Month 1-2)

- **Target:** 1-2 hospital labs in Lagos
- **Goal:** Validate workflows, collect feedback
- **Success Criteria:** Positive NPS, <5 critical bugs, 50% TAT improvement

#### Phase 2: Early Adopters (Month 3-4)

- **Target:** 5-10 labs (hospital + reference labs)
- **Goal:** Prove scalability
- **Success Criteria:** 90% feature adoption, <3% churn, consistent TAT improvements

#### Phase 3: General Availability (Month 5-6)

- **Target:** All VardMD labs
- **Goal:** 30+ lab customers
- **Success Criteria:** 4.5+ star rating, 10% MoM growth

### Training & Support

- Video tutorials (test processing, result upload)
- Barcode scanner setup guide
- In-app tooltips
- Help desk (email + WhatsApp)
- Monthly best practices webinars

---

## Success Metrics & KPIs

### Business Metrics

| Metric             | Measurement                   | Target                 |
| ------------------ | ----------------------------- | ---------------------- |
| Lab adoption rate  | % of VardMD labs using portal | 80% within 6 months    |
| Daily active users | Lab techs logging in daily    | 70% retention          |
| Test volume        | Tests processed via system    | 5,000/month by month 6 |

### Operational Metrics

| Metric             | Measurement                      | Target                    |
| ------------------ | -------------------------------- | ------------------------- |
| Average TAT        | Time from order to result        | 2-3 hours                 |
| STAT TAT           | STAT test completion time        | <1 hour (100% compliance) |
| Sample mix-up rate | Incorrect specimen errors        | 0%                        |
| Equipment uptime   | % time calibrated and functional | >95%                      |

### Quality Metrics

| Metric                       | Measurement                       | Target      |
| ---------------------------- | --------------------------------- | ----------- |
| Critical result notification | Time to notify provider           | <15 minutes |
| Validation rate              | % results reviewed before release | 100%        |
| Rejected sample rate         | % samples unsuitable              | <3%         |

---

## Risks & Mitigation

### High Risk

**Risk 1: Sample Barcode System Failures**

- **Impact:** HIGH - Cannot track specimens
- **Probability:** MEDIUM
- **Mitigation:**
  - Manual entry fallback
  - Require photos of sample labels
  - Daily scanner testing
  - Backup scanners available

**Risk 2: Lab Staff Resistant to Digital Workflow**

- **Impact:** HIGH - Low adoption
- **Probability:** MEDIUM
- **Mitigation:**
  - Involve lab managers in design
  - Emphasize time savings and safety
  - Hands-on training sessions
  - Gradual rollout, not big-bang

**Risk 3: Critical Results Not Delivered Urgently**

- **Impact:** CRITICAL - Patient harm
- **Probability:** LOW
- **Mitigation:**
  - Redundant notification channels (app + SMS + email)
  - Acknowledgment required
  - Escalation if no response within 10 min
  - Audit log for all critical notifications

---

## Development Roadmap

### Phase 1: Core Workflow (Weeks 1-2)

**Deliverables:**

- Order queue and acceptance
- Sample registration
- Result upload
- Validation workflow
- Release and notification

**Success Criteria:**

- Tech can process routine test end-to-end
- Critical results flagged
- Notifications sent

### Phase 2: Quality & Equipment (Week 3)

**Deliverables:**

- Sample rejection workflow
- Test catalog management
- Equipment calibration tracking
- Basic TAT reporting

**Success Criteria:**

- Recollection automated
- Equipment alerts working
- TAT calculated correctly

### Phase 3: Analytics & Polish (Week 4)

**Deliverables:**

- Advanced reports
- Charts and trends
- Mobile responsive improvements
- Export functionality

**Success Criteria:**

- All reports generate correctly
- UI/UX polished
- Performance targets met

---

## Open Questions

**Q1:** Should system allow releasing results without validation in emergencies?

- **Options:** Require override workflow vs. Block completely
- **Decision Needed By:** Week 1

**Q2:** How to handle tests ordered but never performed (patient no-show)?

- **Options:** Auto-cancel after 7 days vs. Require manual cancellation
- **Decision Needed By:** Week 1

**Q3:** Should barcode scanning be required or optional?

- **Options:** Mandatory (safest) vs. Optional with manual entry
- **Decision Needed By:** Week 1

**Q4:** Integration level with automated analyzers?

- **Options:** Full bi-directional interface vs. Manual result entry
- **Decision Needed By:** Week 2

---

## Conclusion

The VardMD Lab & Imaging Portal addresses critical pain points: lost orders, specimen mix-ups, slow turnaround times, and compliance burdens. By digitalizing workflows, implementing barcode tracking, and automating notifications, we will:

1. **Improve patient safety** through zero specimen mix-ups
2. **Increase efficiency** by reducing TAT by 50%
3. **Ensure quality** with mandatory multi-stage validation
4. **Maintain compliance** with automated audit trails

Success measured by 80% adoption, <3 hour TAT, and >4.5/5.0 staff satisfaction.

---

**Document Status:** ✅ Ready for Stakeholder Review  
**Approval Required:**

- [ ] Product Owner
- [ ] Lab Director
- [ ] Technical Lead
- [ ] Compliance Officer

---

**END OF PRODUCT REQUIREMENTS DOCUMENT**
