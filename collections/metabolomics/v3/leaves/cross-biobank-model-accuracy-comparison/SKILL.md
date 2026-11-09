---
name: cross-biobank-model-accuracy-comparison
description: Use when projected MiMIR surrogate scores must be validated against previously
  established performance (LOBOV AUCs from BBMRI-nl), e.g. checking whether a locally
  computed diabetes surrogate AUC agrees with the reference cohort AUC.
when_to_use_negative:
- Only metabolite measurements are available and no phenotypic variables can be uploaded
  — AUCs cannot be computed without the corresponding binary phenotype for each surrogate
- The goal is to calibrate surrogate values to the local dataset (MiMIR::plattCalibration)
  rather than to evaluate and compare model accuracy across biobanks
- Input metabolites have not yet been QC-prepped and projected into surrogate scores
  (use QCprep_surrogates / calculate_surrogate_scores first)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MiMIR
  role: Provides roc_surro, roc_surro_subplots, binarize_all_pheno and LOBOV_accuracies
    functions plus the acc_LOBOV reference dataset for computing and comparing surrogate
    AUCs
  repo: github.com/DanieleBizzarri/MiMIR
- name: pROC
  role: ROC-curve library underlying MiMIR::roc_surro used to compute surrogate AUCs
    against binarized phenotypes
- name: R
  role: Runtime environment for the MiMIR package and comparison workflow (R version
    3.6+)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btac388
    title: 'MiMIR: R-shiny application to infer risk factors and endpoints from Nightingale
      Health’s 1H-NMR metabolomics data'
  source_content_hash: sha256:71b01800aab01fab055d8bf5ec8db4d96a2f5e390466c43f9bee4425be323204
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MiMIR
  - R
  - pROC
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cross-biobank-model-accuracy-comparison

## Summary

Validates MiMIR-computed 1H-NMR metabolomics surrogate scores for clinical variables by computing local AUCs and comparing them against the packaged LOBOV (Leave One Biobank Out Validation) reference accuracies from BBMRI-nl, surfacing cross-biobank agreement for each surrogate model.

## When to use

Use when projected MiMIR surrogate scores must be validated against previously established performance (LOBOV AUCs from BBMRI-nl), e.g. checking whether a locally computed diabetes surrogate AUC agrees with the reference cohort AUC.

## When NOT to use

- Only metabolite measurements are available and no phenotypic variables can be uploaded — AUCs cannot be computed without the corresponding binary phenotype for each surrogate
- The goal is to calibrate surrogate values to the local dataset (MiMIR::plattCalibration) rather than to evaluate and compare model accuracy across biobanks
- Input metabolites have not yet been QC-prepped and projected into surrogate scores (use QCprep_surrogates / calculate_surrogate_scores first)

## Inputs

- Projected surrogate scores from the upstream MiMIR projection/calibration step
- Phenotypic dataset with clinical variables (e.g. MiMIR synthetic_phenotypic_dataset)
- Binarized phenotypic variables from MiMIR::binarize_all_pheno
- Packaged acc_LOBOV reference dataset of BBMRI-nl validation AUCs

## Outputs

- Per-surrogate ROC curves and AUC values (pROC-based, via MiMIR::roc_surro)
- ROC curve panel figures (MiMIR::roc_surro_subplots)
- Comparison table of local AUCs vs LOBOV AUCs written to surrogate_auc_comparison.csv

## How to apply

Load the projected surrogate scores from the upstream projection step together with a phenotypic dataset (e.g. the MiMIR synthetic_phenotypic_dataset) and binarize all phenotypic variables with MiMIR::binarize_all_pheno. For every surrogate whose binary phenotype is available, compute the ROC curve and its AUC with MiMIR::roc_surro (pROC-based) on surrogate values versus the binarized phenotype, and render per-surrogate ROC panels with MiMIR::roc_surro_subplots for visual inspection. Then load the packaged acc_LOBOV reference dataset and build the local-AUC vs LOBOV-AUC comparison with MiMIR::LOBOV_accuracies, writing the result to surrogate_auc_comparison.csv. The rationale is that a surrogate whose AUC closely matches the reference (e.g. diabetes surrogate AUC 0.85 in LLS_PAROFFs vs mean LOBOV AUC 0.8) generalizes across biobanks, so agreement is judged for biological plausibility by expert review.

## Related tools

- **MiMIR** (Provides roc_surro, roc_surro_subplots, binarize_all_pheno and LOBOV_accuracies functions plus the acc_LOBOV reference dataset for computing and comparing surrogate AUCs) — github.com/DanieleBizzarri/MiMIR
- **pROC** (ROC-curve library underlying MiMIR::roc_surro used to compute surrogate AUCs against binarized phenotypes)
- **R** (Runtime environment for the MiMIR package and comparison workflow (R version 3.6+))

## Examples

```
b_pheno <- MiMIR::binarize_all_pheno(pheno); roc_res <- MiMIR::roc_surro(surrogates, b_pheno); auc_comparison <- MiMIR::LOBOV_accuracies(roc_res, acc_LOBOV)
```

## Evaluation signals

- Every AUC reported in surrogate_auc_comparison.csv is a numeric value in the range [0, 1]
- Each surrogate row is present in the comparison table together with its LOBOV reference AUC for side-by-side comparison
- Local AUCs are biologically plausible relative to the LOBOV reference (e.g. diabetes surrogate AUC 0.85 in LLS_PAROFFs vs mean LOBOV AUC 0.8), confirmed by expert_review
- ROC curve panels were rendered for every surrogate whose binary phenotype was available, for visual inspection
- Phenotypic variables were binarized (binarize_all_pheno) before AUC computation, not used as continuous values

## Limitations

- Comparison is only possible for surrogates whose corresponding phenotypic variable is uploaded and binarizable; missing phenotypes leave AUC gaps
- LOBOV references are specific to BBMRI-nl cohorts, so local disagreement may reflect population differences rather than model failure and requires expert review for biological plausibility
- The MiMIR App/package requires R version 3.6+ and the pROC dependency for ROC computation; no changelog is available for tracking reference-accuracy updates

## Evidence

- [readme] pROC dependency in MiMIR: "if (!require("pROC")) install.packages("pROC")"
- [readme] MiMIR purpose: project and calibrate metabolic surrogate values: "It
allows to easily explore new metabolomics measurements assayed by
Nightingale Health; project previously published metabolic scores; and
calibrate the metabolic surrogate values to a desired"
- [readme] Example phenotypic dataset for testing variable naming: "These example dataset can be used to test
the App and to understand how the variables in your own dataset should
be named."
- [methods] Surrogate score computation call: "`calculate_surrogate_scores(met, pheno, PARAM_surrogates, bin_names, Nmax_miss, Nmax_zero, post, roc, quiet)`"
- [results] Diabetes surrogate AUC vs LOBOV reference: [evidence span withheld — anchor status: not_found]
