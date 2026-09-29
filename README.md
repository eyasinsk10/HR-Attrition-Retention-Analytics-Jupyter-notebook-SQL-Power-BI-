# HR-Attrition-Retention-Analytics-Jupyter-notebook-SQL-Power-BI-
# HR Analytics: Employee Attrition & Retention Dashboard

An end-to-end HR analytics project that takes a raw Kaggle dataset through **Python cleaning → MySQL storage & analysis → Power BI dashboarding**, with **Excel** used for cross-checking, to answer one question: **why are 51% of this company's employees leaving, and what can HR do about it?**


## Business Problem

Employee turnover is one of the costliest problems an HR team faces. In this dataset, **1,533 of 3,000 employees (51.1%) have left** the company between 2018 and August 2023. Annualized attrition climbed from **11.9% in 2019 to 27.9% in 2022**, and in 2023 the company lost 596 people while hiring only 335 — headcount is now shrinking.

This project was built to answer the questions an HR Head / CHRO actually asks:

- How bad is attrition, and is it getting worse?
- Which departments, tenure groups and employee types are leaving fastest?
- Are we losing good performers, and is training budget being wasted on people who leave anyway?
- Is attrition fair across gender, race and age groups, or concentrated somewhere?

---

## Project Workflow

Instead of loading the CSV straight into Power BI, this project follows a full analytics pipeline, end to end:

```
Kaggle (raw CSV)
      │
      ▼
Python + Pandas in Jupyter Notebook   → data cleaning, type fixes, null handling
      │
      ▼
MySQL                                  → cleaned data loaded into a database
      │
      ▼
SQL                                    → business questions answered with queries
      │
      ▼
Power BI (Import from MySQL)           → data model, DAX measures, 4-page dashboard
      │
      ▼
Excel                                  → spot-checked KPI totals against SQL/Power BI
```

**1. Data cleaning (Python / Pandas, Jupyter Notebook)**
Raw HR data from Kaggle was cleaned and standardized — fixing data types (dates, numeric fields), handling missing values, and preparing a load-ready table before it went anywhere near a database or dashboard.

**2. Database load (MySQL)**
The cleaned dataset was loaded into MySQL, turning a flat file into a queryable table that supports the same kind of ad-hoc business questions a real HR analytics stack would face.

**3. Business analysis (SQL)**
Key business questions were answered directly in SQL before any visualization work — for example:

```sql
-- Overall attrition rate
SELECT
    ROUND(SUM(CASE WHEN IsActive = 'Terminated' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 1) AS attrition_rate_pct
FROM hr_human;

-- Attrition rate by department
SELECT
    DepartmentType,
    COUNT(*) AS headcount,
    SUM(CASE WHEN IsActive = 'Terminated' THEN 1 ELSE 0 END) AS exits,
    ROUND(SUM(CASE WHEN IsActive = 'Terminated' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 1) AS attrition_rate_pct
FROM hr_human
GROUP BY DepartmentType
ORDER BY attrition_rate_pct DESC;

-- Attrition rate by tenure bucket
SELECT
    TenureBucket,
    COUNT(*) AS headcount,
    ROUND(SUM(CASE WHEN IsActive = 'Terminated' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 1) AS attrition_rate_pct
FROM hr_human
GROUP BY TenureBucket;

-- Training cost lost to employees who exited
SELECT
    ROUND(SUM(CASE WHEN IsActive = 'Terminated' THEN `Training Cost` ELSE 0 END), 0) AS training_cost_on_exits,
    ROUND(SUM(`Training Cost`), 0) AS total_training_cost
FROM hr_human;
```

**4. Dashboarding (Power BI)**
The MySQL table was imported directly into Power BI (Get Data → MySQL database), where a `DimDate` calendar table and DAX measures were built to support year-based slicing across a 4-page report.

**5. Cross-checking (Excel)**
Final KPI totals (headcount, exits, attrition rate, training cost) were independently recalculated in Excel using PivotTables and formulas, and matched against both the SQL output and the Power BI cards before publishing.

---

## Dataset

| | |
|---|---|
| **Source** | Kaggle (HR employee dataset) |
| **Size** | 3,000 employees, 42 columns |
| **Period** | Hires and exits from Aug 2018 to Aug 2023 |
| **Covers** | Demographics, department / business unit, tenure, termination type, performance score, engagement / satisfaction / work-life balance survey scores, training program and cost |
| **Attrition flag** | `IsActive` (Active = 1,467, Terminated = 1,533), consistent with `ExitDate` and `TerminationType` |

**Data preparation notes**
- `IsActive` is the single source of truth for who has left.
- 144 employees have a negative tenure (start date after survey date); excluded from tenure averages and shown as "Unknown" in tenure buckets.
- "Attrition Rate %" on the Overview page is **cumulative** (all exits ÷ all employees ever). Yearly trends use an **annualized** rate (exits in year ÷ average headcount for that year).

---

## Tools Used

- **Python (Pandas, Jupyter Notebook)** — data cleaning and preparation
- **MySQL** — data storage and SQL-based business analysis
- **Power BI Desktop + DAX** — data modeling and dashboard build
- **Excel** — KPI cross-checking and validation

---

## Dashboard Pages

