---
name: blank-sample-contamination-assessment
description: Use when a feature-by-sample intensity table includes blank samples (labeled
  'blank' in the group-label row) and features must be flagged as blank-contaminated
  before selecting high-quality features for normalization.
when_to_use_negative:
- The input table contains no blank-sample columns (no 'blank' group labels in the
  first row), so no blank comparison is possible
- The goal is general feature quality assessment beyond blank contamination — the
  remaining MAFFIN criteria (RT range, QC RSD, serial-QC correlation) are separate
  skills
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MAFFIN
  role: R package whose FeatureSelection function implements the blank filter (BlankFilter
    parameter) and assigns the 'Blank' failure reason in the Quality column
  repo: https://github.com/HuanLab/MAFFIN
- name: R
  role: Runtime environment for installing MAFFIN via devtools and running the blank-filter
    comparison on the feature table
- name: devtools
  role: Used to install the MAFFIN package from GitHub before running FeatureSelection
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btac355
    title: 'MAFFIN: metabolomics sample normalization using maximal density fold change
      with high-quality metabolic features and corrected signal intensities'
  source_content_hash: sha256:f96ba96cbc1871f0983a875face29d48c45e201c63506a95609cf2c5f23d3786
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
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# blank-sample-contamination-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Screens metabolomics features for method-blank contamination by comparing sample intensity against blank-sample intensity, the first of MAFFIN's four orthogonal high-quality feature selection criteria used to exclude abiotic features from normalization factor calculation.

## When to use

Use when a feature-by-sample intensity table includes blank samples (labeled 'blank' in the group-label row) and features must be flagged as blank-contaminated before selecting high-quality features for normalization.

## When NOT to use

- The input table contains no blank-sample columns (no 'blank' group labels in the first row), so no blank comparison is possible
- The goal is general feature quality assessment beyond blank contamination — the remaining MAFFIN criteria (RT range, QC RSD, serial-QC correlation) are separate skills

## Inputs

- Feature-by-sample intensity dataframe (FeatureTable, e.g. MAFFIN's TestingData) whose first row carries group labels including 'blank' for blank samples
- Intensity threshold parameter (IntThreshold) defining how much sample intensity must exceed blank intensity

## Outputs

- Per-feature binary blank-filter flag
- Failure reason label 'Blank' for features failing the criterion, contributing to the composite Quality column of the normalized feature table

## How to apply

Parse the first row of the feature intensity table (feature rows, sample columns) to identify columns labeled 'blank'. For each feature, compare its intensity in real biological samples against its intensity in method-blank injections, applying an intensity threshold (IntThreshold in MAFFIN's FeatureSelection) so a feature passes only when sample intensity meaningfully exceeds blank intensity — MAFFIN's workflow implements this as sample intensity exceeding 2x the method-blank intensity. Assign a per-feature pass/fail flag and, for failures, record the reason label 'Blank', mirroring the Quality column MAFFIN adds to its output. Combine this decision with the other three orthogonal criteria (RT elution range, QC replicate RSD < 25%, and serial-QC Pearson correlation > 0.9) to obtain the composite quality label. Rationale: excluding features detected in blanks minimizes bias from abiotic features or metabolites with poor quantitative performance when computing normalization factors.

## Related tools

- **MAFFIN** (R package whose FeatureSelection function implements the blank filter (BlankFilter parameter) and assigns the 'Blank' failure reason in the Quality column) — https://github.com/HuanLab/MAFFIN
- **R** (Runtime environment for installing MAFFIN via devtools and running the blank-filter comparison on the feature table)
- **devtools** (Used to install the MAFFIN package from GitHub before running FeatureSelection)

## Examples

```
inputTable = read.csv(filename); MAFFINTable = MAFFINNorm(inputTable)
```

## Evaluation signals

- Features whose sample intensity does not exceed 2x the method-blank intensity must receive the 'Blank' failure label and be excluded from the high-quality feature set
- Standalone blank-filter flags must agree with the Blank failure reasons in MAFFIN's FeatureSelection output on the same TestingData demo table
- No feature flagged 'Blank' should appear among the retained HQ features used for normalization factor calculation
- The per-feature flag table must cover every row of the input FeatureTable with no missing decisions

## Limitations

- The blank criterion alone is insufficient: a feature passing the blank filter can still be rejected by the RT, LowRSD, or LowCor criteria, so the flag must be combined with the other three orthogonal criteria
- The check depends on correct group labeling — blank columns must be explicitly labeled 'blank' in the first row of the input table

## Evidence

- [abstract] MAFFIN uses multiple orthogonal criteria to select high-quality features for normalization factor calculation, which minimizes the bias caused by abiotic features or metabolites with poor: "MAFFIN uses multiple orthogonal criteria to select high-quality features for normalization factor calculation, which minimizes the bias caused by abiotic features or metabolites with poor"
- [methods] FeatureSelection signature: "FeatureSelection(FeatureTable, BlankFilter, RtRange, QCRSD, SQCcor, IntThreshold, SampleInCol, output)"
- [methods] MAFFIN main algorithm modules: "MAFFIN main algorithm consists of three modules, **high-quality feature selection, MS signal intensity correction, and maximal density fold change normalization**"
- [readme] Quality labels including Blank: "Quality` labels the feature quality. For low-quality features, MAFFIN elaborates the reasons including Blank, RT, LowRSD, and LowCor corresponding to four feature selection criteria."
- [readme] blank labeling instruction: "use **blank** for blank samples"
