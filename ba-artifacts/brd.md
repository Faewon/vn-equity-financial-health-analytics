# Business Requirements Document (BRD)

---

## Project Details

| Field | Detail |
|---|---|
| **Project Name** | Financial Health Early Warning System |
| **Creator** | Khai Hoang |
| **Document No.** | BRD-001 |
| **Date** | 2026-08-23 |
| **Version No.** | 1.0 |

---

## 1. Executive Summary Snapshot

This Business Requirements Document (BRD) defines the business need, scope, and requirements for the **Financial Health Early Warning System**, a data analytics solution designed to help investment analysts monitor the financial health of publicly listed companies and detect early warning signs of financial risk (e.g., declining cash flow, rising leverage, deteriorating profitability).

Currently, financial analysts manually collect quarterly financial statements from multiple sources and calculate financial ratios in spreadsheets. This process is time-consuming, error-prone, and reactive rather than proactive. This project proposes an automated pipeline that ingests financial statement data, calculates key financial health indicators, and surfaces risk signals through an interactive dashboard.

**Purpose of this BRD:** To document the business problem, scope, and functional/non-functional requirements needed to design and build the system, and to serve as the reference point of alignment between business stakeholders and the implementation team.

**Audience:** Head of Investment Analysis, Senior Financial Analysts, IT/Data Engineering Team, Compliance Officer, and Portfolio/Fund Managers.

---

## 2. Project Description

Investment analysts at [Company] are responsible for tracking the financial performance of listed companies within their coverage universe to support investment recommendations. Today, this process relies on manually downloading quarterly financial statements (balance sheet, income statement, cash flow statement) from public sources, entering data into Excel, and calculating ratios such as ROE, ROA, and leverage by hand.

This manual approach creates three core challenges:
1. **Time inefficiency**: consolidating data for a single company can take hours; comparing multiple companies across a sector is even slower.
2. **Inconsistent risk detection**: there is no systematic method to flag early warning signals (e.g., negative free cash flow while profits appear healthy), so risks are often identified only after they materialize in stock price movements.
3. **Data entry errors**: manual copy-paste from financial statements introduces the risk of transcription mistakes that can lead to flawed analysis.

The project aims to automate the data collection, standardization, and risk-indicator calculation process, and present the results through a self-service dashboard, reducing analyst workload and enabling earlier detection of financial risk.

---

## 3. Project Scope

| IN-SCOPE ITEMS | OUT-OF-SCOPE ITEMS |
|---|---|
| Item 1: Quarterly financial statement data (balance sheet, income statement, cash flow) for 8–15 listed companies within a single sector | Item 1: Real-time intraday financial/market data |
| Item 2: Automated data ingestion from public sources (e.g., vnstock API, HOSE/HNX official disclosures) | Item 2: Consolidated financial statements for multinational conglomerates |
| Item 3: Calculation of core financial health indicators (ROE, ROA, leverage ratios, liquidity ratios, free cash flow) | Item 3: Predictive stock price forecasting models |
| Item 4: Interactive Power BI dashboard for cross-company and cross-period comparison | Item 4: Direct integration with trading/order execution systems |
| Item 5: Rule-based risk alerting (e.g., flag companies breaching pre-defined financial thresholds) | Item 5: Mobile application version of the dashboard |
| Item 6: Lightweight ML model for financial health classification/clustering | Item 6: Automated report generation and distribution via email |

---

## 4. Business Drivers

**Business Driver 1: Analyst Time Efficiency**
Manual data consolidation currently consumes a significant portion of analysts' working hours each reporting quarter. Automating this process frees analysts to focus on higher-value interpretive and advisory work rather than data entry.

