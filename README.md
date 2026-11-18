# Severe maternal morbidity by race and ethnicity in Texas (2019 and 2020)

Healthcare AI course project. We measure racial and ethnic disparities in severe maternal morbidity (SMM, CDC 21 indicators) among Texas delivery hospitalizations, and build and audit SMM prediction models.

## Data (not in this repo)

Texas Health Care Information Collection (THCIC), Texas Department of State Health Services (DSHS), Texas Inpatient Public Use Data File, 1Q2019 to 4Q2020.

- Each team member accepts the Data Use Agreement on the DSHS PUDF page and downloads their own copy.
- Never commit, upload or email the data or any file made from it. The .gitignore blocks the common data file types, but check `git status` before every commit.
- Cite DSHS in every report.

Files to download into one folder outside this repo (for example `~/Documents/DataSets`):
- PUDF1Q2019 to PUDF4Q2020, tab-delimited zips (do not rename them)
- THCICListofRecordIDsofOutpatientdataappearinginQ22020InpatientDataset.xlsx
- THCICPUDF3Q2020RecordIDs06-23-2021Addenda.xlsx

## How to run

1. Set `dataDir` in the first code cell of each notebook to your data folder.
2. Run `notebooks/Texas_PUDF_Delivery_SMM_2019_2020.ipynb` (about 3 minutes). It saves the cohort and model files in `dataDir/output`.
3. Run `notebooks/Texas_PUDF_SMM_EDA_2019_2020.ipynb` (data exploration).

Python with pandas, numpy, matplotlib, statsmodels, pyarrow and openpyxl (Anaconda has all of them).

## Docs

- `docs/decisions.md`: data and preprocessing decisions (scope, ICD-10 crosswalk, cohort, outcomes, race and missing data, encoding, comorbidities).
- `docs/project_objectives.md`: revised objectives after the data exploration.

## Before you commit

- Clear all notebook outputs (they contain counts from the data).
- Do not paste results with small counts into issues, commits or docs before checking the Data Use Agreement rules.
