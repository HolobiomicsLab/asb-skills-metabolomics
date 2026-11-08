---
name: svm-classification-hypertension-status
description: Use when you need to quantify how much biological signal a normalisation
  method preserves — e.g. comparing hRUV variants, ratio, SVR, SERRF, or RLSC on their
  cross-validated accuracy for a binary clinical outcome such as hypertension status.
when_to_use_negative:
- When the evaluation goal is unsupervised normalisation quality (e.g. variability
  among replicates or signal-drift removal) rather than retention of a supervised
  biological signal — the paper assesses normalisation on multiple criteria, and this
  skill covers only the supervised-accuracy criterion
- When hypertension (or other outcome) labels are unavailable for the samples, since
  the SVM cross-validation requires a supervised response variable
- When the study design lacks embedded biological replicates across batches, since
  hierarchical hRUV depends on intra- and inter-batch replicates for unwanted-variation
  estimation
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: hRUV
  role: Provides the hierarchical normalisation variants (e.g. loessAllShort_batch_Hc)
    whose cross-validated SVM accuracy is benchmarked; installed as an R package
  repo: github.com/SydneyBioX/hRUV
- name: R
  role: Analysis environment; the SVM classifier, 10-fold cross-validation splits,
    and randomisation (sample function) are all performed in R
- name: SydneyBioX/BioHEART_metabolomics
  role: Source of the BioHEART-CT metabolomics abundance matrix and hypertension status
    labels used as benchmark input
  repo: SydneyBioX/BioHEART_metabolomics
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:5d33377bb9330dc97a88e759765473425c263bfb949c4d16b232d72ab2d65a2f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SydneyBioX/BioHEART_metabolomics
  - hRUV
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# svm-classification-hypertension-status

## Summary

Benchmarks metabolomics normalisation methods by their ability to retain biological signal, using an SVM classifier to predict hypertension status from normalised metabolite abundances under 30 repeats of 10-fold cross-validation. This reproduces the accuracy comparison (Fig. 3c) of the hRUV paper on the BioHEART-CT cohort.

## When to use

Use when you need to quantify how much biological signal a normalisation method preserves — e.g. comparing hRUV variants, ratio, SVR, SERRF, or RLSC on their cross-validated accuracy for a binary clinical outcome such as hypertension status.

## When NOT to use

- When the evaluation goal is unsupervised normalisation quality (e.g. variability among replicates or signal-drift removal) rather than retention of a supervised biological signal — the paper assesses normalisation on multiple criteria, and this skill covers only the supervised-accuracy criterion
- When hypertension (or other outcome) labels are unavailable for the samples, since the SVM cross-validation requires a supervised response variable
- When the study design lacks embedded biological replicates across batches, since hierarchical hRUV depends on intra- and inter-batch replicates for unwanted-variation estimation

## Inputs

- Raw or preprocessed metabolomics abundance matrix from the BioHEART-CT discovery cohort (targeted LC-MS/MS plasma metabolomics, Agilent 1260 Infinity HPLC + Qtrap5500)
- Hypertension status labels for BioHEART-CT samples
- Normalisation method implementations to benchmark: hRUV variants (e.g. loessAllShort_batch_Hc), ratio, SVR, SERRF, RLSC
- Replicate design (plateDesign function in the hRUV package) defining intra-batch short replicates and inter-batch batch replicates

## Outputs

- cv_accuracy_results.csv containing per-method cross-validated classification accuracies
- Ranked comparison of normalisation methods by mean accuracy over 30 repeats × 10 folds (Fig. 3c analogue)
- Normalised abundance matrix per method as intermediate output

## How to apply

Load the BioHEART-CT metabolomics abundance matrix and hypertension status labels from the SydneyBioX/BioHEART_metabolomics repository. Apply each candidate normalisation method — including the hierarchical hRUV variant 'loessAllShort_batch_Hc' (loess drift correction plus intra-batch RUV with short replicates, followed by hierarchical inter-batch RUV) and competitors such as ratio, SVR, SERRF, and RLSC — to produce normalised abundance matrices; metabolites not present in at least 50% of samples should be filtered beforehand. For each normalised matrix, train an SVM classifier to predict hypertension status, running 30 repeats of 10-fold cross-validation with randomisation performed using the sample function in R. Compute the mean cross-validated accuracy per method, rank methods by average accuracy, and save the results to cv_accuracy_results.csv. The expected outcome is that loessAllShort_batch_Hc achieves the best prediction accuracy, consistent with the paper's claim that hRUV better retains biological variation while removing unwanted variation.

## Related tools

- **hRUV** (Provides the hierarchical normalisation variants (e.g. loessAllShort_batch_Hc) whose cross-validated SVM accuracy is benchmarked; installed as an R package) — github.com/SydneyBioX/hRUV
- **R** (Analysis environment; the SVM classifier, 10-fold cross-validation splits, and randomisation (sample function) are all performed in R)
- **SydneyBioX/BioHEART_metabolomics** (Source of the BioHEART-CT metabolomics abundance matrix and hypertension status labels used as benchmark input) — SydneyBioX/BioHEART_metabolomics

## Examples

```
devtools::install_github("SydneyBioX/hRUV", build_vignettes=TRUE, dependencies = TRUE)
```

## Evaluation signals

- The hRUV variant 'loessAllShort_batch_Hc' ranks highest in mean cross-validated SVM accuracy among the compared normalisation methods
- Accuracy for each method is the mean over exactly 30 repeats of 10-fold cross-validation (300 held-out folds per method), with randomisation via R's sample function
- cv_accuracy_results.csv is produced with one accuracy row per evaluated normalisation method
- Before normalisation, metabolites absent from more than 50% of samples have been filtered out and outlying measurements among the three consecutive starting-QC samples removed
- Reordering of methods by accuracy is stable across CV repeats (low run-to-run variance in mean accuracy)

## Limitations

- hRUV expects data without missing values, which is often not possible in large-scale metabolomics data and requires an imputation workaround before classification
- The paper documents residual slight signal shifts between batches 6–7 and 13–14 due to unintended clustering of batch replicates, which may affect accuracy estimates for metabolites in those batches
- Selection of negative-control metabolites is challenging because metabolite signals depend on individual chemical properties; however, no significant differences were found between data-driven negative controls and all metabolites as negative controls
- Names/abbreviations of all 13 compared normalisation methods appear only in Tables 1–2 of the paper, which are not included in the available text

## Evidence

- [discussion] hRUV performs consistently better than them in retaining biological variation whilst at the same time removing unwanted variation within and across batches.: "hRUV performs consistently better than them in retaining biological variation whilst at the same time removing unwanted variation within and across batches."
- [discussion] All randomization was performed using the sample function in R.: "All randomization was performed using the sample function in R."
- [other] Run 30 repeats of 10-fold cross-validation per method (R sample function for randomisation) and compute the mean classification accuracy.: [evidence span withheld — anchor status: not_found]
- [discussion] metabolites that were not present in at least 50% of the samples were filtered out: "metabolites that were not present in at least 50% of the samples were filtered out"
- [discussion] While the proposed hRUV algorithm expects data without missing values, this is often not possible in large-scale metabolomics data: "While the proposed hRUV algorithm expects data without missing values, this is often not possible in large-scale metabolomics data"
