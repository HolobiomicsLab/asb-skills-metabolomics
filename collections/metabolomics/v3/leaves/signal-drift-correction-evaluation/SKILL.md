---
name: signal-drift-correction-evaluation
description: Decision support for choosing and validating a within-batch signal-drift
  correction smoother (rlm vs loess) when metabolomics abundance data show run-order-dependent
  drift and downstream accuracy/replicate consistency must be judged.
when_to_use_negative:
- The input already has run-order effects removed or was produced by a drift-correcting
  normaliser such as hRUV's intraNorm step — re-fitting smoothers would double-correct.
- The data are pooled-QC-only (no biological replicates or phenotype labels embedded),
  since signal-retention evaluation via supervised classification cannot be performed.
- The dataset is single-batch with no run-order structure, where within-batch drift
  correction has no target to remove.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: hRUV
  role: Provides the intra-batch signal-drift correction (intraNorm with rlm or loess
    smoothers) and the hierarchical inter-batch RUV whose variants are being evaluated.
  repo: https://github.com/SydneyBioX/hRUV
- name: R
  role: Implementation language; randomisation of the 30 × 10-fold cross-validation
    for accuracy evaluation is performed with the sample function in R.
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:4ea88e213a61c9bd07fe31412b9ac18baa639414921d1fe3cdb152cac1fa1825
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/hRUV
  tools:
  - R
  - hRUV
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# signal-drift-correction-evaluation

## Summary

Evaluates how well a metabolomics normalisation workflow corrects within-batch signal drift by comparing alternative smoothers (robust linear rlm vs non-linear loess) fitted per metabolite against run order, and by assessing drift correction downstream via replicate variability and supervised biological-signal retention (e.g. hypertension prediction accuracy).

## When to use

Decision support for choosing and validating a within-batch signal-drift correction smoother (rlm vs loess) when metabolomics abundance data show run-order-dependent drift and downstream accuracy/replicate consistency must be judged.

## When NOT to use

- The input already has run-order effects removed or was produced by a drift-correcting normaliser such as hRUV's intraNorm step — re-fitting smoothers would double-correct.
- The data are pooled-QC-only (no biological replicates or phenotype labels embedded), since signal-retention evaluation via supervised classification cannot be performed.
- The dataset is single-batch with no run-order structure, where within-batch drift correction has no target to remove.

## Inputs

- Metabolomics abundance matrix (metabolite × sample) with run-order and batch annotations per well (BioHEART-CT plasma samples)
- Hypertension status labels for supervised signal-retention evaluation
- Replicate annotations (pooled QC, short within-row replicates, inter-batch replicates) for drift/replicate-variability assessment

## Outputs

- Drift-adjusted log2 metabolite-by-sample matrix (batch and run-order annotations preserved)
- Cross-validated classification accuracy per normalisation/drift-correction variant (e.g. cv_accuracy_results.csv) and ranking of methods

## How to apply

Log2-transform the metabolite abundance matrix, then for each batch fit a robust smoother of log2 signal against run order per metabolite and subtract the fitted values to remove drift. The article evaluates two smoother families: robust linear models (rlm) and non-linear loess (local regression), embedded in the hRUV intraNorm step; the variant 'loessAllShort_batch_Hc' (loess drift correction plus intra-batch RUV with short replicates followed by hierarchical inter-batch RUV) achieved the best downstream performance. Judge correction quality on two axes: low variability among embedded sample replicates (pooled QC, short, and batch replicates) and retention of biological signal, operationalised as mean supervised classification accuracy for hypertension status from 30 repeats of 10-fold cross-validation using an SVM on the drift-adjusted matrix. Rank candidate smoothers/variants by that cross-validated accuracy and record results (e.g. cv_accuracy_results.csv) so drift correction is validated rather than assumed.

## Related tools

- **hRUV** (Provides the intra-batch signal-drift correction (intraNorm with rlm or loess smoothers) and the hierarchical inter-batch RUV whose variants are being evaluated.) — https://github.com/SydneyBioX/hRUV
- **R** (Implementation language; randomisation of the 30 × 10-fold cross-validation for accuracy evaluation is performed with the sample function in R.)

## Examples

```
R: devtools::install_github("SydneyBioX/hRUV"); library(hRUV); res <- intraNorm(log2(abundance), run_order, batch, smoother = "loess")  # then evaluate via 30×10-fold CV
```

## Evaluation signals

- Fitted smoother per metabolite should track the run-order trend visible in raw log2 abundance; subtracting it should remove systematic run-order dependence while leaving non-drift variation intact.
- Drift-adjusted matrix should show low variability among embedded replicates (pooled QC, short, and batch replicates) compared with the raw matrix.
- Cross-validated hypertension-prediction accuracy (30 repeats × 10-fold, SVM) should match or exceed the article's reported ranking, with the loess-based 'loessAllShort_batch_Hc' variant performing best.
- Accuracy results should be reproducible across the 30 randomised CV repeats (seeded via the R sample function) and comparable in scale across all normalisation methods tested.
- No missing values should remain in the drift-adjusted matrix — the hRUV algorithm expects complete data, so imputation (e.g. DMwR2 knn) is required beforehand.

## Limitations

- hRUV expects data without missing values, which is often not possible in large-scale metabolomics, requiring an imputation workaround before drift correction.
- Residual slight shift in signal between batches 6–7 and 13–14 remained for selected metabolites when replicate sampling clustered on only one half of the plate, limiting capture of plate-wide positional variability.
- No significant difference was found between data-driven negative-control metabolites and using all metabolites; the authors note inter-batch sample replicates were more effective for batch correction, so drift-correction evaluation should not over-attribute gains to negative-control choice.
- Rankings depend on the chosen evaluation axis; the article judges drift correction by both replicate variability and supervised biological-signal retention, and conclusions may not transfer to other phenotypes.

## Evidence

- [methods] Within each batch, signal drift is corrected using a robust smoother that captures the trends visible by run order: [evidence span withheld — anchor status: not_found]
- [results] Among the evaluated normalization methods, the hierarchical hRUV variant 'loessAllShort_batch_Hc': [evidence span withheld — anchor status: not_found]
- [discussion] While the proposed hRUV algorithm expects data without missing values, this is often not possible in large-scale metabolomics data: "While the proposed hRUV algorithm expects data without missing values, this is often not possible in large-scale metabolomics data"
- [discussion] hRUV performs consistently better than them in retaining biological variation whilst at the same time removing unwanted variation within and across batches.: "hRUV performs consistently better than them in retaining biological variation whilst at the same time removing unwanted variation within and across batches."
- [readme] Our novel tool is a novel hierarchical approach to removing unwanted variation by harnessing information from sample replicates embedded in the seequence: "Our novel tool is a novel hierarchical approach to removing unwanted variation by harnessing information from sample replicates embedded in the seequence of experimental runs/batches and applying"
