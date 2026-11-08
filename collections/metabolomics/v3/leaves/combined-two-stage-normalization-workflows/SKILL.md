---
name: combined-two-stage-normalization-workflows
description: Use when a metabolomics featuredata matrix shows both batch/QC-RLSC-correctable
  drift and sample-level intensity differences that no single normalization (QC metabolite,
  QC sample, or scaling) fixes alone.
when_to_use_negative:
- A single normalization stage suffices — e.g. data needing only per-sample median/mean/sum/ref
  scaling, which NormScaling handles directly
- The input is raw peak intensities not yet in featuredata format (unique sample and
  metabolite row/column names required) or still contains missing values needing LogTransform/MissingValues
  preprocessing first
- The first-stage method requires QC-sample or internal-standard annotations (metabolitedata
  flags, QC sampledata) that are not available for the dataset
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0266
tools:
- name: NormalizeMets
  role: Provides the NormCombined function that sequences two normalization methods
    (e.g. rlsc then median) on a featuredata matrix
  repo: metabolomicstats/NormalizeMets
- name: R
  role: Execution environment in which NormalizeMets (version requiring R >= 3.4.3)
    is installed and NormCombined is called
  repo: https://cran.r-project.org/
- name: statTarget
  role: Reference implementation of the QC-RLSC method that can serve as the first
    ('rlsc') stage of the combined workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-018-1347-7
    title: 'NormalizeMets: assessing, selecting and implementing statistical methods
      for normalizing metabolomics data'
  source_content_hash: sha256:bbf33b239602bd2a6c0beef701ef697c708e5a6678a1f8f0157cc25b6dd4f569
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/metabolomicstats/NormalizeMets
  tools:
  - NormalizeMets
  - R
  - statTarget
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0266
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Combined two-stage normalization workflows

## Summary

Chains two normalization methods back-to-back (e.g. QC-RLSC signal correction followed by median scaling) using the NormalizeMets NormCombined function, to remove both analytical drift and per-sample intensity variation in metabolomics feature matrices.

## When to use

Use when a metabolomics featuredata matrix shows both batch/QC-RLSC-correctable drift and sample-level intensity differences that no single normalization (QC metabolite, QC sample, or scaling) fixes alone.

## When NOT to use

- A single normalization stage suffices — e.g. data needing only per-sample median/mean/sum/ref scaling, which NormScaling handles directly
- The input is raw peak intensities not yet in featuredata format (unique sample and metabolite row/column names required) or still contains missing values needing LogTransform/MissingValues preprocessing first
- The first-stage method requires QC-sample or internal-standard annotations (metabolitedata flags, QC sampledata) that are not available for the dataset

## Inputs

- featuredata matrix (samples as row names, metabolites as column names), typically log-transformed and KNN-imputed
- sampledata dataframe (e.g. run order, factors of interest) needed by the first-stage method such as QC-RLSC
- vector of two normalization method names, e.g. c('rlsc','median')

## Outputs

- Combined normalized featuredata matrix (two stages applied sequentially)
- CSV file of the normalized matrix when savefinaloutput=TRUE and finaloutputname is set

## How to apply

Prepare the matrix in featuredata format (log-transformed via LogTransform with zerotona=TRUE, missing values imputed via MissingValues with method='knn', k=10, feature/sample cutoffs 0.8). Call NormCombined passing an ordered vector of two methods, e.g. methods=c('rlsc','median'): the first stage applies QC sample-based robust LOESS signal correction (span=0, deg=2, lg=TRUE) to remove injection-order drift, and the second applies sample-wise scaling (median/mean/sum/ref) to equalize sample intensities. Set savefinaloutput=TRUE with a finaloutputname to write the combined matrix out. Rationale: the article notes researchers routinely use 'one method followed by another', and the demonstrated combination is rlsc followed by median. Judge success by comparing residual RLA plots and p-value distributions from LinearModelFit against the single-method (NormQcmets, NormQcsamples, NormScaling) outputs on the same data.

## Related tools

- **NormalizeMets** (Provides the NormCombined function that sequences two normalization methods (e.g. rlsc then median) on a featuredata matrix) — metabolomicstats/NormalizeMets
- **R** (Execution environment in which NormalizeMets (version requiring R >= 3.4.3) is installed and NormCombined is called) — https://cran.r-project.org/
- **statTarget** (Reference implementation of the QC-RLSC method that can serve as the first ('rlsc') stage of the combined workflow)

## Examples

```
NormCombined(featuredata=imp$featuredata, methods=c("rlsc","median"), savefinaloutput=TRUE, finaloutputname="combined_norm")
```

## Evaluation signals

- Output matrix has the same dimensions, row names (samples) and column names (metabolites) as the input featuredata
- Residual RLA plots from LinearModelFit show reduced within-group variation relative to both single-method (NormQcsamples-only, NormScaling-only) outputs
- p-value distributions from linear models fitted to the combined-normalized data are compared against single-method outputs for the same factors (e.g. gender, Age, bmi)
- savefinaloutput=TRUE produces a CSV whose values match the in-memory combined matrix after the second (median) stage

## Limitations

- Only normalization method combinations supported by the package's underlying functions can be sequenced; the demonstrated pair is c('rlsc','median')
- Unwanted variation arises from batch effects, matrix effects, and confounding biological variation — combined correction cannot recover from poor input quality or wrongly specified factors
- Method choice should be re-evaluated per dataset and research question; the package's own guidance is comparative evaluation rather than assuming any fixed combination is optimal

## Evidence

- [methods] Combined normalization rationale: "researchers use a combination of the above normalizations (i.e., one method followed by another). This can be achieved using the _NormCombined_ function."
- [methods] NormCombined signature: "NormCombined<-function(featuredata, methods = c("rlsc", "median"),
  savefinaloutput = FALSE, finaloutputname = NULL, ...)"
- [methods] Scaling methods available for the second stage: [evidence span withheld — anchor status: not_found]
- [readme] Why combined correction is needed: [evidence span withheld — anchor status: external_doc]
