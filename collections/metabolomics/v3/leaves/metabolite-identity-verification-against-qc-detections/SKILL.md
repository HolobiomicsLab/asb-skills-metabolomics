---
name: metabolite-identity-verification-against-qc-detections
description: Use when applying QC-based drift correction (lowess using pooled surrogate
  QCs) and you must pre-compile the list of features undetected in QCs — e.g. to confirm
  xanthosine 5'-monophosphate, detected only in study samples, is excluded a priori.
when_to_use_negative:
- The target feature is detected in the surrogate QC injections — it can be drift-corrected
  by the QC-based lowess model and does not belong on the excluded list
- You plan to use dbnorm's statistical correction models (ber, ber-bagging, parametric
  or nonparametric ComBat), which retained xanthosine 5'-monophosphate in SKIPOGH
  and do not impose this QC-detection filter
- The dataset contains no surrogate QC samples, so no QC-based detection profile exists
  to filter against
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: LOWESS-Normalization
  role: Reference implementation of the QC sample-based lowess drift-correction model
    whose feature-coverage limitation this filtering step anticipates
- name: dbnorm
  role: R package providing lowess and statistical (ber, ComBat) batch-correction
    models used to contrast QC-based filtering with statistical-model retention; also
    supplies emvd/emvf missing-value handling
  repo: https://github.com/NBDZ/dbnorm
- name: R
  role: Environment in which the feature matrix is loaded, log2-transformed, and the
    all-zero/NA-in-QC feature list is compiled
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-84824-3
    title: DBnorm as an R package for the comparison and selection of appropriate
      statistical methods for batch effect correction in metabolomic studies
  source_content_hash: sha256:6eaa9a7b1dd5726f64ad6d72126e4ef583f3e466616429dcb9c579955e1f73c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NBDZ/dbnorm
  tools:
  - LOWESS-Normalization
  - R
  - dbnorm
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# metabolite-identity-verification-against-qc-detections

## Summary

Identifies metabolic features that a QC sample-based lowess drift-correction model must discard a priori because they have zero/NA intensity in all surrogate pooled-QC injections (e.g. xanthosine 5'-monophosphate, detected only in study samples), separating them from features that can be batch-corrected.

## When to use

Use when applying QC-based drift correction (lowess using pooled surrogate QCs) and you must pre-compile the list of features undetected in QCs — e.g. to confirm xanthosine 5'-monophosphate, detected only in study samples, is excluded a priori.

## When NOT to use

- The target feature is detected in the surrogate QC injections — it can be drift-corrected by the QC-based lowess model and does not belong on the excluded list
- You plan to use dbnorm's statistical correction models (ber, ber-bagging, parametric or nonparametric ComBat), which retained xanthosine 5'-monophosphate in SKIPOGH and do not impose this QC-detection filter
- The dataset contains no surrogate QC samples, so no QC-based detection profile exists to filter against

## Inputs

- log2-transformed feature-intensity matrix in .csv format (experiments in rows, features in columns, batch level in the first column per dbnorm convention)
- Sample-role annotation distinguishing study samples from surrogate pooled-tissue QC injections and their injection order across batch blocks
- Polarity/batch metadata for LC–MS injections (to identify which QCs analyzed a feature's corresponding polarity)

## Outputs

- qc_lowess_corrected_data.csv — drift-corrected feature table restricted to QC-injected features
- qc_excluded_features.csv — a-priori discarded-feature list (zero/NA in all QC injections)

## How to apply

Load the untargeted metabolomics feature matrix (e.g. 232 adipose tissue samples plus 32 pooled-tissue surrogate QCs injected in random order across batch blocks) and log2-transform each feature, as dbnorm requires inputs 'normalized and scaled on the log2' with experiments in rows and features in columns of a .csv. Flag every feature with zero or NA intensity in all surrogate pooled-QC injections and compile these into an a-priori excluded-feature list, since a QC-based lowess model cannot estimate drift for features it never observes. Confirm the presence of metabolites detected only in study samples — in the SKIPOGH demonstration lowess 'filtered out' xanthosine 5'-monophosphate while the dbnorm statistical models (ber, parametric/nonparametric ComBat, ber-bagging) retained it. Then run the QC-based lowess correction (LOWESS-Normalization reference implementation) only on the QC-injected feature subset. Save the corrected table and the discarded-feature list as separate outputs for downstream verification.

## Related tools

- **LOWESS-Normalization** (Reference implementation of the QC sample-based lowess drift-correction model whose feature-coverage limitation this filtering step anticipates)
- **dbnorm** (R package providing lowess and statistical (ber, ComBat) batch-correction models used to contrast QC-based filtering with statistical-model retention; also supplies emvd/emvf missing-value handling) — https://github.com/NBDZ/dbnorm
- **R** (Environment in which the feature matrix is loaded, log2-transformed, and the all-zero/NA-in-QC feature list is compiled)

## Examples

```
data<-read.csv("mydata.csv",sep=",",header=T,row.names=1); excl<-colnames(data)[apply(data[qc_idx,],2,function(x) all(is.na(x)|x==0))]
```

## Evaluation signals

- xanthosine 5'-monophosphate appears in qc_excluded_features.csv, matching the article's demonstration that lowess filtered it out
- Every entry in the excluded-feature list has zero or NA intensity in all surrogate pooled-QC injections (verifiable by row-subsetting the QC samples)
- qc_lowess_corrected_data.csv contains no feature from the excluded list — the corrected table and discarded list are disjoint and together cover all input features
- Cross-check against a dbnorm statistical model (e.g. ber or parametric ComBat): the filtered feature should be retained there, reproducing the SKIPOGH contrast between lowess and statistical models

## Limitations

- QC sample-based correction structurally cannot correct metabolites below the limit of detection in pooled QCs; such features are irrecoverably dropped, unlike with dbnorm statistical models
- Lowess-corrected data can retain substantial residual batch association (maximum 78% residual variability in SKIPOGH, vs <1% for ber and parametric ComBat)
- dbnorm's Visodbnorm/dbnormSCORE functions are suggested for fewer than 2000 features for computational speed
- Input data must already be normalized and scaled on log2, with batch levels in the first column; untransformed or misformatted data violates the package contract

## Evidence

- [results] xanthosine 5′-monophosphate, which was filtered out with lowess: "xanthosine 5′-monophosphate, which was filtered out with lowess"
- [readme] Data to be uploaded must be normalized and scaled on the log2 to account for the high abundance features (variables) by which technical heterogeneity might be overlooked. The input data must be in .csv format: "Data to be uploaded must be normalized and scaled on the log2 to account for the high abundance features (variables) by which technical heterogeneity might be overlooked."
- [discussion] Periodic injection of QC samples is one of the most commonly used methodologies in the metabolomics: "Periodic injection of QC samples is one of the most commonly used methodologies in the metabolomics"
- [abstract] some residual association was observed in lowess- and nonparametric Com Bat-corrected data: [evidence span withheld — anchor status: not_found]
