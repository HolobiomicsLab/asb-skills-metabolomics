---
name: bootstrap-resampling-model-validation
description: Use when a metabolite-phenotype association model (e.g. Spearman/partial
  correlation of metabolic features against a disease variable like BMI, adjusted
  for confounders such as Age, Gender, T2D) needs stability assessment via resampling
  of study subjects.
when_to_use_negative:
- When the goal is the primary metabolome-wide association scan itself (partial correlations
  or generalized linear models) rather than validation of an already-computed association
  model — use the package's MWAS model functions instead.
- When associations must not account for epidemiological confounders such as age and
  gender is not the concern; MWAS_bootstrapping is designed around confounder-adjusted
  univariate models.
- When sample size is too small for stable resampling with replacement of study subjects
  — bootstrap CIs will be unreliable.
edam_operation: http://edamontology.org/operation_3657
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_0203
tools:
- name: MWASTools
  role: R/Bioconductor package exposing MWAS_bootstrapping(metabo_SE, metabolite_id,
    disease_id, confounder_ids, assoc_method, iterations) for non-parametric bootstrapping
    of confounder-adjusted association models
  repo: https://bioconductor.org/packages/MWASTools/
- name: R
  role: Runtime environment in which MWASTools and the bootstrapping workflow are
    executed (R >= 3.4)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btx477
    title: 'MWASTools: an R/bioconductor package for metabolome-wide association studies'
  source_content_hash: sha256:65859f394163adddc641b9dcac4d873187a955935d14360efd9d3d966d32f8ec
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MWASTools
  - R
  edam_operation: http://edamontology.org/operation_3657
  edam_topics:
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# bootstrap-resampling-model-validation

## Summary

Validate metabolite-phenotype association models (e.g. confounder-adjusted Spearman correlations of metabolic features against BMI) by non-parametric bootstrapping, as implemented in the MWASTools function MWAS_bootstrapping.

## When to use

Use when a metabolite-phenotype association model (e.g. Spearman/partial correlation of metabolic features against a disease variable like BMI, adjusted for confounders such as Age, Gender, T2D) needs stability assessment via resampling of study subjects.

## When NOT to use

- When the goal is the primary metabolome-wide association scan itself (partial correlations or generalized linear models) rather than validation of an already-computed association model — use the package's MWAS model functions instead.
- When associations must not account for epidemiological confounders such as age and gender is not the concern; MWAS_bootstrapping is designed around confounder-adjusted univariate models.
- When sample size is too small for stable resampling with replacement of study subjects — bootstrap CIs will be unreliable.

## Inputs

- metabo_SE: a SummarizedExperiment/MultiAssayExperiment metabolomics data container
- Phenotype metadata with metabolite_id, disease_id (e.g. BMI), and confounder_ids (e.g. Age, Gender, T2D)
- assoc_method (association model, e.g. 'spearman') and number of bootstrap iterations

## Outputs

- bootstrap_validation.csv: per-feature table with point estimate, bootstrap mean, and lower/upper percentile confidence-interval bounds
- Per-feature bootstrap estimate distributions

## How to apply

Load the metabolomics data container (metabo_SE, a SummarizedExperiment/MultiAssayExperiment object) with phenotype metadata carrying metabolite_id, disease_id (BMI) and confounder_ids (Age, Gender, T2D). For each selected metabolic feature, compute the association with the disease variable as a Spearman correlation adjusted for confounders, using partial regression on confounder residuals as in the package's partial-correlation MWAS model. Resample the study subjects with replacement for a user-set number of iterations (assoc_method, e.g. 'spearman', and iterations are user parameters), recomputing the confounder-adjusted estimate in every replicate. Collect the per-feature bootstrap estimate distributions and derive percentile-based confidence intervals. Assemble a validation table of point estimate, bootstrap mean, and lower/upper CI bounds per feature, and write it to bootstrap_validation.csv. Judgment on whether the CIs validate the reported associations is deferred to expert review — the skill's deliverable is the finite-estimate, per-feature CI table.

## Related tools

- **MWASTools** (R/Bioconductor package exposing MWAS_bootstrapping(metabo_SE, metabolite_id, disease_id, confounder_ids, assoc_method, iterations) for non-parametric bootstrapping of confounder-adjusted association models) — https://bioconductor.org/packages/MWASTools/
- **R** (Runtime environment in which MWASTools and the bootstrapping workflow are executed (R >= 3.4))

## Examples

```
MWAS_bootstrapping(metabo_SE, metabolite_id, disease_id = "BMI", confounder_ids = c("Age", "Gender", "T2D"), assoc_method = "spearman", iterations = 1000)
```

## Evaluation signals

- bootstrap_validation.csv exists and contains one row per selected metabolic feature
- All estimate, bootstrap mean, and CI bound fields in the output CSV are finite (no NaN/Inf)
- Upper confidence-interval bound >= bootstrap mean >= point-estimate ordering is coherent for each feature's reported interval
- Number of bootstrap replicates per feature matches the configured iterations parameter
- Any judgment that the CIs 'validate' the BMI associations is deferred to expert review rather than asserted by the pipeline

## Limitations

- The workflow only produces per-feature bootstrap CIs; whether those CIs validate the reported BMI associations requires expert review and is not decided computationally.
- Bootstrapping resamples the study subjects with replacement, so estimates depend on the available sample size and may be unstable for small cohorts.
- Multivariate models that fail to account for confounding factors such as age and gender can distort observed metabolite-condition associations; bootstrapping an unadjusted model would inherit that bias.

## Evidence

- [intro] bootstrapping of association models: "bootstrapping of association models; visualization of statistical outcomes"
- [intro] package provides versatile MWAS pipeline including bootstrapping: "Our package provides a versatile and user-friendly MWAS pipeline with the following key functionalities: quality control (QC) analysis; metabolite-phenotype association models (partial correlations,"
- [intro] association models adjusted for epidemiological confounders: "metabolite-phenotype association models (partial correlations, generalized linear models) adjusted for epidemiological confounders (e.g. age or gender)"
- [intro] confounders can distort observed associations: "However, a major limitation of these multivariate models from the epidemiological perspective is that they do not properly account for cofounding factors (e.g. age, gender), which might distort the"
- [abstract] MWASTools key functionalities include MWAS with partial correlations and GLMs: "Key functionalities of our package include: quality control analysis; metabolome-wide association analysis using various models (partial correlations, generalized linear models); visualization of"
