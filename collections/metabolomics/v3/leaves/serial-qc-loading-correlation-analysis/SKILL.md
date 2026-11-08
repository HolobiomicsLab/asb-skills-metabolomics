---
name: serial-qc-loading-correlation-analysis
description: Use when a feature-by-sample intensity table includes serial QC columns
  labeled SQC_### (### = relative loading amount) and you must quantify how well each
  feature's intensity correlates with injected loading before normalization.
when_to_use_negative:
- The input table has no serial QC samples (no SQC_### columns with encoded loading
  amounts) — the correlation test cannot be computed and only blank, RT, and RSD criteria
  apply
- You only need a single normalization pass (MAFFINNorm) without inspecting per-criterion
  feature quality — call MAFFINNorm directly instead of reimplementing this filter
- Serial QC labels do not encode relative loading amounts (e.g. unlabeled pooled QC),
  making the intensity-vs-loading regression undefined
tools:
- name: MAFFIN
  role: R package providing FeatureSelection (SQCcor parameter) and TestingData; its
    Quality/LowCor labeling and SQC_points output define the expected result schema
  repo: github.com/HuanLab/MAFFIN
- name: R
  role: Runtime environment for parsing group labels, computing Pearson correlations
    across SQC_### columns, and cross-validating against FeatureSelection
- name: stringr
  role: MAFFIN import used to parse SQC_### column-name suffixes into numeric loading
    amounts
- name: devtools
  role: Used to install MAFFIN from GitHub before running the correlation analysis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btac355
    title: 'MAFFIN: metabolomics sample normalization using maximal density fold change
      with high-quality metabolic features and corrected signal intensities'
  source_content_hash: sha256:7af36485abe2f2441e821866c59b67f14a2e761f891b4195cfb5ecb96fc341bb
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/HuanLab/MAFFIN
  tools:
  - MAFFIN
  - R
  - devtools
  - stringr
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# serial-qc-loading-correlation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tests whether a metabolic feature's intensity tracks the relative loading amounts of serial QC samples (SQC_###) via Pearson correlation (r > 0.9), one of MAFFIN's four orthogonal high-quality-feature criteria used to pick features for normalization factor calculation.

## When to use

Use when a feature-by-sample intensity table includes serial QC columns labeled SQC_### (### = relative loading amount) and you must quantify how well each feature's intensity correlates with injected loading before normalization.

## When NOT to use

- The input table has no serial QC samples (no SQC_### columns with encoded loading amounts) — the correlation test cannot be computed and only blank, RT, and RSD criteria apply
- You only need a single normalization pass (MAFFINNorm) without inspecting per-criterion feature quality — call MAFFINNorm directly instead of reimplementing this filter
- Serial QC labels do not encode relative loading amounts (e.g. unlabeled pooled QC), making the intensity-vs-loading regression undefined

## Inputs

- Feature-by-sample intensity dataframe whose first row carries group labels, including serial QC columns labeled SQC_### (relative loading amounts)
- Mapping from SQC_### column labels to numeric loading amounts (parsed from the label suffix)
- Optional per-feature thresholds already applied by the other criteria (blank intensity > 2x method blank, RT within analytical gradient, QC RSD < 25%)

## Outputs

- Per-feature Pearson correlation of intensity vs serial-QC loading with pass/fail flag (LowCor) and composite Quality label
- Count of detected serial-QC points used in the correlation test per feature (MAFFIN's SQC_points)
- feature_selection_results.csv with criterion-level flags, quality labels, and SQC point counts

## How to apply

Parse the group-label row of the input table to isolate columns whose labels match the SQC_### convention, where '###' encodes the sample's relative loading amount (e.g. SQC_1, SQC_2, SQC_3 mean 1, 2 and 3 uL). For each feature row, extract its intensities across these serial-QC samples, order them by the encoded loading amount, and compute the Pearson correlation between intensity and loading. Retain the feature as high-quality if r > 0.9; features failing this test are labeled 'LowCor' in MAFFIN's Quality scheme (FeatureSelection's SQCcor parameter). Also record the number of detected serial-QC points used in the test, which MAFFIN reports as the SQC_points column and reuses downstream in IntCorrection. Rationale: metabolites with good quantitative performance scale with injected amount, so a strong intensity-loading correlation filters abiotic features and poorly behaving metabolites that would bias normalization factors.

## Related tools

- **MAFFIN** (R package providing FeatureSelection (SQCcor parameter) and TestingData; its Quality/LowCor labeling and SQC_points output define the expected result schema) — github.com/HuanLab/MAFFIN
- **R** (Runtime environment for parsing group labels, computing Pearson correlations across SQC_### columns, and cross-validating against FeatureSelection)
- **stringr** (MAFFIN import used to parse SQC_### column-name suffixes into numeric loading amounts)
- **devtools** (Used to install MAFFIN from GitHub before running the correlation analysis)

## Examples

```
devtools::install_github("Waddlessss/MAFFIN"); res <- FeatureSelection(TestingData, SQCcor=0.9, SampleInCol=TRUE, output=TRUE)
```

## Evaluation signals

- Every feature failing the test receives the 'LowCor' failure reason, consistent with MAFFIN's Quality-labeling scheme (Blank, RT, LowRSD, LowCor)
- SQC_points per feature equals the number of detected serial-QC columns with nonzero/usable intensity, matching the SQC_points column returned by MAFFINNorm
- Standalone Pearson r > 0.9 decisions agree with the SQCcor output of FeatureSelection on the same TestingData demo table
- Retained HQ features (composite of all four criteria) match the set MAFFINNorm would use for normalization factor calculation
- feature_selection_results.csv contains a row per feature with correlation values, pass/fail flags, and serial-QC point counts

## Limitations

- Features must elute within the analytical gradient and pass blank and RSD criteria in combination; the correlation test alone does not guarantee high quality (criteria are orthogonal)
- Correlation strength depends on having enough detected serial-QC points; too few detected SQC points makes the r estimate unreliable
- Group labels of real biological samples must not collide with reserved labels (blank, RT, QC, SQC_###) or stratification breaks

## Evidence