1. **Executive Overview** — headcount, exits, attrition rate, hires vs. exits by year
2. **Attrition Deep-Dive** — attrition by department, business unit, tenure bucket, termination type, and month
3. **Demographics & Diversity** — gender, race, age and marital mix, with attrition rate by group
4. **Performance, Engagement & Training** — performance ratings, survey scores, training outcomes and cost

screenshot(Dashboard):::
 <img width="1156" height="632" alt="Executive Overview" src="https://github.com/user-attachments/assets/8c632b0a-184f-4f1d-a1f7-5cf52dcfa534" />

  <img width="1152" height="626" alt="Attrition Deep-Dive" src="https://github.com/user-attachments/assets/34c7f11a-90fc-46d1-ba52-eb83b6aadf96" />

  <img width="1157" height="630" alt="Demographics   Diversity" src="https://github.com/user-attachments/assets/67e824e1-f0ed-4c42-ad3b-e98f1c20e0cb" />

  <img width="1155" height="630" alt="Performance,Engagement   Training" src="https://github.com/user-attachments/assets/f2204b44-9841-4a62-8394-188d686ee5f3" />



## KPI Summary

| KPI | Value | DAX (Power BI) |
|---|---|---|
| Total Employees (all records) | 3,000 | `COUNTROWS('hr human')` |
| Current Headcount | 1,467 | `CALCULATE(COUNTROWS('hr human'), 'hr human'[IsActive] = "Active")` |
| Total Exits | 1,533 | `CALCULATE(COUNTROWS('hr human'), 'hr human'[IsActive] = "Terminated")` |
| Cumulative Attrition Rate | 51.1% | `DIVIDE([Total Exits], [Total Employees])` |
| Annualized Attrition (2022) | 27.9% | Exits in year ÷ average of opening & closing headcount |
| Avg Tenure at Exit | 1.34 yrs | `CALCULATE(AVERAGE('hr human'[TenureYears]), 'hr human'[IsActive] = "Terminated", 'hr human'[TenureYears] >= 0)` |
| Voluntary Exit Share | 50.1% | `DIVIDE(CALCULATE([Total Exits], 'hr human'[TerminationType] IN {"Voluntary","Resignation"}), [Total Exits])` |
| Female Share of Workforce | 56.1% | `DIVIDE(CALCULATE(COUNTROWS('hr human'), 'hr human'[GenderCode] = "Female"), COUNTROWS('hr human'))` |
| Avg Engagement Score | 2.94 / 5 | `AVERAGE('hr human'[Engagement Score])` |
| Total Training Cost | $1.68M | `SUM('hr human'[Training Cost])` |

---

## Key Insights

1. **New hires leave fastest.** Attrition is 69.6% for employees with under 1 year of tenure, 53.1% for 1–3 years and 26.1% for 3–5 years. Nearly half of all exits (47.6%) happen in the first year.
2. **Attrition is accelerating and hiring is not keeping up.** Annualized attrition rose from 11.9% (2019) to 27.9% (2022). In 2023, exits (596) far exceeded hires (335).
3. **Production drives the volume.** It holds 67% of employees and accounts for 66% of all exits (1,014). Small teams show the highest rates: Executive Office 79.2%, Admin Offices 60.0%, Software Engineering 55.7%.
4. **Half of exits are voluntary.** Voluntary + Resignation make up 50.1% of exits, Involuntary 25.3%, Retirement 24.6%.
5. **High performers are not staying.** 51.8% of "Exceeds" employees have left — no better than the 50.9% rate for "Fully Meets".
6. **Training money is being lost.** $857K (51%) of the $1.68M training spend went to employees who later left. Attrition is 54.3% after failed training vs. 48.7% after passed training.
7. **Attrition is fair across demographics.** Gender differs by only 0.7 points (51.4% F vs 50.7% M); race groups differ by about 3 points. Temporary staff leave more (53.4%) than full-time staff (49.8%).
8. **Survey scores don't explain exits.** Engagement is 2.94 for both leavers and stayers; work-life balance is only slightly lower for leavers (2.95 vs 3.03) — current surveys are a weak early-warning signal.

---

## Business Recommendations

| Insight | Recommended action |
|---|---|
| New hires leave in year 1 | Build a 90-day onboarding and buddy program with check-ins at 30, 60 and 90 days |
| Attrition rising, hiring falling | Set a quarterly attrition target and pause volume hiring until early-tenure retention improves |
| Production drives volume | Run stay and exit interviews in Production first; review shift patterns and supervisor quality |
| High rates in small teams | Review Executive Office and Admin Offices individually (succession planning, workload) |
| Half of exits are voluntary | Add 6- and 12-month stay interviews and a documented career-path conversation |
| High performers leaving | Set up a retention review for "Exceeds" employees (growth path, recognition, pay check) |
| Training spend lost | Train after probation, fix failing programs, track cost per retained employee |
| Temporary staff leave more | Offer conversion-to-permanent paths for high-rated temporary staff |
| Surveys don't predict exits | Move to short quarterly pulse surveys on manager, growth and pay |

---

## About Me

Built by **SK Eyasin Ali** — B.Tech in Electronics & Communication Engineering (2026), self-taught in data analytics (Python, SQL, Power BI, Excel). Open to Data Analyst / Business Analyst fresher roles.

[LinkedIn]([#](https://www.linkedin.com/in/sk-eyasin-ali-638378323/)) · [Email](eyasinsk887@gmail.com)
