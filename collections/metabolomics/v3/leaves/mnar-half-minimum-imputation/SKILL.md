---
name: mnar-half-minimum-imputation
description: Use when a metabolomics concentration table (.csv/.xlsx) contains missing
  peak values that cluster within whole groups per metabolite — indicating values
  below the detection limit (MNAR) — and those values should be filled with zero or
  half the minimum observed concentration.
when_to_use_negative:
- Missingness is scattered at low percentage per group (MCAR) — use missForest, impute
  K-Nearest Neighbor, mean, or median instead of zero/half-minimum replacement
- Input is raw instrument data or un-annotated spectra — Viime is intended to pick
  up the workflow after preprocessing and annotation
- The table is already complete (no missing values) — imputation would alter valid
  zeros/absences
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_2269
tools:
- name: Viime
  role: Web application implementing the Armitage et al. (2015) heuristic that classifies
    missing data as MNAR or MCAR and dispatches MNAR values to zero/half-minimum imputation
  repo: girder/viime
- name: R
  role: Hosts the custom viime package whose CSV-processing functions perform the
    imputation; specialized algorithms run as R packages
- name: OpenCPU
  role: Internal REST service bridging the Flask/Pandas backend to the R imputation
    functions
  repo: https://www.opencpu.org/
- name: Pandas
  role: Parses the raw input file and exchanges data frames with the R side (viime/opencpu.py)
  repo: girder/viime
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:1c51d144f30abd264f80779670b2a55e47a04f8b7b7d0ed4f74a70f578746595
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
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# mnar-half-minimum-imputation

## Summary

Viime's MNAR imputation strategy: metabolite missing values heuristically classified as Missing Not At Random (MNAR) — typical of peaks below detection limits — are replaced with zero or half the minimum observed concentration, following the Armitage et al. (2015) approach, as distinct from MCAR methods (missForest, impute KNN, mean, median).

## When to use

Use when a metabolomics concentration table (.csv/.xlsx) contains missing peak values that cluster within whole groups per metabolite — indicating values below the detection limit (MNAR) — and those values should be filled with zero or half the minimum observed concentration.

## When NOT to use

- Missingness is scattered at low percentage per group (MCAR) — use missForest, impute K-Nearest Neighbor, mean, or median instead of zero/half-minimum replacement
- Input is raw instrument data or un-annotated spectra — Viime is intended to pick up the workflow after preprocessing and annotation
- The table is already complete (no missing values) — imputation would alter valid zeros/absences

## Inputs

- Metabolomics concentration table (.csv or .xlsx) with metabolite concentration columns and designated group/factor columns
- Selection of MNAR replacement strategy (zero or half-minimum)

## Outputs

- Fully imputed concentration table written to CSV with the same rows and columns as the input and no remaining missing values
- Per-metabolite MNAR/MCAR classification implied by the imputation dispatch

## How to apply

Load the metabolomics concentration table and identify missing entries per metabolite per group, respecting the user-designated group/factor columns. Heuristically classify each metabolite's missing values as MNAR or MCAR based on the percentage of missing values within each group and metabolite, following the Armitage et al. (2015) strategy adopted by Viime. For metabolites classified as MNAR (values absent below detection limits), replace missing entries with zero or half the minimum observed concentration, as selected. Metabolites classified as MCAR are routed instead to Random Forest (missForest), K-Nearest Neighbor (impute), mean, or median, which Viime executes via R packages called through OpenCPU. Write the completed, fully imputed data frame to an output CSV. Validation: the output CSV exists, has the same rows and columns as the input, and contains no remaining missing values.

## Related tools

- **Viime** (Web application implementing the Armitage et al. (2015) heuristic that classifies missing data as MNAR or MCAR and dispatches MNAR values to zero/half-minimum imputation) — girder/viime
- **R** (Hosts the custom viime package whose CSV-processing functions perform the imputation; specialized algorithms run as R packages)
- **OpenCPU** (Internal REST service bridging the Flask/Pandas backend to the R imputation functions) — https://www.opencpu.org/
- **Pandas** (Parses the raw input file and exchanges data frames with the R side (viime/opencpu.py)) — girder/viime

## Evaluation signals

- Output CSV exists and has exactly the same rows and columns as the input concentration table
- Output contains no remaining missing values (zero NaN/NA entries)
- Every metabolite classified MNAR has imputed entries equal to zero or exactly half the minimum observed concentration for that metabolite/group
- Group and factor columns are preserved unchanged relative to the input
- MCAR-classified metabolites (low per-group missing percentage) were NOT filled with zeros, confirming correct dispatch between MNAR and MCAR methods

## Limitations

- The MNAR/MCAR split is heuristic, based on the percentage of missing values per group per metabolite (Armitage et al., 2015); borderline missingness patterns may be misclassified
- Zero/half-minimum replacement distorts variance and correlation structure for MNAR metabolites, affecting downstream PCA and scaling choices
- R-backed imputation methods require the OpenCPU service (devops Docker container); the opencpu.py bridge is described as work-in-progress backend code
- MNAR imputation in Viime runs automatically once errors are corrected, so the user has limited control over per-metabolite overrides

## Evidence

- [abstract] Viime uses a sophisticated imputation strategy (Armitage, Godzien, Alonso-Herranz, López-Gonzálvez, & Barbas, 2015), heuristically classifying missing data as Missing Not At Random (MNAR) or Missing: "Viime uses a sophisticated imputation strategy (Armitage, Godzien, Alonso-Herranz, López-Gonzálvez, & Barbas, 2015), heuristically classifying missing data as Missing Not At Random (MNAR) or Missing"
- [abstract] Random Forest and K-Nearest Neighbor were implemented with the R packages missForest and impute, respectively.: "Random Forest and K-Nearest Neighbor were implemented with the R packages missForest and impute, respectively."
- [readme] This installation includes a custom R package called viime, which contains all functions that are exposed to the API server for CSV file processing.: [evidence span withheld — anchor status: not_found]
- [abstract] The intent of Viime package is to pick up the workflow after preprocessing and annotation and go all the way through statistical analysis and visualization.: "The intent of Viime package is to pick up the workflow after preprocessing and annotation and go all the way through statistical analysis and visualization."
