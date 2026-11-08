---
name: roc-auc-evaluation-of-surrogate-scores
description: Use when metabolomics-based surrogate scores (e.g. for diabetes) have
  been projected from Nightingale Health 1H-NMR metabolomics and the corresponding
  binary phenotypic variables are available to benchmark local AUCs against LOBOV
  reference performance.
when_to_use_negative:
- The binary phenotype corresponding to a surrogate is unavailable, since roc_surro
  requires the binarized phenotypic variable as ground truth
- The metabolomics data have not yet been projected/calibrated through MiMIR's upstream
  scoring step, so no surrogate scores exist to evaluate
- The goal is continuous-variable performance (e.g. correlation or RMSE) rather than
  binary classification discrimination measured by AUC
edam_operation: http://edamontology.org/operation_3899
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MiMIR
  role: Provides roc_surro (pROC-based AUC computation), roc_surro_subplots, binarize_all_pheno,
    LOBOV_accuracies and the acc_LOBOV reference dataset
  repo: github.com/DanieleBizzarri/MiMIR
- name: pROC
  role: Underlying ROC curve and AUC statistics library used by MiMIR's roc_surro
- name: R
  role: Runtime environment for the MiMIR package (R version 3.6+)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btac388
    title: 'MiMIR: R-shiny application to infer risk factors and endpoints from Nightingale
      Health’s 1H-NMR metabolomics data'
  source_content_hash: sha256:ab1427bd7ef58ffa98d9a9f702712fed1e3cd4fdaa8cf07da93cdaf863c5db69
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MiMIR
  - R
  - pROC
  edam_operation: http://edamontology.org/operation_3899
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# roc-auc-evaluation-of-surrogate-scores

## Summary

Validates MiMIR's metabolomics-based surrogate scores for clinical variables by computing pROC-based AUCs against binarized phenotypes and comparing them side-by-side with the packaged LOBOV (Leave One Biobank Out Validation) reference AUCs from BBMRI-nl.

## When to use

Use when metabolomics-based surrogate scores (e.g. for diabetes) have been projected from Nightingale Health 1H-NMR metabolomics and the corresponding binary phenotypic variables are available to benchmark local AUCs against LOBOV reference performance.

## When NOT to use

- The binary phenotype corresponding to a surrogate is unavailable, since roc_surro requires the binarized phenotypic variable as ground truth
- The metabolomics data have not yet been projected/calibrated through MiMIR's upstream scoring step, so no surrogate scores exist to evaluate
- The goal is continuous-variable performance (e.g. correlation or RMSE) rather than binary classification discrimination measured by AUC

## Inputs

- Projected surrogate scores matrix from the upstream metabolomics projection step (Nightengale Health 1H-NMR metabolomics)
- Phenotypic dataset (e.g. MiMIR synthetic_phenotypic_dataset) containing the clinically relevant variables
- Packaged acc_LOBOV reference dataset of Leave One Biobank Out Validation AUCs within BBMRI-nl

## Outputs

- Per-surrogate ROC curve panels for visual inspection
- AUC values per surrogate (numeric, in [0, 1])
- Comparison table of local AUCs versus LOBOV reference AUCs written to surrogate_auc_comparison.csv

## How to apply

Load the projected surrogate scores together with the phenotypic dataset (e.g. MiMIR's synthetic_phenotypic_dataset) and binarize all phenotypic variables with MiMIR::binarize_all_pheno. For each surrogate whose binary phenotype is available, compute the ROC curve and AUC using MiMIR::roc_surro, which is pROC-based, using the surrogate values versus the binarized phenotype. Render per-surrogate ROC panels with MiMIR::roc_surro_subplots for visual inspection. Load the packaged acc_LOBOV reference dataset and build the local-versus-LOBOV comparison with MiMIR::LOBOV_accuracies, then write the table to surrogate_auc_comparison.csv; agreement with BBMRI-nl reference AUCs (e.g. diabetes surrogate AUC 0.85 vs mean LOBOV AUC 0.8) is reviewed by experts for biological plausibility.

## Related tools

- **MiMIR** (Provides roc_surro (pROC-based AUC computation), roc_surro_subplots, binarize_all_pheno, LOBOV_accuracies and the acc_LOBOV reference dataset) — github.com/DanieleBizzarri/MiMIR
- **pROC** (Underlying ROC curve and AUC statistics library used by MiMIR's roc_surro)
- **R** (Runtime environment for the MiMIR package (R version 3.6+))

## Examples

```
aucs <- MiMIR::roc_surro(surro, pheno_bin); MiMIR::LOBOV_accuracies(aucs, acc_LOBOV)
```

## Evaluation signals

- Every AUC reported in surrogate_auc_comparison.csv is a numeric value in the range [0, 1]
- Each surrogate row is present with its LOBOV reference AUC for side-by-side comparison
- Local AUCs are biologically plausible relative to the LOBOV reference (e.g. diabetes surrogate AUC 0.85 in LLS_PAROFFs vs mean LOBOV AUC 0.8), as judged by expert review
- ROC curve panels are rendered for every surrogate evaluated and are inspectable via roc_surro_subplots
- All phenotypic variables were binarized with binarize_all_pheno before AUC computation

## Limitations

- Comparison is only possible for surrogates whose corresponding binary phenotype has been uploaded; missing phenotypes cannot be validated locally
- The LOBOV benchmark is specific to BBMRI-nl cohorts, so AUC agreement with other populations may differ
- MiMIR expects column names matching the example datasets (CSV/TSV upload) and R version 3.6+

## Evidence

- [methods] Surrogate scoring pipeline with ROC evaluation entry point: "`calculate_surrogate_scores(met, pheno, PARAM_surrogates, bin_names, Nmax_miss, Nmax_zero, post, roc, quiet)`"
- [readme] pROC is a required dependency for ROC/AUC analysis: "if (!require("pROC")) install.packages("pROC")"
- [readme] Synthetic datasets for testing the app workflow: "the metabolic synthetic dataset, the phenotypic synthetic dataset. These example dataset can be used to test the App"
- [intro] MiMIR supports projecting and calibrating metabolic surrogate scores: "project previously published metabolic scores; and calibrate the metabolic surrogate values to a desired dataset"
