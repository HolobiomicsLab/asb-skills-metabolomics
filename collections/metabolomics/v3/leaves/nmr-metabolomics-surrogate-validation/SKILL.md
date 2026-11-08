---
name: nmr-metabolomics-surrogate-validation
description: Use when projected surrogate scores (e.g. diabetes, mortality, MetaboAge-adjacent
  clinical surrogates) from Nightingale Health 1H-NMR metabolomics must be quality-checked
  against available phenotypes and published LOBOV AUC references.
when_to_use_negative:
- No corresponding binarized phenotypic variable is available for any surrogate, so
  no AUC can be computed locally
- The goal is score projection or Platt calibration of surrogate values (plattCalibration)
  rather than validation against phenotypes
- Input metabolomics data has not yet passed QCprep/QCprep_surrogates and score projection,
  so no surrogate scores exist to validate
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: MiMIR
  role: Provides roc_surro, roc_surro_subplots, binarize_all_pheno, LOBOV_accuracies
    and the acc_LOBOV reference dataset used to validate surrogate scores
  repo: https://github.com/DanieleBizzarri/MiMIR
- name: pROC
  role: R package underlying the ROC-curve and AUC computation in roc_surro
- name: R
  role: Runtime environment (R version 3.6+) in which the MiMIR validation functions
    are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btac388
    title: 'MiMIR: R-shiny application to infer risk factors and endpoints from Nightingale
      Health’s 1H-NMR metabolomics data'
  source_content_hash: sha256:4ee53ffaebafda1b6fed93f9c232f32319b24915c8c540723d52218e5d322b85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DanieleBizzarri/MiMIR
  tools:
  - MiMIR
  - R
  - pROC
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# nmr-metabolomics-surrogate-validation

## Summary

Validate MiMIR-projected metabolomics surrogate scores against binarized clinical phenotypes by computing ROC AUCs (pROC-based) and comparing them to Leave-One-Biobank-Out Validation (LOBOV) reference AUCs from BBMRI-nl.

## When to use

Use when projected surrogate scores (e.g. diabetes, mortality, MetaboAge-adjacent clinical surrogates) from Nightingale Health 1H-NMR metabolomics must be quality-checked against available phenotypes and published LOBOV AUC references.

## When NOT to use

- No corresponding binarized phenotypic variable is available for any surrogate, so no AUC can be computed locally
- The goal is score projection or Platt calibration of surrogate values (plattCalibration) rather than validation against phenotypes
- Input metabolomics data has not yet passed QCprep/QCprep_surrogates and score projection, so no surrogate scores exist to validate

## Inputs

- Projected surrogate scores table produced by the upstream projection step (e.g. from calculate_surrogate_scores)
- Phenotypic dataset with matching sample IDs (e.g. MiMIR synthetic_phenotypic_dataset)
- MiMIR packaged acc_LOBOV reference dataset of BBMRI-nl LOBOV AUCs

## Outputs

- Per-surrogate ROC curve and AUC values (numeric, in [0, 1])
- Per-surrogate ROC curve panels (roc_surro_subplots figures)
- surrogate_auc_comparison.csv containing local AUC vs LOBOV reference AUC per surrogate

## How to apply

Load the projected surrogate scores together with the phenotypic dataset (e.g. MiMIR's synthetic_phenotypic_dataset) and binarize all phenotypic variables with MiMIR::binarize_all_pheno. For each surrogate whose binary phenotype is available, compute the ROC curve and its AUC with MiMIR::roc_surro, which is built on the pROC package, using surrogate values versus the binarized phenotype. Render per-surrogate ROC curve panels with MiMIR::roc_surro_subplots for visual inspection. Then load the packaged acc_LOBOV reference dataset and build the side-by-side comparison of locally computed AUCs versus the BBMRI-nl Leave-One-Biobank-Out Validation AUCs using MiMIR::LOBOV_accuracies, writing the result to surrogate_auc_comparison.csv. As a reference point, the diabetes surrogate reaches AUC 0.85 in LLS_PAROFFs versus a mean LOBOV AUC of 0.8, so local values should be in that ballpark for the model to be trusted.

## Related tools

- **MiMIR** (Provides roc_surro, roc_surro_subplots, binarize_all_pheno, LOBOV_accuracies and the acc_LOBOV reference dataset used to validate surrogate scores) — https://github.com/DanieleBizzarri/MiMIR
- **pROC** (R package underlying the ROC-curve and AUC computation in roc_surro)
- **R** (Runtime environment (R version 3.6+) in which the MiMIR validation functions are executed)

## Examples

```
library(MiMIR); pheno_bin <- binarize_all_pheno(pheno); roc_surro(scores, pheno_bin); LOBOV_accuracies(acc_LOBOV, auc_table) # write surrogate_auc_comparison.csv
```

## Evaluation signals

- Every AUC reported in surrogate_auc_comparison.csv is a numeric value within [0, 1]
- Each surrogate row appears in the comparison table with its LOBOV reference AUC present for side-by-side comparison
- Local AUCs are biologically plausible relative to LOBOV references (e.g. diabetes surrogate AUC 0.85 in LLS_PAROFFs vs mean LOBOV AUC 0.8)
- ROC curve panels render one inspectable curve per surrogate with a binarized phenotype available
- Binarization of phenotypes succeeds for all variables via binarize_all_pheno before ROC computation

## Limitations

- Validation is only possible for surrogates whose corresponding binary phenotype is present in the uploaded phenotypic dataset
- Comparison is against LOBOV AUCs derived within BBMRI-nl; locally computed AUCs in other cohorts may legitimately differ from those references
- Phenotypic variables must be binarized with binarize_all_pheno before ROC computation; unbinarized or misnamed columns will break the comparison

## Evidence

- [other] MiMIR validates surrogate scores by computing AUCs when the corresponding phenotypic variable is uploaded, and these AUCs are compared to LOBOV AUCs: [evidence span withheld — anchor status: not_found]
- [methods] calculate_surrogate_scores signature: "`calculate_surrogate_scores(met, pheno, PARAM_surrogates, bin_names, Nmax_miss, Nmax_zero, post, roc, quiet)`"
- [methods] binarize_all_pheno: "`binarize_all_pheno(data)`"
- [readme] pROC dependency in README: "if (!require("pROC")) install.packages("pROC")"
- [readme] surrogate scores citation from README: "'1H-NMR metabolomics-based surrogates to impute common clinical risk factors and endpoints', EBioMedicine, vol. 75, p. 103764"
