# Hospital_Data_Analysis
Description

Simulates a common real-world hospital problem: patient data scattered across three disconnected systems — Billing (1,300 rows), Visit Log (5,000 rows), and Insurance (10,000 rows) — with different schemas, ID formats, and inconsistent fields. This project builds a unified, single source-of-truth master dataset (16,198 records) via a Python ETL pipeline, standardizes fields (Gender labels, prefixed IDs, common schema), and — notably — surfaces rather than hides a real data quality issue found in the source (89% of Insurance System records had implausible 700+ day "stay lengths"). The cleaned data feeds a Power BI dashboard covering patient volume, revenue, department/diagnosis cost drivers, insurance-outcome relationships, and admission seasonality.

Key Skills Demonstrated
Multi-source data integration and schema standardization
Data quality auditing and transparent reporting of anomalies (rather than silently averaging them away)
Python ETL, SQL analysis layer, Power BI dashboarding
Feature engineering (AgeGroup, AdmissionMonth/DayOfWeek/Hour)
