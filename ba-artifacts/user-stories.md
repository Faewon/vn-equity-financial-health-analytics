# User Stories - Financial Health Early Warning System

## Project Details

| Field | Detail |
|---|---|
| **Project Name** | Financial Health Early Warning System |
| **Document No.** | US-001 |
| **Date** | 2026-08-24 |
| **Version No.** | 1.0 |

**Traced against:** `ba-artifacts/brd.md` (Functional Requirements FR-01 to FR-11, Non-Functional Requirements NFR-01 to NFR-06) and `ba-artifacts/use-case-diagram.png` (Actors: Financial Analyst, Head of Investment Analysis, Portfolio Manager, Scheduler).
**Format:** As a [role], I want [goal], so that [benefit], with Acceptance Criteria in Gherkin (Given/When/Then).
**Priority convention:** Must-have (M) / Should-have (S) / Could-have (C), MoSCoW.
**Estimate convention:** Story Points (SP) on the Fibonacci scale (1, 2, 3, 5, 8), indicative only.

> **Coverage note:** every story below is traced to an FR/NFR id where one exists. Three stories (US-11, US-12, US-13) trace only to a Business Question from the Phase 0 discovery, not to an approved FR. They are marked as proposed backlog items pending a BRD change request, not committed scope.

---

## Epic 0: Authentication & Access Control

Non-functional basis: NFR-05 (dashboard access restricted to authorized internal users). This is an implicit precondition for every dashboard-related use case, not a named use case in the diagram.

### US-00: Log in to the system

**As a** system user (Financial Analyst, Head of Investment Analysis, or Portfolio Manager)
**I want** to log in with my company account
**So that** I can access only the dashboard views and data appropriate to my role

- **Priority:** M
- **SP:** 3
- **Related requirement:** NFR-05
- **Acceptance Criteria:**
  ```gherkin
  Given I have a valid account issued by IT
  When I enter correct credentials on the login page
  Then I am redirected to the dashboard, restricted to the views permitted for my role

  Given I enter incorrect credentials 3 times in a row
  When I attempt to log in a 4th time
  Then the system temporarily locks the account and directs me to contact IT

  Given I am not one of the authorized roles (Financial Analyst, Head of Investment Analysis, Portfolio Manager)
  When I attempt to log in
  Then access is denied, per NFR-05
  ```

---

## Epic 1: Automated Data Pipeline (Bronze, Silver, Gold)

Use case: Trigger Data Ingestion & Refresh (Medallion). Actor: Scheduler (System).

### US-01: Automatically ingest newly published financial statements

**As the** Scheduler (system)
**I want** to automatically retrieve newly disclosed quarterly financial statements for the defined company list via the vnstock API
**So that** dashboard data always reflects the latest officially disclosed figures without manual effort

- **Priority:** M
- **SP:** 8
- **Related requirement:** FR-01, NFR-01 (refresh latency under 24 hours after official disclosure)
- **Acceptance Criteria:**
  ```gherkin
  Given a company on the defined list has just disclosed a new quarterly financial statement
  When the scheduled cron trigger runs
  Then the Bronze layer is updated with the new statement within 24 hours of official disclosure (NFR-01)

  Given the vnstock API is unresponsive or returns an error
  When the ingestion job runs
  Then the system logs the error, alerts the IT/Data Team, and leaves previously ingested data untouched
  ```

### US-02: Standardize data from Bronze to Silver

**As a** data engineer (IT/Data Engineering Team)
**I want** raw financial data standardized into consistent units and validated data types
**So that** figures from different tickers and periods are comparable before ratio calculation

- **Priority:** M
- **SP:** 5
- **Related requirement:** FR-02
- **Acceptance Criteria:**
  ```gherkin
  Given Bronze data contains raw net_revenue, gross_profit, and net_income fields of mixed types
  When the Silver-layer standardization job runs
  Then all monetary fields are cast to a consistent numeric type and records with a null net_revenue are excluded

  Given the Silver job completes
  When dbt tests run against the Silver models
  Then not_null and unique tests on the (ticker, period) key all pass
  ```

### US-03: Compute financial ratios, growth rates, and ML risk labels (Gold layer)

**As a** system
**I want** to calculate ROE, ROA, gross margin, leverage, liquidity, free cash flow, and YoY/QoQ growth for every ticker-period, plus an ML-based risk label
**So that** analysts have ready-made indicators instead of computing them manually in Excel

