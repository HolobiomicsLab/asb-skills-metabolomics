---
name: per-group-qc-criteria-merge
description: Use when an untargeted metabolomics feature table contains multiple pooled
  QC groups (separate pooled QC prepared per sample group) and features should be
  kept if they pass QC criteria in at least one QC pool.
when_to_use_negative:
- The study has a single pooled QC for all samples — use the standard QCs_process
  filtering without the sep_QC merge
- The analysis is targeted and relies on calibration-curve quantification rather than
  untargeted feature filtering
- The feature table has no blank or diluted-QC (QC_half) injections, so criteria III
  and IIII cannot be computed
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3361
tools:
- name: GetFeatistics
  role: Implements the QCs_process pooled-QC filtering, including the separated pooled-QC-per-sample-group
    variant and merge step
  repo: https://github.com/FrigerioGianfranco/GetFeatistics
- name: patRoon
  role: Upstream tool from which untargeted feature tables are imported before QC
    filtering
  repo: https://github.com/rickhelmus/patRoon
- name: MS-DIAL
  role: Alternative upstream source of feature tables imported into GetFeatistics
    for QC filtering
- name: R
  role: Runtime environment (version ≥ 4.3.1) in which GetFeatistics is installed
    and QCs_process is executed
  repo: https://github.com/FrigerioGianfranco/GetFeatistics
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1515/jib-2025-0047
    title: 'Streamlining feature elaboration and statistics analysis in metabolomics:
      the GetFeatistics R-package'
  source_content_hash: sha256:ed541e5cc99f5be5f22b1d1f4730ca9c2f8e56d92b978102a8adc7cf8ea1e92b
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# per-group-qc-criteria-merge

## Summary

A pooled-QC feature-filtering variant in GetFeatistics' QCs_process that computes QC quality cut-off criteria separately for each sample group's pooled QC pool, then merges results by retaining features that survive the criteria in at least one QC group.

## When to use

Use when an untargeted metabolomics feature table contains multiple pooled QC groups (separate pooled QC prepared per sample group) and features should be kept if they pass QC criteria in at least one QC pool.

## When NOT to use

- The study has a single pooled QC for all samples — use the standard QCs_process filtering without the sep_QC merge
- The analysis is targeted and relies on calibration-curve quantification rather than untargeted feature filtering
- The feature table has no blank or diluted-QC (QC_half) injections, so criteria III and IIII cannot be computed

## Inputs

- Feature table from patRoon or MS-DIAL with intensity columns for blank, pooled QC, and diluted-QC (QC_half) groups
- Sample-group metadata mapping pooled QC injections to their sample group (required for the separated pooled-QC variant)
- User-defined cut-offs: QC CV% threshold, QC detection frequency, blank/QC mean intensity ratio, QC/QC_half acceptability range

## Outputs

- Per-step pass/fail table (CSV) reporting which features fail each of the four QC criteria per QC group
- Final retained-feature table (CSV) with features surviving the per-group criteria in at least one QC group

## How to apply

In R, load the feature table (imported via GetFeatistics from patRoon or MS-DIAL output) so that blank, pooled QC, and diluted-QC (QC_half) sample groups are labelled. For each sample group's QC pool, run the four QCs_process criteria: (I) flag features whose CV% across pooled QC injections exceeds the reproducibility cut-off, (II) flag features with low detection frequency across QC injections, (III) flag features whose blank/QC mean intensity ratio shows insufficient excess over blanks, and (IIII) flag features whose QC/QC_half mean intensity ratio falls outside the acceptability range for dilution behaviour. Instead of intersecting all criteria across a single QC pool, evaluate the criteria per QC group and apply the sep_QC merge: a feature is retained if it passes all applicable criteria in at least one QC group. Write the per-step pass/fail results and the final retained-feature table to CSV so each filtering decision is auditable. The rationale is that group-specific QC pools capture group-specific matrix effects, so a feature reproducible in any group's pool is still analytically credible.

## Related tools

- **GetFeatistics** (Implements the QCs_process pooled-QC filtering, including the separated pooled-QC-per-sample-group variant and merge step) — https://github.com/FrigerioGianfranco/GetFeatistics
- **patRoon** (Upstream tool from which untargeted feature tables are imported before QC filtering) — https://github.com/rickhelmus/patRoon
- **MS-DIAL** (Alternative upstream source of feature tables imported into GetFeatistics for QC filtering)
- **R** (Runtime environment (version ≥ 4.3.1) in which GetFeatistics is installed and QCs_process is executed) — https://github.com/FrigerioGianfranco/GetFeatistics

## Examples

```
library(GetFeatistics); kept <- QCs_process(feature_table, sep_QC = TRUE)
```

## Evaluation signals

- For every discarded feature, the per-step pass/fail CSV identifies which of the four criteria (CV%, QC detection frequency, blank/QC ratio, QC/QC_half dilution ratio) it failed and in which QC group
- Every feature in the final retained table passes all four criteria in at least one QC group under the sep_QC merge
- Features failing all QC groups are exactly the complement of the retained table (no silent drops)
- Retained features show QC intensities sufficiently above blanks and QC/QC_half ratios consistent with the expected ~2-fold dilution behaviour
- The final CSV row/column counts match the input feature table dimensions minus the discarded features

## Limitations

- The article names the four criteria but the concrete numeric cut-offs (CV% threshold, blank/QC ratio, QC/QC_half acceptability range) are user-supplied and must be chosen per study
- The separated pooled-QC merge requires correctly labelled group-specific QC pools; misassigned QC metadata will incorrectly retain or drop features
- Retention by 'at least one QC group' means a feature may still be poorly reproducible in other groups' QC pools

## Evidence

- [abstract] filter steps: separated pooled-QC variant: "assign annotation confidence levels, and filter features based on pooled quality"
- [discussion] sep_QC merge is a distinctive capability: "the filtration of features considering pooled QC criteria, including also the use separated pooled QC per sample group"
- [intro] position of QC filtering in the untargeted workflow: "Once features have been extracted and potentially annotated, filtering based on QC quality cut-off criteria is typically performed"
- [readme] README confirms separate QC processing: "including elaboration of feature tables, separate QC processing, advanced statistics such as multiple regression linear models with mixed effects"