**Business Driver 2: Proactive Risk Detection**
Early identification of financial deterioration (e.g., a company's earnings growing while operating cash flow declines) allows the investment team to reassess exposure before the risk is reflected in market price — protecting client portfolios and firm reputation.

**Business Driver 3: Data Consistency and Auditability**
A standardized, automated pipeline ensures that financial ratios are calculated using a consistent methodology across all companies and time periods, reducing the risk of human error and improving the defensibility of analyst recommendations.

**Business Driver 4: Scalability of Coverage**
An automated system allows the team to expand the number of companies and sectors monitored without a proportional increase in analyst headcount.

---

## 5. Present Process

The current process for financial health monitoring is entirely manual:

1. Analyst identifies which companies require review for the current quarter.
2. Analyst manually downloads financial statements (PDF/Excel) from HOSE/HNX disclosure portals or company investor relations pages.
3. Analyst copies relevant line items into an internal Excel workbook.
4. Analyst manually calculates financial ratios (ROE, ROA, leverage, etc.) using Excel formulas.
5. Analyst manually compares ratios across companies and prior periods to identify anomalies or risk signals.
6. Analyst compiles findings into a Word document and shares with the Head of Investment Analysis.

*(See `ba-artifacts/bpmn-as-is.png` for the visual process flow, including swimlanes for Analyst, and approval step by Head of Investment Analysis.)*

**Key pain points:** high manual effort, delayed risk detection, inconsistent ratio calculation methodology across analysts, difficulty scaling to more companies.

---

## 6. Proposed Process

The proposed process automates data collection and calculation, leaving analysts to focus on interpretation and decision-making:

1. System automatically retrieves newly published quarterly financial statements on a scheduled basis (e.g., via vnstock API).
2. Data pipeline ingests raw data into a Bronze layer, then standardizes it into a Silver layer (consistent units, validated data types).
3. System automatically calculates financial health indicators into a Gold layer (ROE, ROA, leverage, liquidity, free cash flow, growth rates).
4. System evaluates each company against pre-defined risk thresholds (decision gateway).
   - If a threshold is breached → system triggers an alert to the analyst.
   - If not → dashboard is updated with the latest data as usual.
5. Analyst reviews the Power BI dashboard, drills down into flagged companies, and documents an investment view.
6. Head of Investment Analysis reviews and approves the analyst's assessment before it is shared with Portfolio/Fund Managers.

*(See `ba-artifacts/bpmn-to-be.png` for the visual process flow.)*

---

## 7. Functional Requirements

### Priority Table

| VALUE | STATUS | DESCRIPTION |
|---|---|---|
| 1 | Immediate | The requirement is critical to the project's success. Without fulfilling this requirement, the project is not possible. |
| 2 | High | The requirement is high priority for the project's success, but the project could still be implemented in a minimum viable product (MVP) scenario. |
| 3 | Moderate | The requirement is important to the project's success, as it provides value, but the project could still be implemented in an MVP scenario. |
| 4 | Low | The requirement is of low priority, but the project's success is not dependent upon it. |
| 5 | Prospective | The requirement is out of the project's scope and is included as a possible component of a prospective release and/or feature. |

### Categories (RC1) — Data Ingestion & Processing

| ID | Requirement | Priority | Raised By |
|---|---|---|---|
| FR-01 | System must automatically retrieve quarterly financial statements (balance sheet, income statement, cash flow) for the defined list of companies | 1 | Head of Investment Analysis |
| FR-02 | System must standardize raw financial data into consistent units and validated data types (Silver layer) | 1 | IT/Data Engineering Team |
| FR-03 | System must calculate core financial ratios (ROE, ROA, gross margin, leverage, liquidity, free cash flow) at the company-period level | 1 | Senior Financial Analyst |
| FR-04 | System must support historical trend calculation (YoY, QoQ growth) for all key indicators | 2 | Senior Financial Analyst |

### Categories (RC2) — Dashboard & Reporting

| ID | Requirement | Priority | Raised By |
|---|---|---|---|
| FR-05 | Dashboard must display a sector-level overview with ranking of companies by key financial ratios | 1 | Senior Financial Analyst |
| FR-06 | Dashboard must allow drill-down into a single company's financial trend across quarters | 1 | Senior Financial Analyst |
| FR-07 | Dashboard must visually highlight companies with high ROE combined with high leverage ("inflated ROE" risk pattern) | 2 | Head of Investment Analysis |
| FR-08 | Dashboard must support filtering by sector and time period | 2 | Senior Financial Analyst |

### Categories (RC3) — Risk Alerting & Classification

| ID | Requirement | Priority | Raised By |
|---|---|---|---|
| FR-09 | System must flag companies whose operating cash flow is negative for two or more consecutive quarters | 1 | Head of Investment Analysis |
| FR-10 | System must classify each company into a financial health category (Healthy / Watch / At Risk) using a lightweight ML model | 3 | Senior Financial Analyst |
| FR-11 | System must allow analysts to view the rationale/inputs behind each risk classification | 2 | Compliance Officer |

---

## 8. Non-Functional Requirements

| ID | Requirement |
|---|---|
| NFR-01 | Data refresh latency must not exceed 24 hours after a company's official financial statement is published |
| NFR-02 | Dashboard must load within 5 seconds for standard queries on the defined dataset (8–15 companies) |
| NFR-03 | All data sources used must be publicly available and comply with the source's terms of service |
| NFR-04 | System must maintain a data lineage trail (Bronze → Silver → Gold) for auditability |
| NFR-05 | Dashboard access must be restricted to authorized internal users (Analysts, Head of Investment Analysis, Portfolio Managers) |
| NFR-06 | System documentation (BRD, data dictionary, BPMN diagrams) must be maintained and version-controlled |

---

## 9. Glossary

| Term/Abbreviation | Explanation |
|---|---|
| BCTC | Financial statements (Vietnamese: Báo cáo tài chính) |
| ROE | Return on Equity |
| ROA | Return on Assets |
| FCF | Free Cash Flow |
| YoY | Year-over-Year |
| QoQ | Quarter-over-Quarter |
| Bronze/Silver/Gold | Medallion data architecture layers representing raw, cleansed, and business-ready data |
| BPMN | Business Process Model and Notation |
| MVP | Minimum Viable Product |
| HOSE/HNX | Ho Chi Minh Stock Exchange / Hanoi Stock Exchange |

---

## 10. References

| Name | Location |
|---|---|
| vnstock library documentation | https://github.com/thinh-vu/vnstock |
| HOSE official disclosure portal | https://cbtt.hsx.vn |
| HNX official disclosure portal | https://www.hnx.vn |
| BPMN As-Is Diagram | `ba-artifacts/bpmn-as-is.png` |
| BPMN To-Be Diagram | `ba-artifacts/bpmn-to-be.png` |
| Stakeholder Analysis | `ba-artifacts/stakeholder-analysis.md` |

---

## 11. Appendix

- Financial health risk thresholds (e.g., leverage ratio limits, cash flow warning conditions) to be finalized in consultation with the Head of Investment Analysis during the requirements refinement phase.
- Sector selection and company list to be confirmed prior to data ingestion (see Project Scope, Item 1).
- Future releases may consider expanding coverage to additional sectors or incorporating real-time market data (currently out of scope — see Section 3).

---

*Disclaimer: This document is a personal portfolio project created to demonstrate Business Analyst skills (requirements gathering, process modeling, and documentation) and does not represent an actual engagement with a named organization.*
