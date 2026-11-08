---
name: metabolomics-feature-quality-filtering
description: Use when a feature-by-sample metabolomics intensity table must be filtered
  to retain only high-quality features before computing normalization factors, and
  per-feature failure reasons are needed.
when_to_use_negative:
- The goal is normalization evaluation (intragroup variation) rather than feature
  selection — use EvaPRMAD/EvaPRSD instead.
- Input lacks labeled blank, RT, QC, and serial-QC (SQC_###) sample groups, so the
  four criteria cannot be evaluated.
- Real biological sample group names collide with the reserved labels blank/RT/QC/SQC_###,
  which the README forbids.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MAFFIN
  role: R package providing FeatureSelection(FeatureTable, BlankFilter, RtRange, QCRSD,
    SQCcor, IntThreshold, SampleInCol, output) and the TestingData demo table
  repo: https://github.com/HuanLab/MAFFIN
- name: R
  role: Runtime environment in which MAFFIN is installed and executed
- name: devtools
  role: Used to install MAFFIN from GitHub via devtools::install_github("Waddlessss/MAFFIN")
- name: stringr
  role: MAFFIN dependency used for parsing SQC_### sample-group labels during stratification
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btac355
    title: 'MAFFIN: metabolomics sample normalization using maximal density fold change
      with high-quality metabolic features and corrected signal intensities'
  source_content_hash: sha256:26e13d1908b63e14952ec45bc0b900ff9113915afde1b70f8c33916a7b4817e4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/HuanLab/MAFFIN
  repo_url: https://github.com/HuanLab/MAFFIN
  tools:
  - MAFFIN
  - R
  - devtools
  - stringr
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# metabolomics-feature-quality-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A multi-criteria filtering procedure, implemented in MAFFIN's FeatureSelection function, that labels metabolomic features as high-quality (HQ) for use in normalization-factor calculation by rejecting abiotic features and metabolites with poor quantitative performance.

## When to use

Use when a feature-by-sample metabolomics intensity table must be filtered to retain only high-quality features before computing normalization factors, and per-feature failure reasons are needed.

## When NOT to use

- The goal is normalization evaluation (intragroup variation) rather than feature selection — use EvaPRMAD/EvaPRSD instead.
- Input lacks labeled blank, RT, QC, and serial-QC (SQC_###) sample groups, so the four criteria cannot be evaluated.
- Real biological sample group names collide with the reserved labels blank/RT/QC/SQC_###, which the README forbids.

## Inputs

- Feature intensity table (dataframe, features in rows, samples in columns) with a first row of group labels: blank, RT, QC, SQC_###
- BlankFilter, RtRange, QCRSD, SQCcor, IntThreshold parameter values
- SampleInCol orientation flag

## Outputs

- Quality-labeled feature table with per-feature pass/fail status and failure reasons (Blank, RT, LowRSD, LowCor)
- Set of high-quality features for downstream normalization-factor calculation
- feature_selection_results.csv (per-feature criterion-level flags and composite quality label)

## How to apply

Parse the first row of the feature intensity table (features in rows, samples in columns) into group labels: blank, RT, QC, and SQC_### serial-QC samples whose ### encodes relative loading amounts. Apply four orthogonal criteria per feature: (a) sample intensity exceeding the method-blank intensity at the IntThreshold (2x by default), (b) elution within the analytical RT gradient range via the RtRange annotation, (c) QC replicate relative standard deviation below the QCRSD cutoff (default RSDs < 25% following FDA's guideline), and (d) Pearson correlation of feature intensity vs serial-QC loading above the SQCcor cutoff (> 0.9). Combine the four binary decisions into a composite Quality label, recording failure reasons Blank, RT, LowRSD, or LowCor for rejected features. Only features passing all criteria are carried into MS signal intensity correction and MDFC normalization.

## Related tools

- **MAFFIN** (R package providing FeatureSelection(FeatureTable, BlankFilter, RtRange, QCRSD, SQCcor, IntThreshold, SampleInCol, output) and the TestingData demo table) — https://github.com/HuanLab/MAFFIN
- **R** (Runtime environment in which MAFFIN is installed and executed)
- **devtools** (Used to install MAFFIN from GitHub via devtools::install_github("Waddlessss/MAFFIN"))
- **stringr** (MAFFIN dependency used for parsing SQC_### sample-group labels during stratification)

## Examples

```
devtools::install_github("Waddlessss/MAFFIN"); inputTesting = TestingData; MAFFINTable = MAFFINNorm(inputTesting)
```

## Evaluation signals

- Every rejected feature carries one of the four failure reasons Blank, RT, LowRSD, or LowCor in its Quality column, matching the package's labeling scheme.
- Standalone per-criterion flags agree with MAFFIN's FeatureSelection output on TestingData for retained HQ features and rejection reasons.
- Each retained feature satisfies: sample intensity > 2x blank intensity, elution inside the RT gradient range, QC replicate RSD < 25%, and serial-QC Pearson correlation > 0.9.
- The number of detected serial-QC points used per feature is recorded and consistent with SQC group labels in the input row.
- Composite quality label equals the AND of the four binary criterion decisions for every feature.

## Limitations

- The four criteria depend on correctly labeled blank, RT, QC, and SQC_### sample groups in the input's first row; mislabeling breaks the filter.
- Real biological sample group names cannot be any of the reserved labels (blank, RT, QC, SQC_###).
- Selection minimizes bias from abiotic features or metabolites with poor quantitative performance, but features rejected for e.g. RSD slightly above 25% are excluded from normalization-factor calculation.

## Evidence

- [other] MAFFIN's high-quality feature selection requires that features elute during the analytical gradient and have low relative standard deviation of signal intensities in replicate QC injections, with a default RSDs < 25% threshold following FDA's guideline.: [evidence span withheld — anchor status: not_found]
