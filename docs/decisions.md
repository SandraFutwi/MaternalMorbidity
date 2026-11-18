# Decisions log: data acquisition and preprocessing

Owner: Member 1 (data acquisition and preprocessing). Notebook: notebooks/Texas_PUDF_Delivery_SMM_2019_2020.ipynb

Data source: Texas Health Care Information Collection (THCIC), Texas Department of State Health Services (DSHS), Texas Inpatient Public Use Data File, 1Q2019 to 4Q2020. Cite DSHS in every report. The data stay outside this repo (Data Use Agreement).

## 1. Scope: pooled 2019 and 2020

- Decision: pool all 8 quarters of 2019 and 2020 (about 719,000 delivery hospitalizations).
- Why: SMM is rare (about 8 per 1,000 deliveries without transfusion). One year gives too few cases for the smaller race and ethnicity groups. Two years is about 5,500 SMM cases.
- Risk: 2020 is a COVID year (fewer elective admissions, changed care). Sensitivity check: every SMM by race table is also run for 2019 only. The 2019 results show the same pattern (NH Black about 2 times NH White).
- Not included: 2016 to 2018. Adding them would need a larger crosswalk and the 2016 file starts soon after the ICD-10 switch (October 2015).

## 2. Code harmonization (crosswalk)

The ICD-10 code set changes every October 1, and the PUDF uses the discharge date, so even 2019 and 2020 use three versions:

| Quarters | ICD-10-CM and PCS version |
|---|---|
| 1Q2019 to 3Q2019 | FY2019 |
| 4Q2019 to 3Q2020 | FY2020 |
| 4Q2020 | FY2021 |

Check done: every delivery and SMM code in the notebook was compared with the official code files (NCHS ICD-10-CM FY2019, FY2020, FY2021; CMS ICD-10-PCS FY2019 and FY2021; the CMS FY2020 PCS file was not available, and a code valid in both FY2019 and FY2021 is treated as valid in FY2020).

- No listed code was deleted in any year.
- All listed procedure codes (hysterectomy, ventilation, tracheostomy, rhythm conversion, transfusion, delivery) are valid in all years.
- FY2020 added I26.93 and I26.94 (pulmonary embolism). Already caught by the I26 prefix. No change.
- FY2021 added 18 sickle cell "with crisis" codes (D57.03, D57.09, D57.213, D57.218, D57.413, D57.418, D57.431 to D57.439, D57.451 to D57.459, D57.813, D57.818). Added to the sickle cell crisis indicator. The new "without crisis" codes (D57.42, D57.44) are not added.
- FY2021 added transfusion codes 30230C0, 30233C0 and the U2 to U4 codes. These are stem cell transplants, not blood products. Not added.
- The anesthesia complication range (O29.112 to O29.119 and so on) now lists only codes that exist.

Category codes (race, ethnicity, age group, discharge status, payer, specialty unit) are the same in all 8 quarterly User Manuals. No crosswalk needed.

## 3. Cohort

- Delivery hospitalization: diagnosis O80, O82 or Z37; or MS-DRG 765 to 768, 774, 775, 783 to 788, 796 to 798, 805 to 807; or PCS 10D00Z0, 10D00Z1, 10D00Z2, 10E0XZZ. This is our definition, not the CDC's.
- Removed record IDs listed by DSHS: 17,623 outpatient records in 2Q2020 (Technical Update) and 4 erroneous records in 3Q2020 (Addenda Table3). Addenda Tables 1 and 2 (inpatient records filed in the outpatient file) are not added back, because we do not use the outpatient file.
- Age: PAT_AGE is an age group code. The CDC range 12 to 55 cannot be cut exactly, so we keep 10 to 54 (codes 04 to 13). We also keep the masked groups 22 to 24, which DSHS uses for patients with HIV or drug or alcohol codes. Dropped: 226 rows.

## 4. Outcomes

- Main: SMM without transfusion (20 CDC indicators). Sensitivity: SMM with transfusion (21 indicators).
- In-hospital death: PAT_STATUS 20, 40, 41, 42. 46 deaths, too few to model. Descriptive only.
- No present-on-admission filter is used.

## 5. Race and ethnicity, and missing data

