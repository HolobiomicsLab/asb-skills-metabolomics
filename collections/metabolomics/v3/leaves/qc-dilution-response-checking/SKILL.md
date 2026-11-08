---
name: qc-dilution-response-checking
description: Use when an untargeted metabolomics feature table contains pooled QC
  and diluted-QC (QC_half) injections and features must be removed whose dilution
  behaviour does not follow the expected fold-change, or when pooled-QC-based filtering
  more broadly is needed.
when_to_use_negative:
- Targeted analyses, where GetFeatistics instead supports calibration curve-based
  quantification with different data weighting options
- Feature tables lacking diluted-QC (QC_half) injections, since the dilution-response
  criterion requires a QC vs QC_half intensity comparison
- Raw MS data files that still require feature extraction (e.g. with XCMS) before
  QC cut-off filtering can be applied
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: GetFeatistics
  role: R package providing the QCs_process pooled-QC filtering function, including
    the QC/QC_half dilution-response criterion and the separated pooled-QC per-group
    variant
  repo: https://github.com/FrigerioGianfranco/GetFeatistics
- name: patRoon
  role: Source of untargeted feature tables that GetFeatistics imports before applying
    pooled-QC filtering
  repo: https://github.com/rickhelmus/patRoon
- name: MS-DIAL
  role: Alternative source of untargeted feature tables that GetFeatistics imports
    before applying pooled-QC filtering
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1515/jib-2025-0047
    title: 'Streamlining feature elaboration and statistics analysis in metabolomics:
      the GetFeatistics R-package'
  source_content_hash: sha256:a206d502d6470fe4c738d11a3b8bc8753a1d53ac642aa060c7b9f71cd0bd3b94
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrigerioGianfranco/GetFeatistics
  tools:
  - GetFeatistics
  - MS-DIAL
  - R
  - patRoon
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# qc-dilution-response-checking

## Summary

Filters untargeted metabolomics features by checking dilution response in pooled quality-control samples — the QC vs diluted-QC (QC_half) mean-intensity ratio criterion (step IIII of GetFeatistics' four-step QCs_process) — alongside the companion QC criteria (CV% reproducibility, QC detection frequency, blank/QC ratio).

## When to use

Use when an untargeted metabolomics feature table contains pooled QC and diluted-QC (QC_half) injections and features must be removed whose dilution behaviour does not follow the expected fold-change, or when pooled-QC-based filtering more broadly is needed.

## When NOT to use

- Targeted analyses, where GetFeatistics instead supports calibration curve-based quantification with different data weighting options
- Feature tables lacking diluted-QC (QC_half) injections, since the dilution-response criterion requires a QC vs QC_half intensity comparison
- Raw MS data files that still require feature extraction (e.g. with XCMS) before QC cut-off filtering can be applied

## Inputs

- Feature table with intensities for blank, pooled QC, and diluted-QC (QC_half) sample groups (e.g. imported from patRoon or MS-DIAL)
- User-defined QC cut-offs: CV% threshold, QC detection-frequency threshold, blank/QC mean-intensity ratio cut-off, and acceptable range of QC/QC_half mean-intensity ratio

## Outputs

- Per-step pass/fail results for each feature across the four QC criteria and the sep_QC merge
- Final retained-feature table written to a CSV file

## How to apply

Load the constructed feature table in R (GetFeatistics) with blank, QC, and diluted-QC (QC_half) sample groups. As context for the dilution step, the four-step QCs_process first flags features with poor reproducibility (CV% of QC intensities), low detection frequency across QC injections, and high blank contribution (blank/QC mean-intensity ratio). Then, for the dilution-response criterion itself, compute the ratio of QC to diluted-QC (QC_half) mean intensities and flag features whose ratio falls outside the range of acceptability expected for the dilution fold-change. If separate pooled QCs were acquired per sample group, apply the sep_QC variant: compute the QC criteria per group and retain features that survive the criteria in at least one QC group (merge step). Write the per-step pass/fail results and the final retained-feature table to a CSV file.

## Related tools

- **GetFeatistics** (R package providing the QCs_process pooled-QC filtering function, including the QC/QC_half dilution-response criterion and the separated pooled-QC per-group variant) — https://github.com/FrigerioGianfranco/GetFeatistics
- **patRoon** (Source of untargeted feature tables that GetFeatistics imports before applying pooled-QC filtering) — https://github.com/rickhelmus/patRoon
- **MS-DIAL** (Alternative source of untargeted feature tables that GetFeatistics imports before applying pooled-QC filtering)

## Examples

```
devtools::install_github("FrigerioGianfranco/GetFeatistics", dependencies = TRUE); library(GetFeatistics)
```

## Evaluation signals

- Features flagged by the dilution step are exactly those whose QC/QC_half mean-intensity ratio falls outside the accepted range for the expected fold-change
- Retained features pass all four criteria (QC CV%, QC detection frequency, blank/QC ratio, QC/QC_half dilution ratio) or survive the sep_QC merge in at least one QC group
- Per-step pass/fail CSV counts decrease monotonically through the four filtering steps, and the final retained-feature table matches the union after the sep_QC merge
- Both the per-step pass/fail results and the retained-feature table are successfully written to CSV files
- Sep_QC merge correctness: a feature failing criteria in one sample group's QC pool but passing in another is retained, not dropped

## Limitations

- The dilution-response criterion requires dedicated diluted-QC (QC_half) injections; without them, only the CV%, detection-frequency, and blank/QC criteria can be applied
- The sep_QC merge keeps features that survive the criteria in at least one QC group, which can retain features that fail QC in other sample groups
- Specific numeric cut-off values (e.g. acceptable QC/QC_half ratio range) are user-supplied and not fixed in the article text

## Evidence

- [abstract] Four-step pooled QC filtering including dilution criterion: "For untargeted studies, it includes dedicated functions to import feature tables from tools like patRoon and MS-DIAL, assign annotation confidence levels, and filter features based on pooled quality"
- [intro] QC cut-off filtering placed in the untargeted workflow: "Once features have been extracted and potentially annotated, filtering based on QC quality cut-off criteria is typically performed"
- [intro] Role of pooled QC samples motivating the filtering: "the inclusion of pooled quality control (QC) samples during sample preparation is a key step to monitor data quality and analytical reproducibility"
- [discussion] Separated pooled-QC per-group variant: "the filtration of features considering pooled QC criteria, including also the use separated pooled QC per sample group"
- [readme] Separate QC processing named in the README: "Getting streamlined elaboration of targeted and non-targeted metabolomics data, including elaboration of feature tables, separate QC processing"
