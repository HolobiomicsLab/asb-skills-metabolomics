---
name: knn-imputation-for-metabolomics
description: Use when a metabolomics concentration table (CSV/.xlsx uploaded to Viime)
  contains missing values heuristically classified as MCAR per metabolite per group,
  and a neighbor-based imputation is preferred over missForest, mean, or median.
when_to_use_negative:
- Missing values are classified as MNAR (below-detection metabolites missing by group)
  — Viime imputes these with zero or half the minimum concentration instead of KNN
- The concentration table contains error conditions Viime flags, such as non-numeric
  concentration data or a missing/non-unique primary ID — these must be corrected
  before automatic imputation
- No OpenCPU/R backend is available (e.g., pure-Python environment), since KNN and
  missForest imputations in Viime are implemented with R packages, not Scikit-learn
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3577
tools:
- name: impute
  role: R package that implements the K-Nearest Neighbor imputation used for MCAR-classified
    missing metabolomics data
- name: Viime
  role: Web application that classifies missingness as MNAR/MCAR and dispatches the
    selected MCAR imputation method (KNN via 'impute') during automatic data imputation
  repo: girder/viime
- name: OpenCPU
  role: Secondary REST service exposing the custom viime R package so the Python backend
    can call the R imputation functions on CSV files
- name: Pandas
  role: Parses the raw uploaded file and handles the pandas↔R data frame handoff in
    viime/opencpu.py around the R imputation call
- name: missForest
  role: Alternative Random-Forest-based R-package imputation offered for the same
    MCAR class as an option alongside KNN, mean, and median
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:7ef10b7eaeb9bf897a844b239dcefd1268b225f74ea3bd93689031d094371975
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/girder/viime
  tools:
  - OpenCPU
  - Pandas
  - R
  - Scikit-learn
  - Viime
  - impute
  - missForest
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# knn-imputation-for-metabolomics

## Summary

Imputes missing values in metabolomics concentration tables that have been classified as MCAR (Missing Completely At Random) using K-Nearest Neighbor imputation from the R 'impute' package, as one of the MCAR methods offered by Viime alongside missForest, mean, and median.

## When to use

Use when a metabolomics concentration table (CSV/.xlsx uploaded to Viime) contains missing values heuristically classified as MCAR per metabolite per group, and a neighbor-based imputation is preferred over missForest, mean, or median.

## When NOT to use

- Missing values are classified as MNAR (below-detection metabolites missing by group) — Viime imputes these with zero or half the minimum concentration instead of KNN
- The concentration table contains error conditions Viime flags, such as non-numeric concentration data or a missing/non-unique primary ID — these must be corrected before automatic imputation
- No OpenCPU/R backend is available (e.g., pure-Python environment), since KNN and missForest imputations in Viime are implemented with R packages, not Scikit-learn

## Inputs

- metabolomics concentration table as CSV (or .xlsx via Viime upload) with missing concentration entries
- MNAR/MCAR classification per metabolite per group (percentage of missing values within each group and metabolite)
- user selection of KNN (R 'impute') as the MCAR imputation method

## Outputs

- fully imputed concentration data frame / output CSV with the same rows and columns as the input and no remaining missing values

## How to apply

First, following the Armitage et al. (2015) strategy used by Viime, classify each metabolite's missing entries as MNAR or MCAR based on the percentage of missing values within each group and metabolite; MNAR entries must be imputed with zero or half-minimum instead of KNN. For metabolites classified as MCAR, apply K-Nearest Neighbor imputation implemented with the R 'impute' package. In Viime's architecture, the imputation runs inside a custom R package exposed through an OpenCPU REST service, with viime/opencpu.py handling communication between pandas and R data frames. The R functions accept a path to a CSV file as the argument and return a completed data frame, which is written out as a fully imputed CSV.

## Related tools

- **impute** (R package that implements the K-Nearest Neighbor imputation used for MCAR-classified missing metabolomics data)
- **Viime** (Web application that classifies missingness as MNAR/MCAR and dispatches the selected MCAR imputation method (KNN via 'impute') during automatic data imputation) — girder/viime
- **OpenCPU** (Secondary REST service exposing the custom viime R package so the Python backend can call the R imputation functions on CSV files)
- **Pandas** (Parses the raw uploaded file and handles the pandas↔R data frame handoff in viime/opencpu.py around the R imputation call)
- **missForest** (Alternative Random-Forest-based R-package imputation offered for the same MCAR class as an option alongside KNN, mean, and median)

## Examples

```
docker run --rm --name viime -p 8004:8004 viime
```

## Evaluation signals

- The output CSV exists and has the same number of rows and columns as the input concentration table
- The output CSV contains no remaining missing values after imputation
- KNN ('impute') was applied only to metabolites classified MCAR; MNAR metabolites were imputed with zero or half-minimum instead
- Result is comparable to the alternative MCAR options (missForest, mean, median) — large divergence from missForest on the same MCAR set warrants review of the MNAR/MCAR classification
- The OpenCPU service and custom viime R package are running and reachable from the Flask backend before the imputation call is issued

## Limitations

- The MNAR vs MCAR classification is heuristic, based on the percentage of missing values per group per metabolite; misclassification routes metabolites to the wrong imputation strategy (zero/half-minimum vs KNN)
- KNN imputation requires the OpenCPU docker instance with the custom viime R package; the backend bridge code at viime/opencpu.py is described in the README as work-in-progress
- Viime only detects missingness after error correction (e.g., missing data exceeding a default threshold within a group or across all samples) — severe missingness may be flagged as an error before imputation is reached

## Evidence

- [abstract] KNN implemented with R 'impute' package: "Random Forest and K-Nearest Neighbor were implemented with the R packages missForest and impute, respectively."
- [abstract] MNAR/MCAR heuristic classification strategy: "Viime uses a sophisticated imputation strategy (Armitage, Godzien, Alonso-Herranz, López-Gonzálvez, & Barbas, 2015), heuristically classifying missing data as Missing Not At Random (MNAR) or Missing"
- [readme] R functions accept CSV path and return data frame: "Functions contained in this package should accept a path to a CSV file as an argument and return a data frame."
- [abstract] OpenCPU provides the R-Python bridge: "The R-python integration is provided by a secondary REST service exposed internally via OpenCPU."
