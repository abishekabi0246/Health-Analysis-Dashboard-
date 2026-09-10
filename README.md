# Healthcare Patient & Hospital Performance Analytics Dashboard

A beginner-to-intermediate Data Analyst portfolio project covering the full
pipeline: synthetic data generation → Python cleaning → SQL analysis →
Power BI star-schema modeling → DAX measures → a 3-page executive dashboard.

> **Data note:** All data in this project is 100% synthetic and randomly
> generated for portfolio purposes. There are no real patients, no real
> hospitals, and no real medical record numbers involved anywhere.

---

## 1. Project Files

| File | Purpose |
|---|---|
| `generate_raw_data.py` | Generates the synthetic "dirty" raw dataset (1,225 rows) |
| `healthcare_data_raw.csv` | Raw synthetic data, intentionally messy |
| `data_cleaning.py` | Pandas script that cleans the raw data step-by-step |
| `healthcare_data_cleaned.csv` | Final cleaned dataset (1,181 rows) — load this into Power BI/SQL |
| `database_setup.sql` | MySQL `CREATE TABLE`, sample `INSERT`s, and 20 analysis queries |
| `dax_measures.txt` | All DAX measures for the Power BI model |
| `README.md` | This file — full project write-up |

---

## 2. Dataset Overview

18 columns, 1,181 clean rows after cleaning (started from 1,225 raw rows):

`Patient_ID, Age, Gender, Department, Admission_Type, Admission_Date,
Discharge_Date, Length_of_Stay, Diagnosis_Category, Treatment_Type,
Doctor_Experience, Waiting_Time_Minutes, Treatment_Cost, Insurance_Type,
Payment_Status, Readmission, Patient_Satisfaction, Emergency_Case`

The raw data was deliberately generated with realistic messiness — missing
values, duplicate rows, text stored where numbers belong ("333.0 USD"),
invalid dates ("0000-00-00"), negative costs, out-of-range satisfaction
scores, and inconsistent text casing — so the cleaning step in
`data_cleaning.py` has genuine problems to solve (see that file's comments
for the full step-by-step explanation).

---

## 3. Power BI Data Model (Star Schema)

**Import:** In Power BI Desktop, use *Get Data → Text/CSV* and select
`healthcare_data_cleaned.csv`. Use Power Query to confirm each column's
data type (Date columns as Date, cost as Decimal Number, IDs as Text).

**Recommended star schema:**

```
                    ┌─────────────────────┐
                    │     Dim_Date         │
                    │  Date (PK)           │
                    │  Year, Month,        │
                    │  MonthName, Quarter  │
                    └──────────┬──────────┘
                               │
┌───────────────┐    ┌─────────▼──────────┐    ┌───────────────────┐
│ Dim_Department │◄───┤   Fact_Visits       ├───►│ Dim_Diagnosis      │
│ Department (PK)│    │  Patient_ID          │    │ Diagnosis_Cat (PK) │
└───────────────┘    │  Age, Gender         │    └───────────────────┘
                      │  Department (FK)     │
┌───────────────┐    │  Admission_Date (FK) ├───►┌───────────────────┐
│ Dim_Insurance  │◄───┤  Discharge_Date      │    │ Dim_Treatment      │
│ Insurance (PK) │    │  Length_of_Stay      │    │ Treatment_Type(PK) │
└───────────────┘    │  Diagnosis (FK)       │    └───────────────────┘
                      │  Treatment_Type (FK)  │
                      │  Waiting_Time_Minutes │
                      │  Treatment_Cost       │
                      │  Insurance_Type (FK)  │
                      │  Payment_Status       │
                      │  Readmission          │
                      │  Patient_Satisfaction │
                      │  Emergency_Case       │
                      └───────────────────────┘
```

- **Fact table (`Fact_Visits`):** one row per hospital visit — contains all
  the measurable numeric values (cost, waiting time, length of stay,
  satisfaction) plus foreign keys pointing to the dimension tables.
- **Dimension tables:** small lookup tables of unique values
  (`Dim_Department`, `Dim_Diagnosis`, `Dim_Treatment`, `Dim_Insurance`) that
  describe *who/what/where* — they make slicers cleaner and let you add
  extra descriptive attributes later without bloating the fact table.
- **`Dim_Date` (the date table):** Create with
  `Dim_Date = CALENDAR(MIN(Fact_Visits[Admission_Date]), MAX(Fact_Visits[Admission_Date]))`
  then add calculated columns for `Year`, `Month`, `MonthName`, `Quarter`.
  Mark it as a Date Table (Modeling → Mark as Date Table). This is what
  makes month/quarter/year slicers and time-intelligence DAX work
  correctly.