- **Priority:** M
- **SP:** 8
- **Related requirement:** FR-03, FR-04, FR-10
- **Acceptance Criteria:**
  ```gherkin
  Given Silver data has complete income statement, balance sheet, and cash flow records for a ticker/period
  When the Gold-layer job runs
  Then the Gold table (fct_chi_so) includes gross_margin, roe, roa, leverage, liquidity ratios, and FCF (FR-03)

  Given at least 2 prior periods of Gold data exist for a ticker
  When the Gold job runs
  Then YoY and QoQ growth rates are computed for each core indicator (FR-04)

  Given the Gold-layer indicators for a ticker-period are complete
  When the ML classification step runs
  Then the ticker is labeled Healthy, Watch, or At Risk in the Gold layer (FR-10)
  ```

### US-04: Evaluate risk-threshold rules and trigger alerts

**As a** system
**I want** to evaluate each freshly computed ticker-period against pre-defined risk thresholds
**So that** a warning is raised automatically as soon as a breach is detected, rather than waiting for an analyst to notice it manually

- **Priority:** M
- **SP:** 5
- **Related requirement:** FR-09. Use case: Evaluate Risk Rules & Classify Health (ML), included by Trigger Data Ingestion & Refresh
- **Acceptance Criteria:**
  ```gherkin
  Given a company's operating cash flow is negative for its 2 most recent consecutive quarters
  When the risk-threshold evaluation gateway runs after a Gold refresh
  Then an early warning alert is generated and the ticker is flagged (FR-09)

  Given no threshold is breached for a ticker-period
  When the evaluation gateway runs
  Then the Gold dataset is published and the Power BI dashboard is refreshed without an alert
  ```

---

## Epic 2: Sector Dashboard & Company Comparison

Use case: View Sector Dashboard, Drill-down Company Trend (extend). Actor: Financial Analyst.

### US-05: View the sector ranking table

**As a** Financial Analyst
**I want** to see key financial ratios for every company in a sector, ranked, on one dashboard
**So that** I can compare companies quickly without opening multiple Excel files

- **Priority:** M
- **SP:** 5
- **Related requirement:** FR-05, FR-08 (sector/period filtering), NFR-02 (load time)
- **Acceptance Criteria:**
  ```gherkin
  Given I am logged in and open the "Sector Overview" page
  When I select a sector
  Then a ranking table of that sector's tickers by ROE, ROA, and revenue growth loads within 5 seconds (NFR-02)

  Given I want to narrow the view
  When I apply the sector filter and/or the time-period filter (FR-08)
  Then the ranking table updates to match the selected sector and period
  ```

### US-06: View a company's financial health classification

**As a** Financial Analyst
**I want** to see each company's Healthy, Watch, or At Risk label next to its ranking row
**So that** I can immediately spot which companies need closer attention

- **Priority:** S
- **SP:** 3
- **Related requirement:** FR-10
- **Acceptance Criteria:**
  ```gherkin
  Given the Gold layer has produced a health label for a ticker-period
  When I view the sector ranking table
  Then the corresponding row displays the Healthy, Watch, or At Risk label
  ```

### US-07: View a company's indicator trend (drill-down)

**As a** Financial Analyst
**I want** to click a company in the ranking table to see a trend chart of its indicators across quarters
**So that** I understand that company's detailed financial trajectory over time

- **Priority:** M
- **SP:** 5
- **Related requirement:** FR-06. Use case: Drill-down Company Trend (extend of View Sector Dashboard)
- **Acceptance Criteria:**
  ```gherkin
  Given I am viewing the sector ranking table
  When I click a company's name
  Then the dashboard shows trend charts of ROE, ROA, leverage, liquidity, and FCF across quarters for that company (FR-06)
  ```

### US-08: Highlight the inflated ROE risk pattern

**As a** Financial Analyst
**I want** companies with high ROE combined with abnormally high leverage visually highlighted
**So that** I can quickly identify ROE driven by financial leverage rather than genuine operating performance

- **Priority:** S
- **SP:** 5
- **Related requirement:** FR-07
- **Acceptance Criteria:**
  ```gherkin
  Given Gold data has ROE and leverage for all companies in a selected period
  When I open the risk view
  Then companies in the top quartile of both ROE and leverage are visually highlighted as a distinct cluster (FR-07)
  ```

### US-09: Export report and assessment thesis

**As a** Financial Analyst
**I want** to export my current dashboard view together with my drafted investment thesis
**So that** I have a self-contained file to attach when submitting my assessment for review

- **Priority:** C
- **SP:** 3
- **Related use case:** Export Report & Assessment Thesis
- **Acceptance Criteria:**
  ```gherkin
  Given I have drafted an investment thesis for a flagged company
  When I click "Export"
  Then the system generates a file containing the underlying indicators, applied filters, and my thesis text
  ```
