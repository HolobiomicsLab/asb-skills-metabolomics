---
name: repeated-cross-validation-accuracy-estimation
description: Use when comparing normalization methods on metabolomics abundance data
  by supervised classification performance — e.g. predicting hypertension status from
  BioHEART-CT plasma metabolite abundances — where a repeated, averaged CV accuracy
  estimate is needed per method.
when_to_use_negative:
- Data have not yet been normalized — this skill evaluates normalized matrices, so
  normalization (e.g. hRUV variants) must be applied first
- The outcome variable is not a binary clinical label such as hypertension status
  — other endpoints need their own classifier and validation design
- The goal is assessing replicate variability or signal-drift removal itself rather
  than retention of biological signal via classification
edam_topics: []
tools:
- name: R
  role: Used to run the 30 repeats of 10-fold cross-validation and perform all randomization
    via the sample function
- name: hRUV
  role: Produces the normalized abundance matrices (e.g. loessAllShort_batch_Hc variant)
    whose classification accuracy is being benchmarked
  repo: github.com/SydneyBioX/hRUV
- name: SVM classifier
  role: Supervised model trained on each normalized matrix to predict hypertension
    status during cross-validation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:28b5cdf77ad9b18cbca556b9d113a5b47c4900e555452c617d060527b9ab3a63
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SVM classifier
  - hRUV
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# repeated-cross-validation-accuracy-estimation

## Summary

Estimates supervised classification accuracy of normalized metabolomics abundance matrices by training an SVM to predict hypertension status over 30 repeats of 10-fold cross-validation, used to benchmark normalization methods (e.g. hRUV variants vs ratio, SVR, SERRF, RLSC) on retention of biological signal.

## When to use

Use when comparing normalization methods on metabolomics abundance data by supervised classification performance — e.g. predicting hypertension status from BioHEART-CT plasma metabolite abundances — where a repeated, averaged CV accuracy estimate is needed per method.

## When NOT to use

- Data have not yet been normalized — this skill evaluates normalized matrices, so normalization (e.g. hRUV variants) must be applied first
- The outcome variable is not a binary clinical label such as hypertension status — other endpoints need their own classifier and validation design
- The goal is assessing replicate variability or signal-drift removal itself rather than retention of biological signal via classification

## Inputs

- Normalized metabolomics abundance matrix (one per normalization method, e.g. loessAllShort_batch_Hc, ratio, SVR, SERRF, RLSC)
- Sample-level hypertension status labels from the BioHEART-CT metabolomics dataset
- Raw abundance matrix and hypertension labels from SydneyBioX/BioHEART_metabolomics

## Outputs

- cv_accuracy_results.csv with mean cross-validated classification accuracy per normalization method
- Ranked list of normalization methods by average accuracy

## How to apply

Take each normalized abundance matrix produced by a candidate method (hRUV variants such as loessAllShort_batch_Hc — loess drift correction plus intra-batch RUV with short replicates followed by hierarchical inter-batch RUV — plus ratio, SVR, SERRF, RLSC), after filtering metabolites not present in at least 50% of samples. Train an SVM classifier to predict hypertension status from each matrix, using the same fold partitioning across methods within each repeat so the comparison is paired. Run 30 repeats of 10-fold cross-validation per method, performing all randomization (fold assignment, replicate resampling) with the sample function in R, and compute the mean classification accuracy per method. Rank methods by average cross-validated accuracy to identify which normalization best retains the biological signal, and write the per-method accuracies to cv_accuracy_results.csv.

## Related tools

- **R** (Used to run the 30 repeats of 10-fold cross-validation and perform all randomization via the sample function)
- **hRUV** (Produces the normalized abundance matrices (e.g. loessAllShort_batch_Hc variant) whose classification accuracy is being benchmarked) — github.com/SydneyBioX/hRUV
- **SVM classifier** (Supervised model trained on each normalized matrix to predict hypertension status during cross-validation)

## Examples

```
Rscript -e 'acc <- sapply(1:30, function(r) mean(sapply(split(sample(nrow(X)), rep(1:10, length.out=nrow(X))), function(f) mean(pred == truth[f])))); write.csv(data.frame(method, mean_acc=acc), "cv_accuracy_results.csv")'
```

## Evaluation signals

- Each method yields exactly 30×10 = 300 fold-level accuracy values, and the mean accuracy is reported per normalization method
- Within each repeat, the same fold assignment (drawn with R's sample function) is applied to every normalization method so the accuracy comparison is paired
- loessAllShort_batch_Hc is expected to achieve the top average accuracy for hypertension prediction among the compared methods
- Results table cv_accuracy_results.csv exists and contains one averaged accuracy entry per evaluated normalization method
- Metabolites absent from ≥50% of samples are excluded from every matrix before classification

## Limitations

- hRUV expects data without missing values, which is often not possible in large-scale metabolomics, requiring an imputation workaround before normalization/CV
- Selection of appropriate negative-control metabolites for batch correction is challenging because metabolite signals depend on their individual chemical properties
- Random fold splits are stochastic; conclusions should rely on the mean over 30 repeats rather than any single cross-validation run

## Evidence

- [discussion] All randomization was performed using the sample function in R.: "All randomization was performed using the sample function in R."
- [discussion] metabolites not present in at least 50% of samples were filtered out: "metabolites that were not present in at least 50% of the samples were filtered out"
- [abstract] hRUV retains biological signal better than existing normalization methods: "We demonstrate significant improvement of hRUV over existing methods in maintaining biological signals whilst removing unwanted variation for large scale metabolomics studies"
- [readme] hRUV package tested on R 4.0.3 with CRAN/Bioconductor dependencies: [evidence span withheld — anchor status: external_doc]
