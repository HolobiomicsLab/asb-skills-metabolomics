---
name: binary-phenotype-binarization
description: Use when a phenotypic dataset with mixed continuous/categorical clinical
  variables (e.g. MiMIR's synthetic_phenotypic_dataset) must be binarized before computing
  ROC curves and AUCs for metabolomics-based surrogate scores.
when_to_use_negative:
- The phenotypic variables are already binary case/control labels — re-running binarization
  is unnecessary
- The goal is metabolite QC or score projection (use QCprep / QCprep_surrogates /
  calculate_surrogate_scores instead of phenotype binarization)
- The surrogate values must be calibrated to a target cohort (use MiMIR::plattCalibration,
  which operates on continuous surrogate values, not binarized phenotypes)
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_3172
tools:
- name: MiMIR
  role: Provides binarize_all_pheno and the downstream roc_surro / LOBOV_accuracies
    functions that consume the binarized phenotypes
  repo: https://github.com/DanieleBizzarri/MiMIR
- name: pROC
  role: ROC/AUC engine underlying MiMIR::roc_surro, applied to surrogate scores versus
    binarized phenotypes
- name: R
  role: Runtime environment (R 3.6+) in which MiMIR and pROC are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btac388
    title: 'MiMIR: R-shiny application to infer risk factors and endpoints from Nightingale
      Health’s 1H-NMR metabolomics data'
  source_content_hash: sha256:2fa3f49c36598a8c261a81cfe1ca508cfda306707ed9dae9d2e3105c60cb9ef1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DanieleBizzarri/MiMIR
  tools:
  - MiMIR
  - R
  - pROC
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# binary-phenotype-binarization

## Summary

Converts uploaded phenotypic variables (e.g. from the MiMIR phenotypic dataset) into binary 0/1 labels using MiMIR::binarize_all_pheno so that metabolomics-based surrogate scores can be evaluated against binary case/control status via ROC/AUC analysis.

## When to use

Use when a phenotypic dataset with mixed continuous/categorical clinical variables (e.g. MiMIR's synthetic_phenotypic_dataset) must be binarized before computing ROC curves and AUCs for metabolomics-based surrogate scores.

## When NOT to use

- The phenotypic variables are already binary case/control labels — re-running binarization is unnecessary
- The goal is metabolite QC or score projection (use QCprep / QCprep_surrogates / calculate_surrogate_scores instead of phenotype binarization)
- The surrogate values must be calibrated to a target cohort (use MiMIR::plattCalibration, which operates on continuous surrogate values, not binarized phenotypes)

## Inputs

- Phenotypic dataset (CSV/TSV, e.g. MiMIR synthetic_phenotypic_dataset) with variables matching example-dataset naming conventions
- Projected surrogate scores from the upstream metabolomics (Nightingale Health 1H-NMR) projection step

## Outputs

- Binarized (0/1) phenotypic matrix aligned with the surrogate scores, suitable input for MiMIR::roc_surro and MiMIR::roc_surro_subplots

## How to apply

Load the phenotypic dataset alongside the projected surrogate scores from the upstream Nightingale Health 1H-NMR metabolomics projection step, then run MiMIR::binarize_all_pheno(data) to binarize every phenotypic variable into case/control form. Binarization is a prerequisite for MiMIR::roc_surro, which computes pROC-based ROC curves and AUCs of each surrogate score against its binarized phenotype. This supports the downstream comparison against Leave One Biobank Out Validation (LOBOV) AUCs within BBMRI-nl via MiMIR::LOBOV_accuracies (e.g. diabetes surrogate AUC 0.85 vs mean LOBOV AUC 0.8). Only binarize variables whose corresponding surrogate phenotype is available; the reference functions QCprep_surrogates and calculate_surrogate_scores expect the binary phenotype names (bin_names) to align with the surrogate model parameters (PARAM_surrogates).

## Related tools

- **MiMIR** (Provides binarize_all_pheno and the downstream roc_surro / LOBOV_accuracies functions that consume the binarized phenotypes) — https://github.com/DanieleBizzarri/MiMIR
- **pROC** (ROC/AUC engine underlying MiMIR::roc_surro, applied to surrogate scores versus binarized phenotypes)
- **R** (Runtime environment (R 3.6+) in which MiMIR and pROC are executed)

## Examples

```
bin_pheno <- MiMIR::binarize_all_pheno(pheno); auc_tab <- MiMIR::roc_surro(scores, bin_pheno)
```

## Evaluation signals

- Every binarized phenotype column contains only 0/1 values after MiMIR::binarize_all_pheno
- Every surrogate whose binary phenotype is available yields a ROC/AUC value in [0, 1] from MiMIR::roc_surro
- Each surrogate row appears in the LOBOV comparison table with both the locally computed AUC and the packaged acc_LOBOV reference AUC
- Binarized phenotype names align with the bin_names expected by calculate_surrogate_scores/PARAM_surrogates

## Limitations

- AUC comparison to LOBOV is only possible when the corresponding phenotypic variable is uploaded; missing phenotypes cannot be binarized or evaluated
- The App expects phenotypic variables to follow the same column naming as the example (synthetic) dataset; mismatched names will fail
- No discussion-section content or changelog is available in the source, so edge-case behavior of binarize_all_pheno (e.g. multi-level categorical handling) is not documented here

## Evidence

- [methods] binarize_all_pheno function signature: "`binarize_all_pheno(data)`"
- [methods] surrogate scoring consumes binary phenotype names and ROC: "`calculate_surrogate_scores(met, pheno, PARAM_surrogates, bin_names, Nmax_miss, Nmax_zero, post, roc, quiet)`"
- [readme] pROC is an imaging/statistics dependency used for ROC analysis: "if (!require("pROC")) install.packages("pROC")"
- [readme] synthetic phenotypic dataset available for testing: "the metabolic synthetic dataset, the phenotypic synthetic dataset. These example dataset can be used to test the App"