- **Relationships:** all dimension tables connect to `Fact_Visits` as
  **one-to-many** (dimension side = "1", fact side = "many"), with a
  **single direction** filter flowing from dimension → fact. For a
  beginner project, keeping the raw table as one flat table with the
  dimension columns still present, and only building `Dim_Date` and
  `Dim_Department` separately, is also perfectly fine — expand the star
  schema as you get more comfortable.

---

## 4. DAX Measures

See `dax_measures.txt` for the full list with plain-English explanations:
Total Patients, Average Age, Total Treatment Cost, Average Treatment Cost,
Average Waiting Time, Average Length of Stay, Readmission Count,
Readmission Rate, Emergency Cases, Average Satisfaction, Total Revenue,
Monthly Patients, and the four components behind the Hospital Performance
Score.

---

## 5. Dashboard Design (3 Pages) — Text Wireframe

### Page 1 — "Hospital Executive Overview"

```
┌──────────────────────────────────────────────────────────────────┐
│  HOSPITAL EXECUTIVE OVERVIEW           [Date] [Dept] [Gender]     │
│                                          [Admission Type] [Ins.]  │
├───────────┬───────────┬───────────┬───────────┬──────────────────┤
│  Total    │  Total    │  Avg      │  Avg      │  Readmission     │
│  Patients │  Cost     │  Wait     │  Satisfac.│  Rate            │
│  1,181    │  $1.46M   │  30 min   │  3.72/5   │  14.1%           │
├───────────┴───────────┴───────────┴───────────┴──────────────────┤
│  Monthly Patient Trend (line chart)                               │
│  ▁▃▅▇▆▄▃▅▇▆▄▃▅▇▆▄▃▅▇▆▄▃▅▇                                          │
├───────────────────────┬────────────────────────────────────────┤
│ Patients by Department │  Treatment Cost by Department           │
│ (bar chart)            │  (bar chart)                            │
├───────────────────────┼────────────────────────────────────────┤
│ Admission Type Split   │  Satisfaction by Department             │
│ (donut chart)          │  (bar chart)                            │
└───────────────────────┴────────────────────────────────────────┘
```

### Page 2 — "Patient & Department Analysis"

```
┌──────────────────────────────────────────────────────────────────┐
│  PATIENT & DEPARTMENT ANALYSIS         [same slicers as Page 1]  │
├───────────────────────┬────────────────────────────────────────┤
│ Age Distribution        │ Diagnosis Distribution                 │
│ (histogram)             │ (bar/treemap)                          │
├───────────────────────┼────────────────────────────────────────┤
│ Avg Waiting Time by Dept│ Avg Length of Stay by Dept              │
│ (bar chart)             │ (bar chart)                             │
├───────────────────────┼────────────────────────────────────────┤
│ Treatment Type Analysis │ Emergency vs Non-Emergency              │
│ (bar/pie)               │ (donut chart)                           │
├─────────────────────────┴───────────────────────────────────────┤
│  Readmission Rate by Department (bar chart)                       │
└──────────────────────────────────────────────────────────────────┘
```

### Page 3 — "Hospital Performance & Insights"

```
┌──────────────────────────────────────────────────────────────────┐
│  HOSPITAL PERFORMANCE & INSIGHTS        [same slicers]           │
├───────────┬───────────┬───────────┬──────────────────────────────┤
│ Best Dept │ Highest   │ Highest   │ Lowest Satisfaction Dept      │
│ (KPI)     │ Wait Dept │ Cost Dept │ (KPI)                         │
├───────────┴───────────┴───────────┴──────────────────────────────┤
│  Hospital Performance Score gauge (0-100, colored by category)   │
├─────────────────────────┬──────────────────────────────────────┤
│ Readmission Analysis     │  Key Recommendations                  │
│ (table by dept)          │  (text box, bullet list)              │
└─────────────────────────┴──────────────────────────────────────┘
```

**Slicers (on every page, synced):** Date, Department, Gender,
Admission Type, Insurance Type, Diagnosis.

---

## 6. Hospital Performance Score (Innovative Feature)

> ⚠️ **This is a project-defined analytical score created for this
> portfolio project. It is not an official medical, clinical, or
> regulatory hospital-rating standard.**

The score (0–100) blends four operational signals, each rescaled to a
0–100 basis so they can be combined fairly, then weighted:

