---
name: qc-replicate-precision-rsd-analysis
description: Use when a feature-by-sample intensity table includes replicate QC sample
  columns and features must be screened for quantitative reliability before computing
  normalization factors.
when_to_use_negative:
- The input table has no replicate QC columns (labeled QC in the group-label row),
  so precision cannot be assessed
- The goal is sample normalization itself (MDFC, median, quantile) rather than high-quality
  feature selection — use MAFFINNorm/FeatureSelection pipeline scope instead
- Features have already been quality-filtered and only signal-intensity correction
  or normalization remains
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MAFFIN
  role: R package whose FeatureSelection function implements the QC replicate RSD
    < 25% filter (LowRSD flag) as one of four orthogonal high-quality feature criteria
  repo: https://github.com/HuanLab/MAFFIN
- name: R
  role: Runtime environment for computing replicate-QC RSDs and running MAFFIN's FeatureSelection
    on the TestingData demo table
  repo: https://github.com/HuanLab/MAFFIN
- name: devtools
  role: Used to install the MAFFIN R package from GitHub
  repo: https://github.com/Waddlessss/MAFFIN
- name: stringr
  role: MAFFIN package dependency (Imports) used for parsing group labels such as
    QC and SQC_### serial-QC sample names
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btac355
    title: 'MAFFIN: metabolomics sample normalization using maximal density fold change
      with high-quality metabolic features and corrected signal intensities'
  source_content_hash: sha256:91eee682e0ec1ce4a1bc6096dc2fadc356e4586ae25a1ba3ff5b392ce01e4f41
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# qc-replicate-precision-rsd-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filters metabolic features by replicate precision: computes the relative standard deviation (RSD) of signal intensities across replicate QC injections and retains only features with RSD < 25%, the FDA-guideline threshold MAFFIN uses to identify high-quality features for normalization factor calculation.

## When to use

Use when a feature-by-sample intensity table includes replicate QC sample columns and features must be screened for quantitative reliability before computing normalization factors.

## When NOT to use

- The input table has no replicate QC columns (labeled QC in the group-label row), so precision cannot be assessed
- The goal is sample normalization itself (MDFC, median, quantile) rather than high-quality feature selection — use MAFFINNorm/FeatureSelection pipeline scope instead
- Features have already been quality-filtered and only signal-intensity correction or normalization remains

## Inputs

- Feature intensity dataframe (features in rows, samples in columns) with a first row of group labels, including QC labels for replicate quality-control injections (e.g., MAFFIN's TestingData)

## Outputs

- Per-feature table with QC RSD pass/fail flags (LowRSD failure reason) and composite Quality label
- Set of high-quality features (RSD < 25%) available for normalization factor calculation
- feature_selection_results.csv (Quality-labeled per-feature table)

## How to apply

Stratify the feature table's group-label row (first row of a dataframe like MAFFIN's TestingData) to isolate the columns labeled QC, which are replicate QC injections. For each feature, compute the RSD of signal intensities across these replicate QC samples. Flag features with RSD >= 25% as low-precision (failure reason LowRSD in MAFFIN's Quality label), keeping RSD < 25% as the pass threshold following FDA's guideline. Combine this binary decision with the other orthogonal criteria (blank filtering, RT gradient range, serial-QC correlation > 0.9) as implemented in MAFFIN's FeatureSelection(FeatureTable, BlankFilter, RtRange, QCRSD, SQCcor, IntThreshold, SampleInCol, output), or run it standalone and cross-validate against FeatureSelection output. Emit a per-feature table with criterion-level flags, the composite quality label, and failure reasons.

## Related tools

- **MAFFIN** (R package whose FeatureSelection function implements the QC replicate RSD < 25% filter (LowRSD flag) as one of four orthogonal high-quality feature criteria) — https://github.com/HuanLab/MAFFIN
- **R** (Runtime environment for computing replicate-QC RSDs and running MAFFIN's FeatureSelection on the TestingData demo table) — https://github.com/HuanLab/MAFFIN
- **devtools** (Used to install the MAFFIN R package from GitHub) — https://github.com/Waddlessss/MAFFIN
- **stringr** (MAFFIN package dependency (Imports) used for parsing group labels such as QC and SQC_### serial-QC sample names)

## Examples

```
devtools::install_github("Waddlessss/MAFFIN"); results <- FeatureSelection(TestingData, BlankFilter=TRUE, RtRange=TRUE, QCRSD=0.25, SQCcor=0.9, IntThreshold=2, SampleInCol=TRUE, output=TRUE)
```

## Evaluation signals

- Features with replicate-QC RSD >= 25% carry the LowRSD failure reason in the Quality column
- Standalone RSD filter results agree with MAFFIN's FeatureSelection output on the same demo table (TestingData) in both retained HQ features and per-criterion rejection reasons
- RSD is computed only over columns labeled QC in the group-label row, excluding blank, RT, SQC_###, and biological sample columns
- Every feature receives both a criterion-level flag and a composite quality label; no feature is silently dropped

## Limitations

- The 25% RSD cutoff is a default following FDA's guideline; features marginally above the threshold are excluded even if they pass the other three criteria
- Group names of real biological samples must not collide with reserved labels (blank, RT, QC, SQC_###) or cohort stratification will misassign columns
- RSD precision alone is insufficient — MAFFIN requires the four criteria jointly (blank intensity > 2x, RT within gradient, RSD < 25%, serial-QC correlation > 0.9)

## Evidence

- [methods] high-quality feature selection requires that features elute during the analytical gradient and have low relative standard deviation of signal intensities in replicate QC injections, with a default RSDs < 25% threshold following FDA's guideline: [evidence span withheld — anchor status: not_found]