- **Note:** this exports a file the analyst downloads and attaches manually to their submission. It is not an automated distribution channel (see the scope note under Epic 4).

---

## Epic 3: Early Warning Alerts

Use case: Receive Early Risk Alert. Actor: Financial Analyst.

### US-10: Receive alert on consecutive negative operating cash flow

**As a** Financial Analyst
**I want** to be alerted when a company's operating cash flow is negative for 2 consecutive quarters
**So that** I can assess the risk promptly

- **Priority:** M
- **SP:** 5
- **Related requirement:** FR-09
- **Acceptance Criteria:**
  ```gherkin
  Given the system has flagged a company per FR-09
  When I open the dashboard
  Then the flagged company appears in a visible alert list, and remains flagged only while the breach condition holds
  ```

### US-11: Detect the "growing on debt" pattern

**As a** Financial Analyst
**I want** to compare revenue growth against debt growth for companies in the same sector over the last 8 quarters
**So that** I can identify which companies are "growing on debt" rather than through genuine business performance

- **Priority:** S (proposed, not yet in BRD scope)
- **SP:** 5
- **Related requirement:** none currently approved. Traces to Business Question BQ-2 from the discovery phase. FR-04 already computes the growth rates this story needs; what is missing is a dedicated comparison view and rule. Recommend raising as proposed FR-12 before sprint commitment.
- **Acceptance Criteria:**
  ```gherkin
  Given Gold data has YoY revenue growth and YoY debt growth for the last 8 quarters (FR-04)
  When I open a "Revenue vs Debt Growth" view
  Then companies where debt growth exceeds revenue growth are flagged
  ```

### US-12: Detect costs growing faster than revenue

**As a** Financial Analyst
**I want** to be alerted when a company's cost growth outpaces its revenue growth
**So that** I can spot early signs of margin erosion

- **Priority:** S (proposed, not yet in BRD scope)
- **SP:** 3
- **Related requirement:** none currently approved. Traces to Business Question BQ-4. Recommend proposed FR-13.
- **Acceptance Criteria:**
  ```gherkin
  Given a company's QoQ cost growth exceeds its QoQ revenue growth
  When the dashboard refreshes
  Then that company appears in a "Cost growing faster than revenue" alert list
  ```

### US-13: Detect dividend risk when FCF is negative

**As a** Financial Analyst
**I want** to be alerted when a company pays steady dividends while free cash flow stays negative
**So that** I can flag a long-term liquidity risk

- **Priority:** S (proposed, not yet in BRD scope)
- **SP:** 5
- **Related requirement:** none currently approved. Traces to Business Question BQ-5. Recommend proposed FR-14.
- **Acceptance Criteria:**
  ```gherkin
  Given a company has paid dividends in 4 or more consecutive quarters
  And its FCF is negative over the same period
  When the risk evaluation job runs
  Then that company is added to a "Dividend paid despite negative FCF" alert list
  ```

---

## Epic 4: Review, Approval, and Sign-off

Use case: Review Analyst Assessment, Approve / Reject Report Thesis (include). Actor: Head of Investment Analysis.

**Scope note:** the BRD lists "automated report generation and distribution via email" as out of scope (BRD Section 3, Item 6). Every story in this epic and Epic 5 follows a pull model: approval makes a thesis visible on the dashboard, and the Portfolio Manager views it there. No story in this document assumes an automated email push.

### US-14: View the rationale behind a risk classification

**As the** Head of Investment Analysis
**I want** to see the specific indicators and thresholds behind a company's risk label
**So that** I can judge whether the recommendation is defensible before approving it

- **Priority:** M
- **SP:** 5
- **Related requirement:** FR-11 (raised by Compliance Officer, for auditability); NFR-04 (data lineage Bronze to Silver to Gold)
- **Acceptance Criteria:**
  ```gherkin
  Given a company carries a risk label (for example "At Risk")
  When I open that company's detail view
  Then I see the underlying indicator values, the thresholds applied, and a lineage trail back to the source Bronze record (FR-11, NFR-04)
  ```

### US-15: Review the analyst's assessment thesis

**As the** Head of Investment Analysis
**I want** to review the investment thesis an analyst has drafted for a flagged company
**So that** I have full context before deciding whether to approve it

- **Priority:** M
- **SP:** 3
- **Related use case:** Review Analyst Assessment
- **Acceptance Criteria:**
  ```gherkin
  Given an analyst has submitted a thesis for a flagged company
  When I open my pending-review queue
  Then I see the submitted thesis along with the analyst's name and submission time
  ```

### US-16: Approve or reject a report thesis

