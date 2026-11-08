---
name: metabolic-feature-association-testing
description: Use when a metabolome-wide association (partial correlation or GLM) between
  a metabolic feature and a phenotype such as BMI needs confidence intervals or robustness
  validation by resampling, with confounders like Age, Gender, T2D adjusted.
when_to_use_negative:
- The goal is a metabolome-wide screening p-value set rather than stability assessment
  of an already-selected feature's association — use the package's MWAS association
  models (partial correlations, generalized linear models) instead
- The data container is not a SummarizedExperiment/MultiAssayExperiment object with
  phenotype metadata for metabolite_id, disease_id and confounder_ids
- Quality control, STOCSY metabolite assignment, or network/visualization tasks are
  the goal — these are separate MWASTools functionalities
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0092
tools:
- name: MWASTools
  role: provides MWAS_bootstrapping() and the SummarizedExperiment/MultiAssayExperiment
    data container for confounder-adjusted metabolite–phenotype association models
  repo: https://bioconductor.org/packages/MWASTools/
- name: R
  role: implementation environment for the bootstrapping workflow (R version >= 3.4)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btx477
    title: 'MWASTools: an R/bioconductor package for metabolome-wide association studies'
  source_content_hash: sha256:c4fb242ad8108cb31db59912a3de18d60348cb3cc2087de619fc8932f359ffc6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MWASTools
  - R
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# metabolic-feature-association-testing

## Summary

Validates metabolite–phenotype association models (e.g. BMI vs metabolic features) by non-parametric bootstrapping with MWASTools' MWAS_bootstrapping(), which resamples study subjects with replacement and recomputes a confounder-adjusted association estimate per iteration.

## When to use

Use when a metabolome-wide association (partial correlation or GLM) between a metabolic feature and a phenotype such as BMI needs confidence intervals or robustness validation by resampling, with confounders like Age, Gender, T2D adjusted.

## When NOT to use

- The goal is a metabolome-wide screening p-value set rather than stability assessment of an already-selected feature's association — use the package's MWAS association models (partial correlations, generalized linear models) instead
- The data container is not a SummarizedExperiment/MultiAssayExperiment object with phenotype metadata for metabolite_id, disease_id and confounder_ids
- Quality control, STOCSY metabolite assignment, or network/visualization tasks are the goal — these are separate MWASTools functionalities

## Inputs

- metabo_SE: metabolomics data container (SummarizedExperiment/MultiAssayExperiment) with feature assays
- phenotype metadata: metabolite_id, disease_id (BMI), confounder_ids (Age, Gender, T2D)
- assoc_method: association model choice (e.g. 'spearman')
- iterations: number of bootstrap resampling iterations

## Outputs

- bootstrap_validation.csv: per-feature table with point estimate, bootstrap mean, and lower/upper percentile-based confidence-interval bounds
- per-feature bootstrap estimate distributions

## How to apply

Load the metabolomics data as a SummarizedExperiment/MultiAssayExperiment object (metabo_SE) whose phenotype metadata carries metabolite_id, disease_id (BMI) and confounder_ids (Age, Gender, T2D). For the selected metabolic feature, compute the confounder-adjusted Spearman (partial-correlation) point estimate as in the package's MWAS model, regressing on confounder residuals. Call MWAS_bootstrapping() with assoc_method (e.g. 'spearman') and a user-set number of bootstrap iterations; subjects are resampled with replacement and the adjusted association estimate is recomputed in every replicate. Collect the per-feature bootstrap estimate distribution and derive percentile-based confidence intervals from it. Assemble a validation table (point estimate, bootstrap mean, lower/upper CI bounds) per feature and write it to bootstrap_validation.csv.

## Related tools

- **MWASTools** (provides MWAS_bootstrapping() and the SummarizedExperiment/MultiAssayExperiment data container for confounder-adjusted metabolite–phenotype association models) — https://bioconductor.org/packages/MWASTools/
- **R** (implementation environment for the bootstrapping workflow (R version >= 3.4))

## Examples

```
MWAS_bootstrapping(metabo_SE, metabolite_id, disease_id = "BMI", confounder_ids = c("Age", "Gender", "T2D"), assoc_method = "spearman", iterations = n_boot)
```

## Evaluation signals

- bootstrap_validation.csv exists and contains one row per analyzed metabolic feature
- each row has finite estimate, bootstrap-mean and lower/upper CI fields (no NA/Inf)
- confidence intervals are percentile-based from the bootstrap estimate distribution, not normal-approximation
- bootstrap mean is consistent with the point estimate within expected sampling variability for the chosen iteration count
- final judgment on whether CIs validate reported BMI associations is deferred to expert review rather than auto-declared

## Limitations

- Bootstrapping only resamples subjects with replacement from the same study; it cannot overcome inadequate confounder adjustment, which the article notes can distort observed metabolite–condition associations
- The workflow computes a confounder-adjusted Spearman (partial-correlation) estimate; other association models (e.g. generalized linear models) would need separate handling
- Whether the bootstrap CIs actually validate reported BMI associations is deferred to expert review, not decided automatically
- MWASTools requires R version >= 3.4

## Evidence

- [abstract] MWASTools is an R package designed to provide an integrated pipeline to analyse metabonomic data in large-scale epidemiological studies: "MWASTools is an R package designed to provide an integrated pipeline to analyse metabonomic data in large-scale epidemiological studies"
- [intro] Our package provides a versatile and user-friendly MWAS pipeline with the following key functionalities: quality control (QC) analysis; metabolite-phenotype association models (partial correlations,: "Our package provides a versatile and user-friendly MWAS pipeline with the following key functionalities: quality control (QC) analysis; metabolite-phenotype association models (partial correlations,"
- [intro] bootstrapping of association models; visualization of statistical outcomes: "bootstrapping of association models; visualization of statistical outcomes"
- [abstract] The MWASTools R package is implemented in R (version > =3.4): "The MWASTools R package is implemented in R (version > =3.4)"
