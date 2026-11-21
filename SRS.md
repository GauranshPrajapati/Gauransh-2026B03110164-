# Software Requirements Specification (SRS)
## Project: Bike Rental Demand Analysis & Fleet Optimization System

**Version:** 1.0 (Final) | **Date:** November 20, 2026 | **Candidate:** Gauransh Prajapati (ID: 2026B03110164)  
**Dataset:** `Bike_Rental_Project extended.csv` (1,000 records, 9 fields) | **Repository:** [https://github.com/GauranshPrajapati/HCL-Project-by-Gauransh.git](https://github.com/GauranshPrajapati/HCL-Project-by-Gauransh.git)

---

## Table of Contents
1. [Introduction](#1-introduction)
2. [General Description](#2-general-description)
3. [Functional Requirements](#3-functional-requirements)
4. [Interface Requirements](#4-interface-requirements)
5. [Performance Requirements](#5-performance-requirements)
6. [Design Constraints](#6-design-constraints)
7. [Non-Functional Attributes](#7-non-functional-attributes)
8. [Preliminary Schedule and Budget](#8-preliminary-schedule-and-budget)
9. [Appendices](#9-appendices)
10. [Uses of SRS Document](#10-uses-of-srs-document)
11. [FAQs on SRS Format](#11-faqs-on-srs-format)
12. [Conclusion](#12-conclusion)

---

## 1. Introduction

### 1.1 Purpose
This document specifies the requirements for a micro-mobility data analytics system that processes historical bike-sharing transaction records. The system assists municipal transport authorities and private bike-share operators in forecasting rental demand across temporal cycles, seasonal transitions, and fluctuating weather conditions. The findings directly inform operational fleet planning, truck-based station rebalancing schedules, and maintenance staffing. This specification is written for the data analyst implementing the pipeline, operations managers utilizing the deliverables, and academic evaluators assessing project rigor.

### 1.2 Scope
The system ingests a transaction-level CSV file (1,000 observations), executes missing-value detection and median-based imputation, extracts and verifies calendar and hourly attributes, performs multi-factor aggregations (season, day-of-week, weather, hour), applies the Interquartile Range (IQR) technique for anomaly detection, conducts calendar consistency audits, and exports cleaned tabular deliverables. The software functions as an analytical decision-support engine; it does not process live GPS streams, execute financial transactions, or control physical dock locking mechanisms.

### 1.3 Definitions, Acronyms and Abbreviations

| Term | Meaning |
| :--- | :--- |
| **SRS** | Software Requirements Specification |
| **IQR** | Interquartile Range ($Q_3 - Q_1$), non-parametric measure of statistical dispersion used to isolate outliers |
| **Median Imputation** | Replacing missing data points with the median value of the attribute to avoid outlier distortion |
| **Fleet Rebalancing** | Logistical process of transporting bicycles from saturated docking hubs to empty stations |
| **Diurnal Pattern** | Demand variability observed across distinct operating hours within a 24-hour cycle |
| **Pearson r** | Bivariate correlation coefficient measuring linear dependency between two continuous variables |
| **Ground Truth** | Calendar calculations based on standard Gregorian calendar logic (e.g. day-of-week indexing) |

### 1.4 References
- **Project brief:** Project 3, Bike Rental Demand Analysis (Mobility / Smart City Domain).
- **Primary analysis notebook:** `Gauransh(2026B03110164).ipynb`.
- **Cleaned dataset:** `Bike_Rental_Project extended.csv` (1,000 records, 9 fields).
- **Repository link:** [https://github.com/GauranshPrajapati/HCL-Project-by-Gauransh.git](https://github.com/GauranshPrajapati/HCL-Project-by-Gauransh.git)

### 1.5 Overview
Section 2 provides an overview of the system context, product functions, and target user profiles. Sections 3 through 7 define functional requirements, system interfaces, performance expectations, design constraints, and quality attributes. Section 8 details project scheduling and cost estimates. Sections 9 through 12 present analytical appendices, professional use-cases for this SRS, frequently asked questions, and a concluding evaluation.

---

## 2. General Description

### 2.1 Product Perspective
The system operates as an offline, batch-oriented data analytics pipeline. It ingests historical micro-mobility records, executes analytical transformations in memory, and writes structured summary tables to disk. While standalone, its generated datasets directly integrate with fleet operations dashboards and logistical rebalancing routes.

### 2.2 Product Functions (Summary)
- **Ingestion & Type Auditing:** Load the CSV dataset and inspect dimensional shape and attribute data types.
- **Missing Value Imputation:** Report missing values and impute gaps in `Humidity_Percent` using median values.
- **Datetime Decomposition:** Parse `DateTime` into native datetime objects, extracting `Date`, `Month`, and `Hour`.
- **Chronological Consistency Check:** Assert equivalence between parsed timestamp hour and pre-existing `Hour` column.
- **Utilization Benchmarking:** Compute total cumulative rentals and average rental velocity per record.
- **Peak Demand Identification:** Isolate the top 10 highest demand periods using `nlargest`.
- **Seasonal Comparative Analysis:** Segment demand across Spring, Summer, Autumn, and Winter.
- **Weekday vs. Weekend Segmentation:** Evaluate differences in utilization between business days and weekends.
- **Meteorological Sensitivity:** Measure demand variations across Clear, Cloudy, Rain, and Heavy Rain conditions.
- **Diurnal Profile Analysis:** Aggregate rental volume by operating hour (mean, sum, count).
- **Vectorized Array Math:** Compute arithmetic averages of temperature and rental count using NumPy arrays.
- **Statistical Anomaly Detection:** Determine upper and lower outlier fences using the 1.5 $\times$ IQR rule.
- **Factorial Cross-Tabulation:** Construct a 2D Season $\times$ Weather pivot matrix.
- **Daily Demand Extremes:** Aggregate demand by calendar date to isolate the top 5 busiest and slowest days.
- **Linear Correlation Testing:** Calculate Pearson correlation coefficient between temperature and rental volume.
- **Data Quality Auditing:** Verify synthetic `Is_Weekend` flags against real calendar ground truth.
- **Deliverable Generation:** Serialize cleaned dataset and 3 summary tables to disk.

### 2.3 User Characteristics

| User Role | Operational Needs |
| :--- | :--- |
| **Data Analyst** | Executes data pipeline, validates data hygiene, inspects distributions, and extends analytical scripts. |
| **Fleet Logistics Manager** | Reviews peak hours, seasonal variations, and weather thresholds to schedule rebalancing vans. |
| **City Transport Planner** | Examines macro utilization trends to allocate municipal cycling infrastructure funding. |
| **Academic Reviewer / Mentor** | Verifies completeness, statistical validity, and compliance with assigned project tasks. |

### 2.4 General Constraints
- The dataset reflects two operational snapshots per day (Hour 0 midnight and Hour 12 noon).
- The analysis relies strictly on historical observations without real-time API connectivity.
- All computations must run within standard Jupyter/Colab runtime constraints.

### 2.5 Assumptions and Dependencies
- Each row represents a valid telemetry reading uniquely identified by `Record_ID`.
- Missing humidity values are Missing Completely at Random (MCAR) and safely imputable via median.
- Python 3.8+ runtime with pandas and NumPy libraries is available in the operating environment.

---

## 3. Functional Requirements

Each functional requirement corresponds to a verifiable project task and deliverables milestone:

| ID | Requirement Statement | Task |
| :--- | :--- | :--- |
| **FR-1** | The system shall load `Bike_Rental_Project extended.csv` and display its dimensional shape, head rows, and column data types. | Task 1 |
| **FR-2** | The system shall report the exact count of missing values per column across the dataset. | Task 2 |
| **FR-3** | The system shall impute missing `Humidity_Percent` values using the attribute median (58.0%) and verify zero remaining nulls. | Task 2 |
| **FR-4** | The system shall convert `DateTime` to pandas datetime using format `'%d-%m-%Y %H:%M'` and derive `Date`, `Month`, and `Hour_Check`. | Task 3 |
| **FR-5** | The system shall verify that `Hour_Check` matches the pre-existing `Hour` column and drop the redundant verification column. | Task 3 |
| **FR-6** | The system shall calculate total cumulative rentals (34,712) and mean rentals per record (34.71). | Task 4 |
| **FR-7** | The system shall isolate and display the top 10 highest-demand records using `.nlargest(10, 'Rental_Count')`. | Task 5 |
| **FR-8** | The system shall compute average rentals by Season and identify the peak season (Spring: 35.63) and quietest season (Summer: 33.79). | Task 6 |
| **FR-9** | The system shall compute average rental demand segmented by the `Is_Weekend` binary indicator. | Task 7 |
| **FR-10** | The system shall compute average rental demand across Weather conditions (Clear, Rain, Cloudy, Heavy Rain). | Task 8 |
| **FR-11** | The system shall compute multi-metric aggregations (mean, sum, count) grouped by operating Hour (Hour 0 vs. Hour 12). | Task 9 |
| **FR-12** | The system shall use NumPy ndarray conversions to compute the arithmetic mean of `Temperature_C` (23.90°C) and `Rental_Count` (34.71). | Task 10 |
| **FR-13** | The system shall compute Q1 (31.0), Q3 (39.0), IQR (8.0), and isolate records exceeding the upper fence (Upper = 51.0). | Task 11 |
| **FR-14** | The system shall construct a 2D Season by Weather pivot table calculating mean rentals per category combination. | Task 12 |
| **FR-15** | The system shall aggregate rentals by calendar Date and report the top 5 busiest operating days and top 5 slowest days. | Task 13 |
| **FR-16** | The system shall calculate the Pearson correlation coefficient between `Temperature_C` and `Rental_Count` ($r = -0.012$). | Task 14 |
| **FR-17** | The system shall audit `Is_Weekend` against real calendar day-of-week and count unique calendar months per season label. | Extra |
| **FR-18** | The system shall produce actionable operational recommendations backed by empirical findings. | Recommendations |
| **FR-19** | The system shall export `cleaned_bike_rental.csv`, `hourly_demand_summary.csv`, `seasonal_comparison.csv`, and `weather_comparison.csv`. | Deliverables |

### 3.1 Business Rules
- **Imputation Rule:** Median is mandatory over mean to prevent bias from skewed sensor readings.
- **Anomaly Rule:** Standard Tukey multiplier of 1.5 $\times$ IQR defines the outlier threshold.
- **Idempotency Rule:** Executing data cleaning scripts multiple times must yield identical data state.

---

## 4. Interface Requirements

### 4.1 User Interface
The primary user interface is the interactive Jupyter Notebook `Gauransh(2026B03110164).ipynb`. Each analytical task is separated into distinct sections featuring Markdown explanatory headers, code cells, and inline tabular stdout displays.

### 4.2 Input Interface

| Field Name | Type | Constraint / Domain | Description |
| :--- | :--- | :--- | :--- |
| `Record_ID` | Integer | Unique, > 0 | Sequential transaction identifier |
| `DateTime` | String | DD-MM-YYYY HH:MM | Timestamp of observation |
| `Season` | Text | Spring, Summer, Autumn, Winter | Meteorological season label |
| `Weather` | Text | Clear, Cloudy, Rain, Heavy Rain | Atmospheric weather category |
| `Temperature_C` | Decimal | 2.6 to 43.6 °C | Ambient temperature measurement |
| `Humidity_Percent` | Decimal | 25.0 to 95.0 % (11 nulls) | Relative humidity percentage |
| `Is_Weekend` | Text | 'Yes', 'No' | Weekend indicator flag |
| `Hour` | Integer | 0, 12 | Operational measurement hour |
| `Rental_Count` | Integer | 18 to 52 | Number of bicycles rented |

### 4.3 Output Interface
- Formatted console tables printed during script execution.
- Four exported CSV deliverables for downstream operational use.
- Formal Software Requirements Specification (SRS) document.

### 4.4 Software Interface
Python 3.8+ interpreter, pandas library for relational aggregations, and NumPy for array operations.

### 4.5 Hardware Interface
Any standard laptop or desktop computer with at least 4 GB RAM and 200 MB free disk space.

---

## 5. Performance Requirements
- **PR-1:** The complete 16-task pipeline shall execute in under 3.0 seconds on standard consumer hardware.
- **PR-2:** The system shall support datasets up to 500,000 records in under 30 seconds due to fully vectorized operations.
- **PR-3:** Memory utilization shall remain below 200 MB throughout runtime execution.
- **PR-4:** Pipeline execution shall be deterministic, producing bit-exact identical metrics on repeated runs.

---

## 6. Design Constraints
- **Language and Libraries:** Python 3 with pandas and NumPy.
- **Coding Standards:** Compliance with PEP 8 conventions, clean human-readable comments, and modern f-strings.
- **Data Privacy:** The dataset contains aggregate station numbers and no personally identifiable customer information (PII).
- **Cross-Platform Portability:** File path logic supports both local environments and Google Colab (`/content/`) directories.

---

## 7. Non-Functional Attributes

| Attribute | Specification |
| :--- | :--- |
| **Correctness** | Mathematical verification of all totals, means, percentiles, IQR boundaries, and correlation coefficients. |
| **Reliability** | Robust handling of missing values with median imputation; assertion checks for timestamp consistency. |
| **Usability** | Clear modular notebook layout with separate Markdown headings and dedicated outputs per task. |
| **Maintainability** | Clean, un-obfuscated Python code structured logically by numbered task IDs. |
| **Portability** | Seamlessly runs on Windows, macOS, Ubuntu, and Google Colaboratory. |
| **Transparency** | Every data cleaning decision, median replacement value, and calendar audit discrepancy is documented. |

---

## 8. Preliminary Schedule and Budget

### 8.1 Schedule

| Phase | Duration | Deliverable Output |
| :--- | :--- | :--- |
| **Phase 1: Requirements & Data Ingestion** | 1 day | SRS document draft, schema inspection |
| **Phase 2: Data Cleaning & Feature Extraction** | 1 day | Median imputation, datetime conversion |
| **Phase 3: Macro & Group Demand Analysis** | 2 days | Seasonal, weather, and day-type demand tables |
| **Phase 4: Statistical Modeling & Anomaly Detection** | 1 day | IQR outlier detection, pivot tables, correlation |
| **Phase 5: Validation, Deliverables & Final Review** | 1 day | Quality audit, 4 CSV files, final report |
| **Total Estimated Effort** | about 6 working days | Completed analytical solution |

### 8.2 Budget

| Expense Item | Estimated Cost |
| :--- | :--- |
| **Analyst Effort (~6 working days)** | Primary labor cost (academic / internal project) |
| **Software Stack (Python, pandas, NumPy, Jupyter)** | Free and open-source ($0.00) |
| **Compute Hardware (Standard PC / Colab free-tier)** | Existing infrastructure ($0.00) |
| **Contingency Buffer** | 10-15% of estimated effort |

---

## 9. Appendices

### Appendix A: Key Results from the Dataset

| Metric / Parameter | Observed Empirical Value |
| :--- | :--- |
| **Total Records Processed** | 1,000 observations |
| **Missing Humidity Values Imputed** | 11 records (imputed with median: 58.0%) |
| **Total Cumulative Rentals** | 34,712 units |
| **Average Demand per Observation** | 34.71 units / record |
| **Peak Demand Season** | Spring (35.63 average rentals) |
| **Lowest Demand Season** | Summer (33.79 average rentals) |
| **Weekend vs. Weekday Average** | Weekend: 35.01 vs. Weekday: 34.43 |
| **Highest Demand Weather Condition** | Clear (35.24 average rentals) |
| **Lowest Demand Weather Condition** | Heavy Rain (33.83 average rentals) |
| **Hourly Demand Distribution** | Hour 0: 34.40 avg \| Hour 12: 35.03 avg |
| **Mean Temperature (NumPy)** | 23.90 °C |
| **Statistical Outliers (IQR Method)** | 2 records identified (demand = 52, above upper fence of 51.0) |
| **Busiest Calendar Day** | 2026-01-29 (93 total rentals) |
| **Slowest Calendar Day** | 2026-02-25 (48 total rentals) |
| **Temperature-Demand Correlation** | $r = -0.012$ (negligible linear dependence) |
| **Calendar Ground-Truth Agreement** | 50.7% match rate |

### Appendix B: Requirement Traceability Matrix
Every functional requirement FR-1 through FR-17 maps directly to the assigned 14 core project tasks plus extra quality checks and deliverable exports.

### Appendix C: Operational Recommendations
- **Spring Peak Fleet Allocation:** Increase active bike fleet and maintenance staff by 10-15% during Spring months to accommodate peak demand.
- **Midday Fleet Rebalancing:** Prioritize logistics transit vans between 10:00 AM and 1:00 PM to rebalance stations ahead of peak afternoon demand.
- **Adverse Weather Logistics:** Schedule routine fleet maintenance and workshop repairs during Heavy Rain periods when demand drops to lowest levels.
- **Outlier Surge Preparedness:** Maintain an on-demand buffer reserve of 50-70 bikes at major transit hubs to handle extreme surges above 50 rentals.
- **High-Frequency Telemetry:** Expand sensor sampling from two readings per day (Hour 0 and 12) to continuous 24-hour hourly telemetry.

---

## 10. Uses of SRS Document
- **Shared Understanding:** Establishes a formal baseline between analysts, fleet planners, and project mentors.
- **Development Blueprint:** Guides implementation directly from numbered requirements rather than ambiguous goals.
- **Verification Baseline:** Provides objective criteria to test data accuracy, imputation validity, and aggregation correctness.
- **Change Control:** Evaluates future enhancement requests (e.g. predictive machine learning models) against existing scope.

---

## 11. FAQs on SRS Format

**Q: What is the primary purpose of an SRS in a data analytics project?**  
A: It formally defines the expected data transformations, business rules, statistical boundaries, and deliverables before and during code implementation.

**Q: What distinguishes functional requirements from non-functional attributes?**  
A: Functional requirements specify what the system calculates (e.g. IQR outliers, pivot tables). Non-functional attributes specify how well it performs (e.g. execution speed, memory footprint, reproducibility).

**Q: Why are schedule and budget sections included in an academic/analyst SRS?**  
A: They provide realistic project planning estimates, establish milestone tracking, and align engineering effort with project deliverables.

---

## 12. Conclusion
This Software Requirements Specification defines the operational purpose, analytical scope, data schemas, functional requirements, and performance targets for the Bike Rental Demand Analysis System. By translating the 14 project tasks into testable requirements and documenting empirical baseline findings, this document ensures the delivered analysis is mathematically sound, reproducible, and operationally valuable. The resulting pipeline delivers cleaned datasets, summary tables, and evidence-based recommendations to help bike-share operators plan fleet availability effectively.
