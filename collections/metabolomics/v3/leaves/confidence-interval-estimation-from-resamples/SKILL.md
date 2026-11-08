---
name: confidence-interval-estimation-from-resamples
description: Use when a metabolite–phenotype point estimate (e.g. confounder-adjusted
  Spearman correlation with BMI) from an MWAS model needs uncertainty quantification
  or validation via subject-level resampling with replacement.
when_to_use_negative:
- When the goal is a genome/metabolome-wide screen of associations rather than uncertainty
  quantification of already-estimated associations (use the package's MWAS association
  models instead)
- When the association model cannot be recomputed on resampled subjects, e.g. data
  lacking the confounder or phenotype columns needed to refit the partial-correlation
  model per replicate
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3334
tools:
- name: MWASTools
  role: Provides MWAS_bootstrapping, the exported function that resamples the confounder-adjusted
    association model and derives percentile-based confidence intervals for each metabolic
    feature
  repo: https://bioconductor.org/packages/MWASTools/
- name: R
  role: Runtime environment in which MWASTools (R version >= 3.4) and the bootstrapping
    workflow are executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btx477
    title: 'MWASTools: an R/bioconductor package for metabolome-wide association studies'
  source_content_hash: sha256:9a91b3480e5bd1e18bf0edb3a0685fe6d6252e491d86fa1ec3d72da3f59a2e0c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MWASTools
  - R
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# confidence-interval-estimation-from-resamples

## Summary

Validates metabolite–phenotype association models (e.g. BMI–metabolite Spearman correlations adjusted for Age, Gender, T2D) by non-parametric bootstrapping, as implemented in the MWASTools function MWAS_bootstrapping, yielding percentile-based confidence intervals per metabolic feature.

## When to use

Use when a metabolite–phenotype point estimate (e.g. confounder-adjusted Spearman correlation with BMI) from an MWAS model needs uncertainty quantification or validation via subject-level resampling with replacement.

## When NOT to use

- When the goal is a genome/metabolome-wide screen of associations rather than uncertainty quantification of already-estimated associations (use the package's MWAS association models instead)
- When the association model cannot be recomputed on resampled subjects, e.g. data lacking the confounder or phenotype columns needed to refit the partial-correlation model per replicate

## Inputs

- metabo_SE: a SummarizedExperiment/MultiAssayExperiment metabolomics data container
- Phenotype metadata with metabolite_id, disease_id (BMI), and confounder_ids (e.g. Age, Gender, T2D)
- assoc_method parameter (e.g. spearman) and iterations parameter (number of bootstrap resamples)

## Outputs

- Per-feature bootstrap estimate distributions
- Percentile-based confidence-interval bounds (lower/upper) per metabolic feature
- Validation table (e.g. bootstrap_validation.csv) listing point estimate, bootstrap mean, and CI bounds per feature

## How to apply

Load the metabolomics data as a SummarizedExperiment/MultiAssayExperiment container (metabo_SE) together with phenotype metadata holding metabolite_id, disease_id (BMI) and confounder_ids (Age, Gender, T2D). For each selected metabolic feature compute the association with BMI as a Spearman correlation adjusted for confounders via partial regression on the confounder residuals, matching the package's partial-correlation MWAS model. Resample study subjects with replacement for the user-set number of iterations (assoc_method and iterations are parameters of MWAS_bootstrapping), recomputing the confounder-adjusted estimate in every replicate. Collect the per-feature bootstrap distributions and derive percentile-based confidence intervals. Assemble a validation table with point estimate, bootstrap mean, and lower/upper CI bounds per feature; judging whether the CIs validate the reported associations is deferred to expert review.

## Related tools

- **MWASTools** (Provides MWAS_bootstrapping, the exported function that resamples the confounder-adjusted association model and derives percentile-based confidence intervals for each metabolic feature) — https://bioconductor.org/packages/MWASTools/
- **R** (Runtime environment in which MWASTools (R version >= 3.4) and the bootstrapping workflow are executed)

## Examples

```
MWAS_bootstrapping(metabo_SE, metabolite_id, disease_id = "BMI", confounder_ids = c("Age", "Gender", "T2D"), assoc_method = "spearman", iterations = 1000)
```

## Evaluation signals

- bootstrap_validation.csv exists and contains one row per selected metabolic feature
- Each feature row carries finite point estimate, bootstrap mean, and lower/upper CI fields
- CI bounds bracket the point estimate (lower <= point estimate <= upper) as expected for percentile intervals
- Number of resamples matches the user-set iterations parameter, and the point estimate equals the confounder-adjusted Spearman correlation from the original model
- Final accept/reject judgment on whether CIs validate the reported BMI associations is flagged for expert review rather than automated

## Limitations

- Bootstrapping validates stability/uncertainty of association estimates but does not itself decide whether reported BMI associations hold — such judgment is deferred to expert review
- Confounding factors such as age and gender, if not properly accounted for, can distort observed metabolite–condition associations; the bootstrap inherits any misspecification of the confounder adjustment
- Percentile CIs depend on the chosen number of iterations; too few resamples yields unstable interval bounds

## Evidence

- [intro] bootstrapping of association models; visualization of statistical outcomes: "Our package provides a versatile and user-friendly MWAS pipeline with the following key functionalities: quality control (QC) analysis; metabolite-phenotype association models (partial correlations,"
- [intro] metabolite-phenotype association models (partial correlations, generalized linear models) adjusted for epidemiological confounders (e.g. age or gender): "metabolite-phenotype association models (partial correlations, generalized linear models) adjusted for epidemiological confounders (e.g. age or gender)"
- [abstract] MWASTools is an R package designed to provide an integrated pipeline to analyse metabonomic data in large-scale epidemiological studies: "MWASTools is an R package designed to provide an integrated pipeline to analyse metabonomic data in large-scale epidemiological studies"
- [intro] univariate hypothesis testing with efficient handling of epidemiological confounders: "Here, we present a package to perform MWAS using univariate hypothesis testing with efficient handling of epidemiological confounders."
