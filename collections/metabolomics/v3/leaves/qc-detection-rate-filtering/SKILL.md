---
name: qc-detection-rate-filtering
description: Use when a preprocessed LC-MS peak table (e.g. MS-DIAL Excel export)
  must be quality-filtered before statistics, and pooled QC samples are available
  to compute detection rates and Broadhurst RSD/D-ratio metrics.
when_to_use_negative:
- The input is raw LC-MS data that has not yet been peak-picked into a feature table
  (notame expects preprocessed LC-MS data).
- The dataset has no pooled QC samples, so QC detection rate and QC-based RSD/D-ratio
  cannot be computed.
- The goal is hard removal of features rather than flagging — notame's design flags
  low-quality features because they may aid metabolite identification.
edam_operation: http://edamontology.org/operation_3675
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0219
tools:
- name: notame
  role: R package providing mark_nas, flag_detection and flag_quality to compute QC
    detection rates and Broadhurst quality metrics on a MetaboSet
  repo: https://github.com/hanhineva-lab/notame
- name: R
  role: Runtime environment in which the notame flagging workflow is executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:40f0a280f924c3bbfd6a591ba31921b72a0cb62edff55165f219f2be53df856f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hanhineva-lab/notame
  tools:
  - R
  - notame
  edam_operation: http://edamontology.org/operation_3675
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# qc-detection-rate-filtering

## Summary

Flags low-quality molecular features in a MetaboSet (notame) based on QC-sample detection rate and Broadhurst quality metrics (RSD, D-ratio), so only reproducible features proceed to downstream analysis while flagged features are retained for metabolite identification.

## When to use

Use when a preprocessed LC-MS peak table (e.g. MS-DIAL Excel export) must be quality-filtered before statistics, and pooled QC samples are available to compute detection rates and Broadhurst RSD/D-ratio metrics.

## When NOT to use

- The input is raw LC-MS data that has not yet been peak-picked into a feature table (notame expects preprocessed LC-MS data).
- The dataset has no pooled QC samples, so QC detection rate and QC-based RSD/D-ratio cannot be computed.
- The goal is hard removal of features rather than flagging — notame's design flags low-quality features because they may aid metabolite identification.

## Inputs

- Preprocessed LC-MS peak table exported from MS-DIAL as an Excel spreadsheet (e.g. sample_data_whole.xlsx) with sample metadata including a Group column
- MetaboSet object constructed with construct_metabosets from the peak table
- Pooled QC sample annotations identifying which samples are QCs

## Outputs

- Per-feature flag table (feature_quality_flags.csv) containing the Flag column plus detection-rate and quality metrics (RSD, RSD*, D-ratio, D-ratio*)
- MetaboSet with updated featureData flags marking low-quality features for exclusion from downstream statistical analysis

## How to apply

Read the MS-DIAL peak table (e.g. sample_data_whole.xlsx) with read_from_excel and build a MetaboSet via construct_metabosets(group_col = "Group"). First run mark_nas(value = 0) so zero abundances become NA, then flag_detection with qc_limit = 0.7 (group_limit = 0.8) to flag features not detected in at least 70% of QC samples. Next run flag_quality, which computes the robust RSD* (1.4826*MAD/median of QCs) and D-ratio* (MAD_QC/MAD_biological) on QC samples and flags features with RSD* >= 0.2 or D-ratio* >= 0.4, per Broadhurst et al. Extract the Flag column and the detection-rate/quality metrics from fData and write them to feature_quality_flags.csv; keep flagged features in the data rather than removing them, since low-quality features may still aid metabolite identification.

## Related tools

- **notame** (R package providing mark_nas, flag_detection and flag_quality to compute QC detection rates and Broadhurst quality metrics on a MetaboSet) — https://github.com/hanhineva-lab/notame
- **R** (Runtime environment in which the notame flagging workflow is executed)

## Examples

```
set <- mark_nas(construct_metabosets(read_from_excel("sample_data_whole.xlsx"), group_col = "Group"), value = 0); set <- flag_detection(set, qc_limit = 0.7, group_limit = 0.8); set <- flag_quality(set)
```

## Evaluation signals

- The Flag column in feature_quality_flags.csv is populated for every feature in fData, with no missing flags.
- All retained features have QC detection rate >= 0.7 (qc_limit), robust RSD* < 0.2 and D-ratio* < 0.4.
- mark_nas(value = 0) was applied first, so zero abundances appear as NA in the exprs matrix before flagging.
- Flagged features remain present in the MetaboSet (flagged, not dropped), consistent with keeping them for metabolite identification.
- Visual QC (e.g. notame's QC plots) shows retained features are stable across the run relative to flagged ones.

## Limitations

- The package API is experimental and breaking changes are possible; the original antonvsdata/notame repo is no longer maintained and active development moved to hanhineva-lab/notame.
- Thresholds (detection rate 0.7, RSD* 0.2, D-ratio* 0.4) are lab-specific conventions from Broadhurst et al. and may need adjustment for other studies.
- Low-quality features are only flagged, not removed, so downstream analysis code must explicitly exclude flagged features.

## Evidence

- [methods] ```flag_detection``` is used to flag features based on detection: "```flag_detection``` is used to flag features based on detection"
- [methods] ```flag_quality``` is used to flag features based on the other quality metrics: "```flag_quality``` is used to flag features based on the other quality metrics"
- [readme] Identifying and flagging (or removing) low-quality molecular features using quality metrics defined by [Broadhurst et al.]: [evidence span withheld — anchor status: external_doc]
- [methods] LC-MS metabolomics datasets contain a significant number of low-quality features: "LC-MS metabolomics datasets contain a significant number of low-quality features"
- [methods] ```mark_nas``` replaces given values in ```exprs``` part of the MetaboSet object with ```NA```.: "```mark_nas``` replaces given values in ```exprs``` part of the MetaboSet object with ```NA```."
