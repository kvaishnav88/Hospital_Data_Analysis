## Overview
Hospitals rarely run on a single system. Billing, patient-visit logging, and insurance/claims processing are typically handled by separate platforms that were never designed to talk to each other, creating data silos where each department sees its own slice of the picture but no one can answer questions spanning the whole patient journey. This project simulates exactly that scenario, unifying three real-world-style hospital extracts into a single 16,198-record master dataset powering one integrated Power BI dashboard.

## Problem Statement
Hospital leadership cannot get a single, trustworthy view of patient volume, cost, and outcomes because the data that answers those questions is scattered across three disconnected systems with different formats, column names, and — critically — hidden data quality issues that would silently distort any KPI built naively on top of them.

## Dataset
Three source extracts merged into one master table:

| Source | What it tracks | Rows |
|---|---|---|
| Billing System | Doctor, Department, Payment Method, Length of Stay | 1,300 |
| Visit Log | Department, Diagnosis, Charges | 5,000 |
| Insurance System | Insurance status, Recovery outcome, City | 10,000 |

Final merged dataset: **16,198 patient records**.

## Methodology & Techniques

**1. Data Profiling**
Inspected each raw source's schema, null rates, categorical value sets, and date ranges to determine what could be safely merged versus what would need reconciliation — found that only 1 of 3 sources records Payment Method, and only 1 of 3 records Insurance/Recovery outcome, meaning the merged schema would necessarily be sparse in places (handled with explicit "Not Recorded" categories rather than silent nulls).

**2. Data Cleaning & Multi-Source Integration (Python / pandas)**
- Renamed and re-typed columns from all three sources into one common schema.
- Prefixed Patient IDs by source (`BIL-`, `VIS-`, `INS-`) to guarantee global uniqueness after the union, since none of the three ID schemes overlapped or matched a common key.
- Standardized inconsistent text fields (Gender casing, trimmed whitespace).
- Engineered time-based features: `AgeGroup`, `AdmissionMonth`, `AdmissionDayOfWeek`, `AdmissionHour` — used for the dashboard's seasonality and capacity-planning views.
- **Data quality auditing (the technical centerpiece of this project):** computed the gap between Admission and Discharge dates for Insurance System records and discovered that **89% (8,849 of 9,914) had a gap exceeding 90 days** — clinically implausible for a hospital stay. Rather than silently blending this into a single "Length of Stay" KPI (which would have produced a misleading average), the anomalous records were separated into a `RecordDurationDays` field with a `DateGapFlag`, and the trusted `StayDays` KPI was calculated only from clinically plausible records (≤90 days).
- Removed physically impossible values (negative charges, ages outside 0–120).

**3. Analysis Layer (SQL)**
Re-implemented the same cleaning/merge logic as SQL staging tables feeding a `master_hospital_data` view, so the pipeline runs natively in Postgres/MySQL/SQL Server as an alternative to the Python path. Wrote KPI and breakdown queries by department, diagnosis, demographics, time trend, payment method, and insurance/recovery outcome.

**4. Visualization (Power BI)**
Imported the cleaned master table, built DAX measures for headline KPIs, and laid the dashboard out following standard BI storytelling convention: KPI cards → categorical breakdowns → time trends → demographic detail.

## Key Results
- Total patients: **16,198** | Total revenue: **$390.5M** | Avg charge/patient: **$24,109.50**
- Avg length of stay (plausible records only): **25.67 days**, blending short Billing-System visits (~3.3 days avg) with longer Insurance-System cases (~52.6 days avg, capped at 90)
- Cardiology (1,245), Oncology (1,155), and Neurology (1,134) are the highest-volume departments
- Diabetes, Cancer, and Flu are the three most frequent diagnoses (2,445 / 2,411 / 2,296 cases)
- COVID-19 and Injury cases carry the highest average cost per case (~$25,250)
- Recovery rate is nearly identical regardless of insurance status (50.4% uninsured vs. 49.3% insured) — a notable, actionable finding
- **Data quality finding:** 89% of Insurance System records have an implausible admission-discharge gap, pointing to a likely timestamp entry bug — flagged as a concrete IT/records-team action item rather than hidden

## Tech Stack
`Python (pandas, numpy)` · `SQL` · `Power BI (DAX)`

## Skills Demonstrated
Multi-source data integration and schema reconciliation · anomaly detection and transparent data-quality reporting (rather than silently masking bad data) · surrogate key design · time-based feature engineering · translating a messy real-world data problem into a governance recommendation

## Repo Files
`python_etl_hospital.py` · `sql_hospital_analysis.sql` · `master_hospital_data.csv` · `hospital_summary_kpis.csv` · `Hospital_Dashboard_PowerBI_Guide.md` · `Project_Report.md`

---
