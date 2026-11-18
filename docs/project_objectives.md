# Revised project objectives (after data exploration)

Based on notebooks/Texas_PUDF_SMM_EDA_2019_2020.ipynb (718,811 Texas delivery hospitalizations, 2019 and 2020, Texas Inpatient PUDF, DSHS). Numbers below are for team planning only. Do not share them outside the team before checking the Data Use Agreement.

## What the exploration showed

- SMM without transfusion: 7.7 per 1,000 deliveries (1 in 130, 5,531 cases). Strong class imbalance.
- 62% of SMM-with-transfusion cases are transfusion only, so the transfusion version is mostly a transfusion outcome.
- NH Black vs NH White: odds ratio 1.96 crude, 1.72 after age, payer, year and hospital volume. Hispanic: 1.19 crude, 1.09 adjusted. NH Asian or Pacific Islander: no gap after adjustment.
- The Black and White gap is the same in Medicaid and in private insurance, and inside every hospital group.
- The gap is largest for renal failure (3.5 times), and about 2 times for embolism, respiratory, sepsis, cardiac and hypertensive or cerebrovascular events. Hemorrhage: 1.4 times.
- Sickle cell crisis is 15 times higher for NH Black patients but explains little of the total gap (1.95 drops to 1.88 without it).
- High risk groups: age 35+, Medicare, and the masked HIV or drug or alcohol group (24.9 per 1,000).
- 61% of NH Black deliveries are in the third of hospitals with the highest NH Black share. 26 of 229 hospitals are outside the 99.8% funnel limits, so hospitals must be modeled as clusters.
- SMM rose from 7.4 (2019) to 8.0 (2020) per 1,000.
- Too small for their own estimates: NH American Indian (10 cases), Suppressed, Unknown, and in-hospital deaths (46).
- 198 of 254 counties have fewer than 10 cases.
- Blank sex and ZIP come only from DSHS masking (the HIV or drug or alcohol age group). Most blank counties are non-Texas residents (1.5% of patients). Do not impute.
- Admission type (known at admission): elective 6.6 vs emergency 8.9 per 1,000. NH White patients have more elective admissions, but the NH Black gap is the same within each type.
- Hypertensive disorders are the principal diagnosis for 10% of deliveries but 26% of SMM cases, and are more common in NH Black patients (14% vs 11%). Placenta previa or abruption: 72 per 1,000.
- The NH Black gap is at every age (largest at 30 to 39, 2.2 times) and the same for vaginal and cesarean deliveries (1.8 times).
- Medicaid share: 59% Hispanic, 58% NH Black, 30% NH White. Payer is a real confounder.
- Ventilation is almost always coded with ARDS. Shock clusters with DIC and hysterectomy.
- At NH White rates for the same age and payer, NH Black patients would have about 470 fewer SMM cases in 2 years (44%), Hispanic patients about 270 fewer (13%).
- Comorbidities (first look): the NH Black odds ratio goes from 1.83 (age, payer, year) to 1.38 with all comorbidities, 1.47 with present-on-admission comorbidities, and 1.68 without the conditions that can arise during the stay (anemia, severe preeclampsia, bleeding disorder, preterm birth, abruption). The choice of comorbidities changes the answer, so it must be decided and reported.

## Revised objectives

1. Disparity estimation (primary)
   - Outcome: SMM without transfusion. Sensitivity: with transfusion, without sickle cell crisis, 2019 only, Texas residents only.
   - Report excess cases (observed minus expected at NH White rates) next to the odds ratios.
   - Groups vs NH White: Hispanic, NH Black, NH Asian or Pacific Islander, NH Other. NH American Indian, Suppressed and Unknown are descriptive only.
   - Adjust in steps: (a) age, payer, year; (b) plus comorbidities (com_ flags); (c) plus hospital (volume, teaching status and a hospital random effect). Report the odds ratio at each step.
   - Sensitivity for step (b): present-on-admission flags only, and without anemia, severe preeclampsia, bleeding disorder, preterm birth and abruption.
   - New: test whether the gap exists within hospitals.

2. Complication pathways
   - SMM subtypes (hemorrhage, hypertensive or cerebrovascular, sepsis, cardiac, respiratory, renal, embolism) for NH Black and Hispanic vs NH White. Sickle cell crisis on its own.

3. Prediction and fairness
   - Predictors known at admission only: age, payer, admission type, comorbidities present on admission (comPoa_ flags), hospital volume and teaching status, year.
   - The principal diagnosis is not a predictor as is: some codes are complications of the stay (O71, O72, O75).
   - Not predictors (leakage): length of stay, discharge status, cesarean, APR-DRG severity and risk of mortality (computed from all diagnoses, including SMM codes).
   - Logistic regression or LASSO vs gradient boosting.
   - Train 2019, test 2020 (COVID shift), plus a random split within years for comparison.
   - AUPRC, recall at a fixed alert rate, calibration. By group only for groups with 100+ test cases.
   - Race x payer interaction only for Medicaid vs private, and NH White, Hispanic, NH Black.

4. Hospitals and regions (replaces the "if time allows" goal)
   - Hospital volume and share of NH Black patients per hospital.
   - 11 Public Health Regions instead of single counties. March of Dimes data optional (join on county FIPS).

Secondary outcomes: length of stay (as an outcome only). In-hospital death as a total count only.

## Changes from the proposal

| Proposal | Now | Why |
|---|---|---|
| Cohort from the obstetric unit flag and revenue codes | Diagnosis, DRG and procedure codes | The flag is on only 65% of deliveries |
| Median imputation for age and length of stay | Not needed | Age is a group code with no blanks; length of stay is not a predictor |
| z-score scaling of age and length of stay | Not needed | All predictors are categorical |
| Length of stay as a feature | Outcome only | It is caused partly by SMM (leakage) |
| Hospital obstetric-unit volume | Deliveries per hospital per year | Computed directly from the cohort |
| Mortality model | Descriptive only | 46 deaths |
| Region analysis if time allows | Planned, by Public Health Region | Counties are too small |

## Preprocessing status (Member 1)

Done:
- 29 comorbidity flags from all diagnosis codes, in two versions (anywhere and present on admission), plus counts. See decisions.md section 7.
- Public Health Region, residence (Texas, other state, foreign), admission type, transfer in, emergency department flag.
- Hospital teaching status from the facility type file.

Still open:
- Team decision: which comorbidities count as baseline risk (see the comorbidity finding above).
- Optional: the weighted Leonard et al. 2020 index, if we copy the weights from the paper.
