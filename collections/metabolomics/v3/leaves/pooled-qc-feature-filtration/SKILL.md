---
name: pooled-qc-feature-filtration
description: Use when a post-extraction untargeted metabolomics feature table (e.g.
  from XCMS, patRoon or MS-DIAL) contains blank, pooled QC and diluted-QC (QC_half)
  sample groups and unreliable features must be removed before statistics.
when_to_use_negative:
- Input is raw mass-spectrometry data — feature extraction must be done first with
  tools such as XCMS (or patRoon/MS-DIAL feature-table export) before QC filtration
- Targeted metabolomics quantification requiring calibration curves — this skill only
  covers pooled-QC-based feature filtration for untargeted feature tables
- The dataset contains no pooled QC, blank and diluted-QC (QC_half) injections, so
  the four QC criteria cannot be computed
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: GetFeatistics
  role: R package providing the QCs_process function that executes the four pooled-QC
    filters and the separated pooled-QC (sep_QC) per-sample-group merge
  repo: https://github.com/FrigerioGianfranco/GetFeatistics
- name: R
  role: Runtime language in which the feature table is loaded, filtered and exported
    to CSV
- name: patRoon
  role: Upstream tool whose exported untargeted feature tables are imported as the
    input for QC filtration
  repo: https://github.com/rickhelmus/patRoon
- name: MS-DIAL
  role: Alternative upstream feature-detection tool whose feature tables can be imported
    as the filtration input
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1515/jib-2025-0047
    title: 'Streamlining feature elaboration and statistics analysis in metabolomics:
      the GetFeatistics R-package'
  source_content_hash: sha256:60e98a261dfa56a43934d31e01a262060c1e26002c40b9b5f5a2f07d88d522ae
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# pooled-qc-feature-filtration

## Summary

Filter untargeted metabolomics features using pooled quality control criteria via the GetFeatistics QCs_process function, which removes features failing QC reproducibility (CV%), detection frequency, blank/QC intensity ratio, and dilution-response (QC vs QC_half) checks, optionally per sample-group QC pool.

## When to use

Use when a post-extraction untargeted metabolomics feature table (e.g. from XCMS, patRoon or MS-DIAL) contains blank, pooled QC and diluted-QC (QC_half) sample groups and unreliable features must be removed before statistics.

## When NOT to use

- Input is raw mass-spectrometry data — feature extraction must be done first with tools such as XCMS (or patRoon/MS-DIAL feature-table export) before QC filtration
- Targeted metabolomics quantification requiring calibration curves — this skill only covers pooled-QC-based feature filtration for untargeted feature tables
- The dataset contains no pooled QC, blank and diluted-QC (QC_half) injections, so the four QC criteria cannot be computed

## Inputs

- Feature table with intensity columns for blank, pooled QC and diluted-QC (QC_half) sample groups, imported via the GetFeatistics functions for patRoon or MS-DIAL feature tables
- Sample-group metadata assigning each column to blank / QC / QC_half / biological group
- User-set QC cut-offs (CV% threshold in pooled QCs, QC detection-frequency threshold, acceptable blank/QC mean-intensity ratio, acceptable QC/QC_half dilution-response range)

## Outputs

- Filtered feature table (retained features) written to CSV
- Per-step pass/fail evaluation table recording which features fail each of the four QC criteria and the sep_QC merge

## How to apply

Load the feature table into R with GetFeatistics, ensuring the sample groups include blanks, pooled QCs and diluted QCs (QC_half). QCs_process applies four sequential filters: (I) flag features whose pooled-QC intensity relative standard deviation (CV%) exceeds the chosen cut-off, (II) flag features with low detection frequency across QC injections, (III) flag features whose blank/QC mean intensity ratio shows insufficient contrast versus blanks, and (IIII) flag features whose QC/QC_half mean intensity ratio deviates from the expected dilution fold-change. If study batches used separated pooled QCs per sample group, apply the sep_QC variant: compute each criterion within each group's QC pool and retain features surviving the criteria in at least one QC group (merge step). Evaluate every filtering step by reporting which features pass/fail each criterion, then write the per-step pass/fail table and the final retained-feature table to CSV.

## Related tools

- **GetFeatistics** (R package providing the QCs_process function that executes the four pooled-QC filters and the separated pooled-QC (sep_QC) per-sample-group merge) — https://github.com/FrigerioGianfranco/GetFeatistics
- **R** (Runtime language in which the feature table is loaded, filtered and exported to CSV)
- **patRoon** (Upstream tool whose exported untargeted feature tables are imported as the input for QC filtration) — https://github.com/rickhelmus/patRoon
- **MS-DIAL** (Alternative upstream feature-detection tool whose feature tables can be imported as the filtration input)

## Examples

```
devtools::install_github("FrigerioGianfranco/GetFeatistics", dependencies=TRUE); library(GetFeatistics)
```

## Evaluation signals

- Every retained feature passes all four criteria (pooled-QC CV% within cut-off, sufficient QC detection frequency, blank/QC mean-intensity ratio below the acceptance limit, QC/QC_half ratio within the expected dilution fold-change range)
- The per-step pass/fail report accounts for all input features, and each step's feature count is monotonically consistent with sequential filtering
- Under the sep_QC variant, a feature is retained only if it survives all group-specific criteria in at least one sample group's QC pool
- The output CSV contains the final retained-feature table and matches the feature set implied by intersecting the four filter results

## Limitations

- Requires deliberate inclusion of pooled QC samples (and blank and diluted-QC / QC_half injections) during sample preparation; without them the filters cannot be evaluated
- The separated pooled-QC variant assumes QC pools exist per sample group and merges by keeping features passing in at least one group, which may retain features with poor reproducibility in other groups
- The package depends on R (>= 4.3.1) and a matching-architecture Java installation for full installation

## Evidence

- [abstract] For untargeted studies, it includes dedicated functions to import feature tables from tools like patRoon and MS-DIAL, assign annotation confidence levels, and filter features based on pooled quality: "import feature tables from tools like patRoon and MS-DIAL, assign annotation confidence levels, and filter features based on pooled quality"
- [intro] Once features have been extracted and potentially annotated, filtering based on QC quality cut-off criteria is typically performed: "Once features have been extracted and potentially annotated, filtering based on QC quality cut-off criteria is typically performed"
- [discussion] the filtration of features considering pooled QC criteria, including also the use separated pooled QC per sample group: "the filtration of features considering pooled QC criteria, including also the use separated pooled QC per sample group"
- [readme] Getting streamlined elaboration of targeted and non-targeted metabolomics data, including elaboration of feature tables, separate QC processing, advanced statistics: "Getting streamlined elaboration of targeted and non-targeted metabolomics data, including elaboration of feature tables, separate QC processing"