| Component | Weight | Logic |
|---|---|---|
| Patient Satisfaction | 30% | `(avg satisfaction / 5) × 100` |
| Waiting Time | 25% | 100 at 0 min, scaling down to 0 at 120+ min |
| Readmission Rate | 25% | `100 − (readmission rate × 100)` |
| Length of Stay | 20% | 100 at 0 days, scaling down to 0 at 10+ days |

**Classification:**
- 80–100 = Excellent
- 60–79 = Good
- 40–59 = Average
- Below 40 = Needs Improvement

Full DAX for each component is in `dax_measures.txt`. Feel free to adjust
the weights and thresholds — being able to justify *why* you chose them is
a great talking point in interviews.

---

## 7. Insights (based on the actual cleaned dataset)

1. The hospital processed **1,181 visits** with total treatment revenue of
   **$1.46M**, averaging **$1,238 per visit**.
2. **General** is by far the most common diagnosis category (525 of 1,181
   visits, ~44%), well ahead of Respiratory (192) and Fracture (152).
3. Departments are fairly evenly loaded: Neurology sees the most visits
   (208), Cardiology the fewest (193) — no single department is
   overwhelmed relative to the others.
4. **Orthopedics has the longest average wait (34.3 minutes)**, notably
   above Cardiology and Neurology (~28 minutes each).
5. **General Medicine generates the most total revenue** ($268,953),
   despite not having the most patients — its cases tend to cost more on
   average.
6. Overall readmission rate is **14.1%**; **Pediatrics has the highest
   departmental readmission rate at 17.0%**, while **Neurology has the
   lowest at 9.6%**.
7. Patient satisfaction is fairly consistent across departments
   (3.65–3.77 out of 5), suggesting satisfaction issues are more
   individual-visit-driven than department-driven.
8. **34.3% of all visits are flagged as emergency cases** — a meaningful
   share of hospital capacity is going to unplanned care.
9. **$441,433 in treatment cost is sitting in Pending or Unpaid status**
   (30% of total revenue) — a significant receivables/collections gap.
10. Doctor experience shows only a weak relationship with satisfaction
    (ranging 3.63–3.79 across experience buckets) — more experienced
    doctors are not clearly driving higher satisfaction in this dataset.

## 8. Business Recommendations

1. Investigate Orthopedics' intake/triage process specifically — its wait
   time is the outlier, not a hospital-wide problem.
2. Since "General" diagnoses dominate volume, consider a fast-track/
   low-acuity pathway to free up capacity for more complex cases.
3. Target Pediatrics readmissions with a discharge-follow-up program
   (e.g., a callback within 48–72 hours) given its highest readmission
   rate.
4. Study what General Medicine does differently that drives higher
   average revenue per visit — is it treatment mix, longer stays, or
   pricing — and see if it's replicable (ethically and clinically
   appropriately) elsewhere.
5. Prioritize collections on the $441K in Pending/Unpaid balances,
   starting with Self Pay and Government insurance segments if they lag
   Private in payment speed.
6. Because satisfaction doesn't vary much by department, look at
   visit-level drivers instead — e.g., correlate satisfaction with
   waiting time or treatment type to find what actually moves the needle.
7. Since doctor experience alone doesn't predict satisfaction, consider
   whether training, staffing ratios, or specialty fit matter more than
   tenure.
8. Build a standing capacity plan for the ~34% emergency-case share,
   rather than treating emergency volume as purely unpredictable.
9. Set a target readmission rate (e.g., below the current 14.1% average)
   and track it monthly per department using the dashboard's readmission
   page.
10. Use the Hospital Performance Score as a monthly department scorecard
    to catch drift early, and revisit the weighting if leadership
    priorities shift (e.g., more weight on cost control during a budget
    crunch).

## 9. Conclusion

This project demonstrates an end-to-end analytics workflow: generating and
cleaning realistic (synthetic) operational data, modeling it properly for
BI, building measures that answer real business questions, and packaging
findings into an executive-ready dashboard. The techniques here — data
cleaning logic, star-schema modeling, DAX measure design, and a composite
performance score — translate directly to real healthcare, retail,
finance, or operations analytics roles.

---

## 10. Resume Project Description

> **Healthcare Patient & Hospital Performance Analytics Dashboard**
> Built an end-to-end analytics pipeline on a 1,000+ record synthetic
> hospital dataset: cleaned and validated data in Python (Pandas),
> modeled a star schema and loaded it into MySQL, and designed a 3-page
> Power BI dashboard with 12+ DAX measures. Created a custom "Hospital
> Performance Score" combining satisfaction, wait time, readmissions, and
> length of stay into a single 0–100 KPI, and delivered 10 data-driven
> recommendations for hospital operations.

