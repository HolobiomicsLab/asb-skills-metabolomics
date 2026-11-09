---
name: mcar-missforest-imputation
description: Use when a metabolomics concentration table (CSV/XLSX) has missing entries
  whose per-group, per-metabolite missingness percentage classifies them as MCAR,
  and statistically rigorous imputation (missForest, KNN, mean, or median) is required
  before normalization or PCA.
when_to_use_negative:
- Missing values were classified as MNAR (missing not at random) — use zero or half-minimum
  imputation instead of missForest/KNN
- The table is not a metabolomics concentration table (e.g. already-normalized or
  fused multi-platform data) — Viime's intent is to pick up the workflow after preprocessing
  and annotation
- Data exceeds Viime's error thresholds (e.g. missing data exceeding the default threshold
  within a group or across all samples, non-numeric concentration data, missing or
  non-unique primary ID) — correct errors first, since imputation is only automatically
  performed once errors are corrected
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0154
tools:
- name: missForest
  role: R package implementing Random Forest imputation for MCAR-classified missing
    metabolite values
- name: OpenCPU
  role: Secondary REST service exposing the R imputation functions to the Python/Flask
    backend
  repo: https://www.opencpu.org/
- name: Pandas
  role: Parses the raw CSV concentration file and converts to/from R data frames via
    viime/opencpu.py
- name: Viime
  role: Web application orchestrating the MCAR/MNAR classification and imputation
    dispatch in its data-treatment workflow
  repo: https://github.com/girder/viime
- name: R
  role: Hosts the specialized imputation algorithms (missForest, impute KNN) in the
    custom viime R package
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:bde872b222ed619b0b9961ac0b66f50d76cc67c6fb3d807eb8d3e4fc6b37ecb3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OpenCPU
  - Pandas
  - R
  - Scikit-learn
  - Viime
  - missForest
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# mcar-missforest-imputation

## Summary

Imputes metabolite concentration values classified as Missing Completely At Random (MCAR) using Random Forest (missForest) or K-Nearest Neighbor (impute) R packages, or simple mean/median, as part of Viime's Armitage et al. (2015) MCAR/MNAR missingness strategy.

## When to use

Use when a metabolomics concentration table (CSV/XLSX) has missing entries whose per-group, per-metabolite missingness percentage classifies them as MCAR, and statistically rigorous imputation (missForest, KNN, mean, or median) is required before normalization or PCA.

## When NOT to use

- Missing values were classified as MNAR (missing not at random) — use zero or half-minimum imputation instead of missForest/KNN
- The table is not a metabolomics concentration table (e.g. already-normalized or fused multi-platform data) — Viime's intent is to pick up the workflow after preprocessing and annotation
- Data exceeds Viime's error thresholds (e.g. missing data exceeding the default threshold within a group or across all samples, non-numeric concentration data, missing or non-unique primary ID) — correct errors first, since imputation is only automatically performed once errors are corrected

## Inputs

- metabolomics concentration table as a CSV or .xlsx file with group, primary ID, and metabolite concentration columns
- missingness classification flags per metabolite per group (MNAR vs MCAR, from the Armitage et al. 2015 percentage-based heuristic)

## Outputs

- fully imputed metabolomics concentration data frame written to an output CSV with the same rows and columns as the input and no remaining missing values

## How to apply

First classify each metabolite's missing values as MNAR or MCAR using the percentage of missing values within each group and metabolite, following the Armitage et al. (2015) heuristic used by Viime; only values classified as MCAR enter this skill (MNAR values are instead imputed with zero or half-minimum). For MCAR metabolites, dispatch to one of the available methods: Random Forest via the R missForest package, K-Nearest Neighbor via the R impute package, mean, or median. R-based methods are exposed through a custom R package called over OpenCPU; the backend (viime/opencpu.py) handles communication between pandas and R data frames, and R functions accept a path to a CSV file and return a data frame. After imputation, write the completed data frame to an output CSV and verify it has the same rows and columns as the input with no remaining missing values before proceeding to normalization (reference sample, sum scaling, weight/volume), transformation (Log10, Log2, square root, cube root), and scaling.

## Related tools

- **missForest** (R package implementing Random Forest imputation for MCAR-classified missing metabolite values)
- **OpenCPU** (Secondary REST service exposing the R imputation functions to the Python/Flask backend) — https://www.opencpu.org/
- **Pandas** (Parses the raw CSV concentration file and converts to/from R data frames via viime/opencpu.py)
- **Viime** (Web application orchestrating the MCAR/MNAR classification and imputation dispatch in its data-treatment workflow) — https://github.com/girder/viime
- **R** (Hosts the specialized imputation algorithms (missForest, impute KNN) in the custom viime R package)

## Examples

```
docker run --rm --name viime -p 8004:8004 viime  # starts the OpenCPU instance serving the viime R package (missForest/impute) for CSV imputation calls
```

## Evaluation signals

- Output CSV exists and has the same number of rows and columns as the input CSV
- Output contains no remaining missing values after imputation
- MNAR-classified entries were NOT imputed by missForest/KNN (they should carry zero or half-minimum values instead)
- The R function call returns a data frame of the same shape as the input CSV (R functions 'should accept a path to a CSV file as an argument and return a data frame')
- Downstream PCA on the imputed table runs without errors during Viime's interactive data-treatment step

## Limitations

- The MCAR/MNAR assignment is a heuristic based on percentage of missing values per group per metabolite, not a formal statistical test
- Random Forest and KNN imputation require the OpenCPU service (a Docker container exposing the custom viime R package) to be running; the opencpu.py bridge is described as work-in-progress
- The README's R-function interface is work-in-progress; adding methods requires editing the viime R package and rebuilding the Docker image

## Evidence

- [abstract] Viime offers a sophisticated imputation strategy that heuristically classifies missing data as MNAR or MCAR.: [evidence span withheld — anchor status: not_found]
- [abstract] Random Forest and K-Nearest Neighbor were implemented with the R packages missForest and impute, respectively.: "Random Forest and K-Nearest Neighbor were implemented with the R packages missForest and impute, respectively."
- [abstract] The R-python integration is provided by a secondary REST service exposed internally via OpenCPU.: "The R-python integration is provided by a secondary REST service exposed internally via OpenCPU."
- [readme] R package functions accept a CSV path and return a data frame.: "Functions contained in this package
should accept a path to a CSV file as an argument and return a data frame."
- [abstract] Data imputation is automatically performed after error correction.: "Once errors are corrected, data imputation is automatically performed."
