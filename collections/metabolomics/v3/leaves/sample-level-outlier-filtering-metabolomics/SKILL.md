---
name: sample-level-outlier-filtering-metabolomics
description: Use when a Nightingale-format 1H-NMR metabolomics matrix (CSV/TSV) must
  be quality-controlled per sample before computing metaboAge, surrogate clinical
  variables, or published clinical scores with MiMIR.
when_to_use_negative:
- Input data are not Nightingale Health 1H-NMR metabolomics measurements (e.g. LC-MS
  or genotypes) — MiMIR's QC thresholds and MEAN/SD scaling assume the Nightingale
  platform.
- The goal is cohort-level calibration of surrogate values rather than sample QC —
  use MiMIR's plattCalibration instead.
- The metabolite matrix has already passed QCprep and only needs score projection
  via comp.CVD_score, comp.T2D_Ahola_Olli, or comp.mort_score.
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3272
tools:
- name: MiMIR
  role: R package providing QCprep, QCprep_surrogates, subset_samples_miss, subset_samples_zero,
    subset_samples_sd, and impute_miss for sample-level QC before score projection
  repo: https://github.com/DanieleBizzarri/MiMIR
- name: R
  role: Runtime environment (R 3.6+) in which MiMIR and its QC functions are executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btac388
    title: 'MiMIR: R-shiny application to infer risk factors and endpoints from Nightingale
      Health’s 1H-NMR metabolomics data'
  source_content_hash: sha256:18787df245ea03f1513b0b0c711d1c8075c344d0342f64da8f2da35912534653
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DanieleBizzarri/MiMIR
  tools:
  - MiMIR
  - R
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# Sample-level outlier filtering for Nightingale 1H-NMR metabolomics

## Summary

Applies MiMIR's QC pre-processing to a Nightingale Health 1H-NMR metabolomics matrix, removing samples with excessive missing/zero values and SD-based outliers before projecting pre-trained metabolic scores or surrogates.

## When to use

Use when a Nightingale-format 1H-NMR metabolomics matrix (CSV/TSV) must be quality-controlled per sample before computing metaboAge, surrogate clinical variables, or published clinical scores with MiMIR.

## When NOT to use

- Input data are not Nightingale Health 1H-NMR metabolomics measurements (e.g. LC-MS or genotypes) — MiMIR's QC thresholds and MEAN/SD scaling assume the Nightingale platform.
- The goal is cohort-level calibration of surrogate values rather than sample QC — use MiMIR's plattCalibration instead.
- The metabolite matrix has already passed QCprep and only needs score projection via comp.CVD_score, comp.T2D_Ahola_Olli, or comp.mort_score.

## Inputs

- Nightingale Health 1H-NMR metabolomics matrix in CSV or TSV with example-matching column names
- MiMIR bundled parameters: PARAM_metaboAge and PARAM_surrogates (including MEAN/SD values)
- QC thresholds Nmax_miss and Nmax_zero

## Outputs

- QC-filtered metabolite matrix (samples exceeding Nmax_miss/Nmax_zero or SD-based outlier criteria removed)
- Imputed, scaled matrix ready for apply.fit / apply.fit_surro score projection
- Log of removed samples (missing-value, zero-value, and SD-outlier exclusions)

## How to apply

Load the metabolite matrix with example-matching column names and the bundled parameter objects (PARAM_metaboAge for the metaboAge model, PARAM_surrogates for the surrogate models). Run QCprep for metaboAge or QCprep_surrogates for the surrogate models: drop samples whose number of missing values exceeds Nmax_miss and whose number of zero values exceeds Nmax_zero (via subset_samples_miss and subset_samples_zero), then flag SD-based outliers with subset_samples_sd using the bundled MEAN/SD values. Impute the remaining missing values with impute_miss after scaling. Rationale: each pre-trained model is sensitive to missing, zero, and out-of-distribution samples, so these thresholds ensure faithful score derivation. QC is run per model in parallel so each score retains its own normalization as published.

## Related tools

- **MiMIR** (R package providing QCprep, QCprep_surrogates, subset_samples_miss, subset_samples_zero, subset_samples_sd, and impute_miss for sample-level QC before score projection) — https://github.com/DanieleBizzarri/MiMIR
- **R** (Runtime environment (R 3.6+) in which MiMIR and its QC functions are executed)

## Examples

```
library(MiMIR); mat <- read.csv("metabolites.csv"); qc <- QCprep(mat, PARAM_metaboAge, Nmax_miss=1, Nmax_zero=1, quiet=FALSE)
```

## Evaluation signals

- After QCprep/QCprep_surrogates, no retained sample has more than Nmax_miss missing values or Nmax_zero zero values.
- All retained samples fall within the SD-based outlier bounds defined by the bundled MEAN/SD values used in subset_samples_sd.
- The filtered matrix contains no unimputed NA cells after impute_miss.
- Downstream score projection yields the full set of 24 score/surrogate columns populated for all retained samples.
- Column names of the input matrix matched each model's required feature IDs via prep_met_for_scores (with plus-one offset where required) before QC.

## Limitations

- Default Nmax_miss and Nmax_zero thresholds may differ between the metaboAge model (QCprep/PARAM_metaboAge) and the surrogate models (QCprep_surrogates/PARAM_surrogates), so a sample can pass QC for one model and be dropped for another.
- SD-based outlier detection depends on the bundled MEAN/SD reference values; cohorts far outside the reference population (e.g. BBMRI-NL) may have excessive exclusions.
- Samples removed during QC are permanently excluded from all downstream score calculations for that model.

## Evidence

- [methods] QC pre-processing thresholds and functions: "`QCprep(mat, PARAM_metaboAge, quiet, Nmax_zero, Nmax_miss)`"
- [methods] SD-based outlier removal: "`subset_samples_sd(x, MEAN, SD, quiet)`"
- [results] Parallel per-model pre-processing rationale: [evidence span withheld — anchor status: not_found]
- [readme] Input format requirement: "Upload your metabolites with the same column names as in the example dataset (both CSV and TSV are accepted)."
