# MIMIC-III Neuro Cohort Analysis: SQL, Tableau & Excel

An end-to-end clinical data analysis comparing ICU outcomes for neurological/neurodegenerative admissions against the general ICU population, built on the MIMIC-III Clinical Database Demo -- a normalized 8-table SQL database, an interactive Tableau dashboard with parameters, and an Excel QC/pivot analysis.

**Live dashboard:** [Tableau Public -- MIMIC-III Neuro Cohort Analysis](https://public.tableau.com/app/profile/tanvi.gambhir/viz/MIMIC-IIINeuroCohortAnalysis/MIMIC-IIINeuroCohortAnalysis)

## Data

[MIMIC-III Clinical Database Demo v1.4](https://physionet.org/content/mimiciii-demo/1.4/) (Johnson, Pollard, Mark), open access via PhysioNet, no credentialing required. 100 ICU patients, 129 hospital admissions, 136 ICU stays, real de-identified clinical records from Beth Israel Deaconess Medical Center.

**Cohort note:** the neuro/neurodegenerative subgroup identified in this analysis is 8 patients / 9 admissions out of 100 patients / 129 admissions total (~8% of the cohort). This is a random general-ICU sample, not curated to be neuro-heavy, so all neuro-vs-general comparisons below are descriptive for this small subgroup, not statistically powered claims.

## Project structure

**1. SQL -- 8-table relational schema & analysis** (`mimic_neuro_cohort_analysis.ipynb`)
- Designed and built a normalized schema from 8 raw MIMIC tables (`patients`, `admissions`, `icustays`, `diagnoses_icd`, `d_icd_diagnoses`, `prescriptions`, `labevents`, `d_labitems`), with primary keys, a composite key, and cross-referenced foreign keys throughout
- Identified the neuro cohort via a CTE with a nested subquery, matching ICD-9 diagnosis text against a set of neurological/neurodegenerative keywords (Parkinson's, Alzheimer's, dementia, epilepsy, seizure, stroke, multiple sclerosis)
- Used window functions (`RANK()`), scalar subqueries for side-by-side cohort comparisons, and multi-table joins across the full schema
- **Findings:**
  - Neuro admissions average 6.81 ICU days vs. 4.45 overall (~53% longer)
  - Neuro admissions: 44.4% in-hospital mortality vs. 31.0% overall
  - Medication and lab-abnormality patterns cross-validate each other: anti-seizure drugs (phenytoin, levetiracetam, phenobarbital) and Carbidopa-Levodopa (the standard Parkinson's medication) appear as the clinically distinctive prescriptions, and drug-level monitoring labs for the same medications (phenytoin, valproic acid) appear as the clinically distinctive abnormal labs -- two independent tables telling the same clinical story

**2. Tableau -- interactive dashboard with parameters**
- 5 linked views: LOS comparison, mortality comparison, a parameter-driven combo chart letting a viewer toggle between metrics with dynamically formatted labels (days vs. percentage), and top-15 charts for medications and abnormal labs
- Built a **Tableau parameter** with a supporting calculated field (`CASE` logic) to drive the dynamic metric-comparison chart, plus a second calculated field generating properly formatted on-chart labels for each metric type
- Published live on Tableau Public (link above)

**3. Excel -- QC, pivot analysis & lookup** (`mimic_neuro_analysis.xlsx`)
- 6-check QC section: no negative length-of-stay or age values, no impossible date ordering (discharge before admission), no duplicate admission IDs, no missing values across key fields -- plus explicit identification and labeling of MIMIC's intentional age-obscuring convention (ages calculated above 100, up to 300, for the 10 admissions involving patients 89+, per MIMIC's de-identification standard) rather than treating it as a data error
- Two-dimension pivot table (length of stay by neuro cohort × gender) with a pivot chart
- **Finding, with caveat:** female neuro-cohort admissions show a notably longer average LOS than male (8.76 vs. 2.91 days) -- but with only 6 female and 3 male admissions in this subgroup, this gap is substantially driven by a single 29-day outlier stay, not a consistent pattern across the group. Reported here as a flagged observation, not a reliable trend.
- `XLOOKUP` against a reference table to generate readable cohort labels, and conditional formatting on the QC section (green = passed check, grey = informational count, not pass/fail)

## Tools

Python (pandas), SQLite (`sqlite3`), SQL (CTEs, subqueries, window functions, composite/foreign keys), Tableau Public (parameters, calculated fields, dashboard actions), Microsoft Excel (structured references, `XLOOKUP`, PivotTables, PivotCharts, conditional formatting)

## Files in this repo

- `neuro_cohort_analysis.ipynb` -- schema design, data loading, and SQL analysis (run top to bottom; requires the MIMIC-III Demo CSVs, downloaded separately from PhysioNet)
- `mimic_admissions_for_tableau.csv`, `mimic_medications_for_tableau.csv`, `mimic_labs_for_tableau.csv` -- exported datasets feeding Tableau and Excel
- `neuro_analysis.xlsx` -- Excel QC, pivot table/chart, and lookup analysis
- `README.md` -- this file

*(`mimic_neuro.db`, the SQLite database file, and the raw MIMIC-III Demo CSVs are generated/downloaded when running the notebook and aren't tracked in this repo.)*