**As the** Head of Investment Analysis
**I want** to approve or reject, with a stated reason, a submitted assessment
**So that** only a signed-off recommendation becomes the final "Approved Investment Thesis" available to Portfolio Managers

- **Priority:** M
- **SP:** 5
- **Related use case:** Approve / Reject Report Thesis (includes Review Analyst Assessment)
- **Acceptance Criteria:**
  ```gherkin
  Given I am reviewing a thesis in "Pending Review" status
  When I approve it
  Then it becomes the final "Approved Investment Thesis" and is made visible to the relevant Portfolio Manager on their dashboard

  Given I disagree with the assessment
  When I reject it and enter a reason
  Then the status reverts to "Rework" and the analyst sees the rejection reason
  ```

---

## Epic 5: Portfolio Manager Access

Use case: View Approved Investment Report. Actor: Portfolio Manager.

### US-17: View approved investment reports

**As a** Portfolio Manager
**I want** to view the list of approved investment theses relevant to my portfolio
**So that** I can consider rebalancing in light of the latest signed-off risk assessments

- **Priority:** S
- **SP:** 3
- **Related use case:** View Approved Investment Report
- **Acceptance Criteria:**
  ```gherkin
  Given a thesis has been approved by the Head of Investment Analysis (US-16)
  When I log in and open "Approved Reports"
  Then I see the approved thesis, its supporting indicators, and the approval date

  Given no new thesis has been approved since my last visit
  When I open "Approved Reports"
  Then the list simply shows no new items; the system does not push a notification, per the scope note under Epic 4
  ```

---

## Epic 6: Data Governance & Compliance

Not a system use case. The Compliance Officer has no dashboard access, per NFR-05. This is an off-system reporting process between IT/Data and Compliance.

### US-18: Confirm the legitimacy of data sources

**As a** Compliance Officer
**I want** a periodic summary of where financial data comes from and how it is collected
**So that** I can confirm the system only uses data collected lawfully and consistent with each source's Terms of Service

- **Priority:** S
- **SP:** 2
- **Related requirement:** NFR-03 (all sources public and ToS-compliant)
- **Acceptance Criteria:**
  ```gherkin
  Given the system is using one or more data sources (vnstock, HOSE/HNX)
  When the monthly reporting cycle arrives
  Then the IT/Data Team sends the Compliance Officer a summary of data sources in use, outside the dashboard, confirming ToS compliance (NFR-03)
  ```
- **Note:** delivered as a document or email between IT and Compliance, not a dashboard feature. This does not conflict with the scope exclusion on automated report distribution, which applies to the Analyst to Head to Portfolio Manager insight-delivery pipeline that the system itself automates.

---

## Traceability Matrix

| User Story | FR / NFR | Business Question | Use Case |
|---|---|---|---|
| US-00 | NFR-05 | - | implicit precondition |
| US-01 | FR-01, NFR-01 | - | Trigger Data Ingestion & Refresh (Medallion) |
| US-02 | FR-02 | - | Trigger Data Ingestion & Refresh (Medallion) |
| US-03 | FR-03, FR-04, FR-10 | - | Evaluate Risk Rules & Classify Health (ML) |
| US-04 | FR-09 | BQ-1 | Evaluate Risk Rules & Classify Health (ML) |
| US-05 | FR-05, FR-08, NFR-02 | - | View Sector Dashboard |
| US-06 | FR-10 | - | View Sector Dashboard |
| US-07 | FR-06 | - | Drill-down Company Trend |
| US-08 | FR-07 | BQ-3 | View Sector Dashboard |
| US-09 | - | - | Export Report & Assessment Thesis |
| US-10 | FR-09 | BQ-1 | Receive Early Risk Alert |
| US-11 | proposed FR-12 | BQ-2 | Receive Early Risk Alert |
| US-12 | proposed FR-13 | BQ-4 | Receive Early Risk Alert |
| US-13 | proposed FR-14 | BQ-5 | Receive Early Risk Alert |
| US-14 | FR-11, NFR-04 | - | Review Analyst Assessment |
| US-15 | - | - | Review Analyst Assessment |
| US-16 | - | - | Approve / Reject Report Thesis |
| US-17 | - | - | View Approved Investment Report |
| US-18 | NFR-03 | - | off-system |

**Requirements with no story:** none. All of FR-01 to FR-11 and NFR-01 to NFR-06 are covered by at least one story above.
**Stories with no approved requirement:** US-11, US-12, US-13. Resolve by either raising a BRD change request for proposed FR-12 to FR-14, or moving these three to a "Future Work" backlog before sprint planning, consistent with the project's guiding principle that every technical step must trace back to a specific business question or requirement.