- One combined variable, raceEth: Hispanic (any race), NH White, NH Black, NH Asian or Pacific Islander, NH American Indian, NH Other, Suppressed, Unknown.
- Why combined: 19% of deliveries have race 5 (Other), and about two thirds of them are Hispanic. Race alone would hide Hispanic patients in "Other".
- Suppression (User Manual): if a hospital has fewer than 10 patients of a race, the race is changed to 5 and the ethnicity is blanked. We mark race 5 with a blank ethnicity as "Suppressed" (1,370 rows, 0.2%) and set the flag raceSuppressed. A patient whose race really is Other and whose ethnicity is blank looks the same, so this slightly overcounts.
- Treated as missing not at random: no imputation, kept as its own category. Blank or invalid codes are "Unknown" (474 rows).
- Small groups: NH American Indian and Suppressed have fewer than 20 SMM cases. Merge them or report them with the wide confidence interval. Check the Data Use Agreement for the minimum cell size before reporting small counts.

## 6. Encoding

- Features (known at admission): raceEth, ageCat, payerGroup (23 payer codes in 7 groups), admitGroup, teaching, year2020. Comorbidity flags are saved as 0/1 columns (com_ and comPoa_).
- Dummy coding with references NH White, age 25-29 and Private. A switch in the notebook gives full one-hot coding instead.
- The raceSuppressed flag is kept in the data but not in the model matrix, because it equals the Suppressed dummy (collinear).
- Not used as features, because they are measured during or after the stay and partly caused by SMM (leakage): length of stay, discharge status, number of diagnosis codes, the SMM indicators. Cesarean is left out by default for the same reason.
- No numeric scaling needed: all features are categorical.

## 7. Comorbidities

- 29 condition flags built from all 25 diagnosis codes (notebook Step 4b). The list of conditions follows the obstetric comorbidity index idea (Bateman et al. 2013, ICD-10 version by Leonard et al. 2020). The exact code prefixes are our own choice. Check them against the paper before the final report.
- No comorbidity code overlaps an SMM indicator code (checked against ICD-10-CM FY2019, FY2020, FY2021). Examples: abruption uses O45.8 and O45.9 only (O45.0 is in the DIC indicator); sickle cell disease uses the "without crisis" codes only; renal disease leaves out N17.
- Two versions of each flag: com_ (code anywhere on the record, the usual index method) and comPoa_ (present on admission, POA Y or W). Use comPoa_ for prediction. 3.8% of deliveries have no POA code at all, so comPoa_ undercounts slightly.
- Check: SMM rises with the number of comorbidities (2.0 per 1,000 with none, 85 per 1,000 with 5 or more). Prevalence is plausible (prior cesarean 20%, gestational diabetes 8%, chronic hypertension 3.6%).
- Open question for the team: which conditions count as baseline risk. Anemia (O99.0 includes anemia in childbirth and the puerperium, which can follow a hemorrhage), severe preeclampsia, bleeding disorder, preterm birth and abruption can arise during the delivery stay. In a first model (age, payer, year), the NH Black odds ratio is 1.83. With all comorbidities it is 1.38, with POA comorbidities 1.47, and without those five conditions 1.68. Anemia alone moves it the most (1.38 to 1.53). Report the main model and this sensitivity version.

## 8. Other new fields

- Residence from PAT_STATE (User Manual): TX = Texas; AR, LA, NM, OK, ZZ (all other states and territories) = other US state; FC, XX = foreign country; back quote or blank = unknown. 1.5% are not Texas residents.
- region: PUBLIC_HEALTH_REGION (1 to 11) of the patient's address. Blank for non-Texas residents and 2,181 Texas residents (0.3%).
- admitGroup (TYPE_OF_ADMISSION): emergency, urgent, elective, other or unknown. Model feature, reference elective.
- transferIn (SOURCE_OF_ADMISSION 4, 0.6%) and edVisit (EMERGENCY_DEPT_FLAG, 26.5%): saved as information only. A transfer can happen because the complication already started, so it is a mediator for the disparity question.
- teaching (facility type file, FAC_TEACHING_IND): COTH member (A), other teaching (X), non-teaching. Model feature, reference non-teaching. 12 deliveries have no matching hospital (Unknown).

## Outputs (local only, never commit)

- DataSets/output/delivery_cohort_2019_2020.parquet: cohort with flags and cleaned fields
- DataSets/output/model_data_2019_2020.parquet: model-ready table
- DataSets/output/smm_by_race_ethnicity.csv: rates, confidence intervals, rate ratios
