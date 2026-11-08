---
name: metabolite-missing-value-mcar-mnar-classification
description: Use when a metabolomics concentration table (.csv/.xlsx) contains missing
  metabolite measurements and values must be filled prior to normalization, transformation,
  and scaling, with missingness mechanism (MNAR vs MCAR) driving the choice of imputation
  method.
when_to_use_negative:
- Data have not yet passed error correction — missing data exceeding the threshold
  within a group or across all samples, non-numeric concentration values, missing/non-unique
  primary ID, or low-variance metabolites must be corrected first (imputation is 'automatically
  performed' only once errors are corrected)
- The analysis goal is upstream preprocessing or annotation — Viime's intent is to
  pick up the workflow after preprocessing and annotation, not to perform those steps
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Viime
  role: Web application implementing the MCAR/MNAR classification and imputation dispatch
    as an automatic step after error correction
  repo: https://github.com/girder/viime
- name: missForest
  role: R package providing Random Forest imputation for metabolites classified as
    MCAR
- name: impute
  role: R package providing K-Nearest Neighbor imputation for metabolites classified
    as MCAR
- name: OpenCPU
  role: Secondary REST service exposing the custom viime R package's CSV-processing
    imputation functions to the Flask backend
- name: Pandas
  role: Parses the raw concentration CSV and exchanges data frames with the R imputation
    functions via opencpu.py
- name: Flask
  role: RESTful API backend that triggers imputation once errors are corrected in
    the uploaded table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:c0aae883aaacb1e2e0633bc7088f61c7922ef6bccd2a22ea980ab3d23db37824
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/girder/viime
  tools:
  - Flask
  - OpenCPU
  - Pandas
  - R
  - Scikit-learn
  - Viime
  - impute
  - missForest
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# metabolite-missing-value-mcar-mnar-classification

## Summary

Viime's imputation strategy heuristically classifies missing metabolomics values per group/metabolite as MNAR or MCAR (following Armitage et al., 2015), then imputes MNAR with zero or half-minimum and MCAR with missForest, KNN (impute), mean, or median. This distinguishes values missing below detection limit from values missing at random before filling them.

## When to use

Use when a metabolomics concentration table (.csv/.xlsx) contains missing metabolite measurements and values must be filled prior to normalization, transformation, and scaling, with missingness mechanism (MNAR vs MCAR) driving the choice of imputation method.

## When NOT to use

- Data have not yet passed error correction — missing data exceeding the threshold within a group or across all samples, non-numeric concentration values, missing/non-unique primary ID, or low-variance metabolites must be corrected first (imputation is 'automatically performed' only once errors are corrected)
- The analysis goal is upstream preprocessing or annotation — Viime's intent is to pick up the workflow after preprocessing and annotation, not to perform those steps

## Inputs

- Metabolite concentration table uploaded as .xlsx or .csv (with primary ID, factor, group, and metabolite concentration columns designated)
- User-selected MNAR imputation strategy (zero or half-minimum)
- User-selected MCAR imputation method (missForest, impute KNN, mean, or median)

## Outputs

- Fully imputed concentration table (CSV/data frame) with same rows and columns as input and no remaining missing values
- Per-metabolite MNAR/MCAR missingness classification

## How to apply

Load the metabolite concentration table and compute the percentage of missing values per metabolite within each group. Apply the Armitage et al. (2015) heuristic to classify each metabolite's missing values as MNAR (values below detection limit, concentrated in specific groups) or MCAR (missing at random). For MNAR metabolites, impute with zero or half the minimum observed concentration as selected. For MCAR metabolites, dispatch to one of: Random Forest imputation via the R missForest package, K-Nearest Neighbor via the R impute package, or simple mean/median imputation. The R functions are exposed to the Python/Flask backend through OpenCPU, with viime/opencpu.py handling the pandas-to-R data frame conversion; the completed, fully imputed data frame is returned as the output table.

## Related tools

- **Viime** (Web application implementing the MCAR/MNAR classification and imputation dispatch as an automatic step after error correction) — https://github.com/girder/viime
- **missForest** (R package providing Random Forest imputation for metabolites classified as MCAR)
- **impute** (R package providing K-Nearest Neighbor imputation for metabolites classified as MCAR)
- **OpenCPU** (Secondary REST service exposing the custom viime R package's CSV-processing imputation functions to the Flask backend)
- **Pandas** (Parses the raw concentration CSV and exchanges data frames with the R imputation functions via opencpu.py)
- **Flask** (RESTful API backend that triggers imputation once errors are corrected in the uploaded table)

## Examples

```
cd devops && docker build -t viime . && docker run --rm --name viime -p 8004:8004 viime  # OpenCPU instance exposing the viime R imputation functions for CSV processing
```

## Evaluation signals

- Output CSV exists and has the same rows and columns as the input table
- The completed data frame contains no remaining missing values
- MNAR-classified metabolites are filled only with zero or half the minimum observed concentration (no values above the observed minimum are invented)
- MCAR-classified metabolites are filled exclusively by the selected method (missForest, impute KNN, mean, or median)
- Classification respects the Armitage et al. (2015) percentage-of-missingness heuristic computed per group per metabolite

## Limitations

- The MNAR/MCAR assignment is heuristic, not a statistical test; borderline missingness patterns may be misclassified, changing which imputation method is applied
- R-based MCAR methods require the OpenCPU sidecar container; changes to R functions require rebuilding the docker image before they are exposed to the API server
- Missingness above the default error threshold within a group or across samples is flagged as an error to be corrected rather than silently imputed

## Evidence

- [abstract] Heuristic MNAR/MCAR classification with Armitage 2015 strategy: "Viime uses a sophisticated imputation strategy (Armitage, Godzien, Alonso-Herranz, López-Gonzálvez, & Barbas, 2015), heuristically classifying missing data as Missing Not At Random (MNAR) or Missing"
- [abstract] Imputation runs automatically after error correction: "Once errors are corrected, data imputation is automatically performed."
- [abstract] Random Forest and KNN implemented via missForest and impute R packages: "Random Forest and K-Nearest Neighbor were implemented with the R packages missForest and impute, respectively."
- [readme] OpenCPU mediates pandas-to-R data frame exchange for CSV processing functions: "Functions contained in this package should accept a path to a CSV file as an argument and return a data frame. The (work-in-progress) backend code at [viime/opencpu.py](viime/opencpu.py) handles the"
- [abstract] Error detection includes missing data above threshold before imputation: "Errors that are detected include levels of missing data that exceed a default threshold within a group or across all samples, non-numeric data in concentration data"