---

## 11. GitHub README Structure (suggested)

```
# Healthcare Patient & Hospital Performance Analytics Dashboard

## Overview
(1-2 paragraphs: what the project does and why)

## Tech Stack
Python (Pandas) · MySQL · Power BI · DAX

## Project Structure
/data
  healthcare_data_raw.csv
  healthcare_data_cleaned.csv
/scripts
  generate_raw_data.py
  data_cleaning.py
/sql
  database_setup.sql
/dax
  dax_measures.txt
/dashboard
  hospital_analytics.pbix
  screenshots/

## Data Cleaning
(short summary + link to data_cleaning.py)

## SQL Analysis
(short summary + link to database_setup.sql, maybe 2-3 example query results)

## Dashboard
(screenshots of all 3 pages + short description of each)

## Key Insights
(top 5 bullet points)

## Hospital Performance Score
(explain the formula briefly, with the disclaimer that it's project-defined)

## How to Run This Project
1. Clone the repo
2. Run scripts/generate_raw_data.py then scripts/data_cleaning.py
3. Load database_setup.sql into MySQL
4. Open dashboard/hospital_analytics.pbix in Power BI Desktop

## Author
(your name, LinkedIn, portfolio link)
```

---

## 12. Interview Questions & Answers (based on this project)

**Q: Walk me through your data cleaning process.**
A: I started with a raw CSV that had realistic problems — missing values,
duplicate rows, numbers stored as text (like "333.0 USD"), invalid dates,
negative costs, and out-of-range satisfaction scores. I used Pandas to
standardize text formatting, coerce columns to correct types with
`pd.to_numeric`/`pd.to_datetime` and `errors="coerce"`, dropped rows with
unrecoverable date errors, clipped out-of-range numeric values, filled
missing numeric values with the median (robust to outliers), and removed
exact and logical duplicates. I validated the result by re-checking for
nulls and dtype consistency before exporting.

**Q: Why did you use the median instead of the mean to fill missing
values?**
A: The median is less sensitive to outliers — treatment cost, for
example, has a right-skewed distribution because a small number of
surgeries cost much more than routine visits. Using the mean there would
have pulled imputed values upward; the median is more representative of a
"typical" visit.

**Q: Why build a star schema instead of one flat table in Power BI?**
A: A star schema (fact table + dimension tables) reduces redundancy,
makes relationships and filtering explicit, and lets DAX time-intelligence
functions work correctly once there's a proper date table. It also scales
better — as you add more descriptive attributes to a dimension (like
department manager or location), you don't have to duplicate that data
across every fact row.

**Q: How did you decide the weights for your Hospital Performance Score?**
A: I treated it as a composite operational KPI and weighted patient
experience (satisfaction) highest at 30%, since it's the most direct
signal of care quality, then split the remaining 70% between waiting
time, readmissions (each 25%, since both reflect real quality/efficiency
issues), and length of stay (20%, a slightly more indirect efficiency
signal). I was explicit that this weighting is a project design choice,
not a clinical standard — in a real setting I'd validate it with
domain experts and possibly benchmark against something like HCAHPS.

**Q: What would you do differently with real hospital data?**
A: I'd add proper PHI/PII handling and de-identification per HIPAA before
any analysis, validate cleaning rules with clinical/operations
stakeholders instead of just statistical judgment calls, and add data
lineage/versioning so changes to cleaning logic are auditable — that
matters a lot more with real patient outcomes on the line.

**Q: How would you scale this if the dataset had 10 million rows instead
of 1,000?**
A: I'd move cleaning from Pandas (in-memory) to something like
PySpark or push more logic into SQL/the database itself, load into a
proper data warehouse instead of flat CSVs, and switch Power BI from
Import mode to DirectQuery or a hybrid (aggregations) mode so the report
doesn't try to hold the full dataset in memory.

**Q: What was the hardest part of this project?**
A: Deciding what to do with ambiguous "invalid" values — e.g., a
satisfaction score of 0 or 7 could be a typo (off-by-one) or truly bad
data. I chose to treat anything outside 1–5 as missing rather than
guessing a "corrected" value, since silently reinterpreting bad data can
introduce more error than it fixes; that's a defensible, explainable
choice I could justify if challenged.
