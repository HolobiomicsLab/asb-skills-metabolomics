---
name: qc-based-lowess-drift-correction
description: Use when a metabolomics feature matrix includes pooled QC samples injected
  across batches and you must correct across-batch signal drift while identifying
  features (e.g. xanthosine 5'-monophosphate) absent from QCs.
when_to_use_negative:
- A feature is undetected (zero/NA) in all pooled-QC injections — it cannot be drift-corrected
  by a QC-based model and must be excluded a priori; use statistical models such as
  ber or ComBat implemented in dbnorm instead
- The dataset lacks periodically injected QC samples — QC-based lowess requires repeated
  QC measurements to model the drift
- The goal is comparing multiple correction models rather than applying the QC-based
  lowess method — dbnorm's dbnormSCORE/Visodbnorm functions are designed for model
  comparison and selection
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: LOWESS-Normalization
  role: Reference implementation of the QC sample-based lowess drift-correction model
    applied to the QC-injected feature subset
- name: dbnorm
  role: R package providing statistical batch-effect correction models (ber, parametric/nonparametric
    ComBat) and lowess comparison, used to demonstrate the coverage limitation of
    QC-based correction
  repo: https://github.com/NBDZ/dbnorm
- name: R
  role: Environment in which the log2 transformation, QC-detection screening, and
    lowess correction are executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-84824-3
    title: DBnorm as an R package for the comparison and selection of appropriate
      statistical methods for batch effect correction in metabolomic studies
  source_content_hash: sha256:0b347e359c98119c093903eae32058701da17cf032d3e57e96941e282323bfac
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
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# qc-based-lowess-drift-correction

## Summary

Applies QC sample-based lowess drift correction to log2-transformed untargeted metabolomics feature matrices with pooled-tissue surrogate QCs, and compiles the a-priori excluded-feature list of metabolites that are undetected (zero/NA) in all QC injections and therefore cannot be drift-corrected by a QC-based model.

## When to use

Use when a metabolomics feature matrix includes pooled QC samples injected across batches and you must correct across-batch signal drift while identifying features (e.g. xanthosine 5'-monophosphate) absent from QCs.

## When NOT to use

- A feature is undetected (zero/NA) in all pooled-QC injections — it cannot be drift-corrected by a QC-based model and must be excluded a priori; use statistical models such as ber or ComBat implemented in dbnorm instead
- The dataset lacks periodically injected QC samples — QC-based lowess requires repeated QC measurements to model the drift
- The goal is comparing multiple correction models rather than applying the QC-based lowess method — dbnorm's dbnormSCORE/Visodbnorm functions are designed for model comparison and selection

## Inputs

- Untargeted metabolomics feature matrix (.csv) with study samples plus surrogate pooled-tissue QC injections, batch block annotations, and raw feature intensities
- Injection-order/run-order metadata identifying which injections are pooled-tissue surrogate QCs and which polarity mode was used per feature

## Outputs

- qc_lowess_corrected_data.csv — the drift-corrected feature table for QC-detected features
- qc_excluded_features.csv — the a-priori excluded-feature list (zero/NA in all surrogate QC injections, e.g. xanthosine 5'-monophosphate)

## How to apply

Load the feature matrix (e.g. 232 adipose tissue samples plus 32 pooled-tissue surrogate QCs injected in random order across batch blocks) and log2-transform each metabolite feature, as dbnorm expects normalized, log2-scaled data in .csv format with samples in rows, features in columns and the batch level in the first column. Identify features with zero or NA intensity in all surrogate pooled-QC injections and compile them into an a-priori excluded-feature list, since a QC-based drift model cannot estimate correction curves without QC measurements. Confirm that metabolites detected only in study samples (such as xanthosine 5'-monophosphate, missing in QC injections analyzed in the corresponding polarity) appear in that excluded list — the article reports this same coverage limitation for lowess on the SKIPOGH targeted dataset, where dbnorm's statistical models retained the metabolite. On the remaining QC-injected feature subset, run the LOWESS-Normalization QC-based lowess model to fit and remove the overtime signal drift across batches. Save the QC-corrected feature table as qc_lowess_corrected_data.csv and the discarded-feature list as qc_excluded_features.csv.

## Related tools

- **LOWESS-Normalization** (Reference implementation of the QC sample-based lowess drift-correction model applied to the QC-injected feature subset)
- **dbnorm** (R package providing statistical batch-effect correction models (ber, parametric/nonparametric ComBat) and lowess comparison, used to demonstrate the coverage limitation of QC-based correction) — https://github.com/NBDZ/dbnorm
- **R** (Environment in which the log2 transformation, QC-detection screening, and lowess correction are executed)

## Examples

```
data <- read.csv("path/to/mydata.csv", sep = ",", header = TRUE, row.names = 1); library(dbnorm); excluded <- names(data[-1])[colSums(is.na(data[-1]) | data[-1] == 0) > 0]
```

## Evaluation signals

- The excluded-feature list contains exactly those features with zero or NA intensity in all surrogate pooled-QC injections; xanthosine 5'-monophosphate (detected in study samples but absent in QCs of the corresponding polarity) must appear in it
- qc_lowess_corrected_data.csv contains no excluded features and preserves sample/feature dimensions for the QC-injected subset
- Residual batch association of corrected intensities is reduced relative to raw data; note the article observed residual batch variability up to 0.78 (78%) for lowess-corrected SKIPOGH data, so some residual association may remain
- Input matrix was log2-transformed and provided as .csv with samples in rows, features in columns, and the batch level in the first column, per dbnorm's expected format

## Limitations

- QC-based lowess cannot correct metabolites below the limit of detection in pooled QCs — xanthosine 5'-monophosphate was filtered out by lowess whereas dbnorm statistical models retained it, demonstrating the coverage limitation
- Lowess correction left residual batch-associated variability in the SKIPOGH dataset (maximum 0.78, i.e. 78%), underperforming ber and parametric ComBat (maximum residual variability < 0.01)
- dbnorm's Visodbnorm and dbnormSCORE functions are suggested for fewer than 2000 features for better computational speed

## Evidence

- [results] lowess filtered out xanthosine 5'-monophosphate: "xanthosine 5′-monophosphate, which was filtered out with lowess"
- [readme] Data must be log2-transformed csv with batch in first column: "Data to be uploaded must be normalized and scaled on the log2 to account for the high abundance features (variables) by which technical heterogeneity might be overlooked. The input data must be in"
- [abstract] Some residual association remained in lowess-corrected data: [evidence span withheld — anchor status: not_found]
- [discussion] dbnorm provides efficient drift removal across batches: "We provide “dbnorm” as an efficient and user-friendly tool for the removal of drift across batches."
