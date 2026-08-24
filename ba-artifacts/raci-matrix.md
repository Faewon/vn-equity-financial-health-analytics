# RACI Matrix - Financial Health Early Warning System

## Project Details

| Field | Detail |
|---|---|
| **Project Name** | Financial Health Early Warning System |
| **Document No.** | RACI-001 |
| **Date** | 2026-08-24 |
| **Version No.** | 1.0 |

**Traced against:** `ba-artifacts/stakeholder-analysis.md`, `ba-artifacts/brd.md`, `ba-artifacts/use-case-diagram.png`, `ba-artifacts/bpmn-to-be.png`.

## Legend

| Code | Meaning |
|---|---|
| R | Responsible: does the work |
| A | Accountable: owns the outcome, signs off, only one A per activity |
| C | Consulted: gives input before or during the work, two-way communication |
| I | Informed: kept up to date after the fact, one-way communication |

## Roles in Scope

| Role | Source |
|---|---|
| BA (Khai Hoang) | Document creator, requirements and process design |
| Senior Financial Analyst | `stakeholder-analysis.md`, end user |
| Head of Investment Analysis | `stakeholder-analysis.md`, decision maker and final approver |
| IT/Data Engineering Team | `stakeholder-analysis.md`, implementer |
| Compliance Officer | `stakeholder-analysis.md`, risk and compliance |
| Portfolio/Fund Manager | `stakeholder-analysis.md`, internal client |

External Investors/Clients and the Data Vendor (vnstock/HOSE-HNX) are excluded from this matrix. Both are Indirect stakeholders per the stakeholder analysis and do not perform or approve any project activity; they are covered instead by the Engagement Strategy column in `stakeholder-analysis.md`.

---

## RACI Matrix

| Activity | BA | Financial Analyst | Head of Investment Analysis | IT/Data Engineering | Compliance Officer | Portfolio Manager |
|---|---|---|---|---|---|---|
| Business discovery and business questions (Phase 0) | R/A | C | C | I | I | I |
| Stakeholder analysis | R/A | I | C | I | I | I |
| BRD drafting and requirements gathering | R | C | A | C | C | I |
| BRD sign-off | I | I | A | I | C | I |
| BPMN As-Is / To-Be process design | R/A | C | C | C | I | I |
| Use case diagram and DFD design | R/A | C | I | C | I | I |
| Data source selection (vnstock, HOSE/HNX) | C | I | I | R/A | C | I |
| Legal review of data source terms of service | I | I | I | C | R/A | I |
| Bronze layer data ingestion pipeline | I | I | I | R/A | I | I |
| Silver layer standardization (dbt models) | I | I | I | R/A | I | I |
| Gold layer indicator and ratio calculation | C | C | I | R/A | I | I |
| Risk threshold definition (FR-09 rules) | C | C | A | R | I | I |
| ML risk classification model (FR-10) | C | I | C | R/A | I | I |
| Power BI dashboard build | C | C | A | R | I | I |
| SQL business-question analysis | C | R | A | C | I | I |
| UAT / pilot testing of the dashboard | I | R/A | C | C | I | I |
| Risk alert triage and investment thesis drafting | I | R/A | I | I | I | I |
| Review of analyst assessment | I | C | R/A | I | I | I |
| Approve / reject report thesis | I | I | R/A | I | I | I |
| Access provisioning to approved reports | I | I | A | R | I | I |
| Viewing approved investment reports | I | I | I | I | I | R/A |
| Data lineage and audit trail maintenance | I | I | I | R/A | C | I |
| Monthly data source compliance reporting | I | I | I | C | R/A | I |
| Production rollout approval | C | C | A | C | C | I |
| System documentation and README maintenance | R/A | I | I | C | I | I |

---

## Notes

Every row has exactly one A, consistent with the RACI convention that accountability cannot be shared.

The Head of Investment Analysis holds accountability for every activity that ends in a business decision: BRD sign-off, risk threshold definition, dashboard build, SQL analysis, assessment review and approval, access provisioning, and production rollout. This matches the stakeholder analysis, which lists this role as the final approver requiring deep consultation from the requirements stage.

The IT/Data Engineering Team is accountable for the technical pipeline itself (Bronze, Silver, Gold ingestion) since no other stakeholder has the standing or the information to own that outcome, even though the Head of Investment Analysis remains accountable for the business rules the pipeline implements (thresholds, dashboard content).

The Compliance Officer is accountable only for the two activities tied to their stated area of interest in the stakeholder analysis: legal review of data sources and the recurring compliance report. They are Consulted, not Accountable, on the BRD and rollout, and Informed elsewhere, matching their Medium influence and Low impact rating.

The Portfolio Manager is Informed throughout the build and only becomes Responsible/Accountable for the single activity that belongs to them directly: viewing approved reports. This reflects the BRD scope exclusion on automated report distribution; the Portfolio Manager pulls information rather than having it delivered to them, so they are not a stakeholder in the activities that produce it.

Two activities carry a combined R/A where the stakeholder analysis and BRD do not name a separate accountable party distinct from the person doing the work, for example the BA's own artifact production and the Financial Analyst's own thesis drafting. This is acceptable in a RACI matrix when one role is both the sole doer and the sole owner of that specific deliverable.
