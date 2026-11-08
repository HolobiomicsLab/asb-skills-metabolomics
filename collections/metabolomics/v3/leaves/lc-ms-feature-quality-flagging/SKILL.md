---
name: lc-ms-feature-quality-flagging
description: 'Use when a preprocessed LC-MS peak table (e.g. MS-DIAL output) must
  be quality-controlled before statistics: features detected in <70% of QC samples
  or with RSD ≥ 0.2 / D-ratio ≥ 0.4 need flagging.'
when_to_use_negative:
- The data is still raw instrument output (e.g. mzML/NetCDF) and needs peak picking
  first — notame expects an already preprocessed peak table
- The goal is to permanently discard poor features rather than flag them — notame
  flags low-quality features because they may aid metabolite identification
- The dataset has no pooled QC samples, since flag_detection and flag_quality thresholds
  are computed on QC samples
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: notame
  role: R package providing mark_nas, flag_detection and flag_quality used to flag
    low-quality features using Broadhurst et al. quality metrics
  repo: https://github.com/antonvsdata/notame
- name: R
  role: Runtime environment in which the notame flagging workflow is executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:22f6a3ce2efe9e8babf8c5ce1826964e42141f63f8489ef20cf2a893ee1ce2c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/antonvsdata/notame
  tools:
  - R
  - notame
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# lc-ms-feature-quality-flagging

## Summary

Flag low-quality molecular features in preprocessed LC-MS metabolomics data using QC-sample detection rate plus Broadhurst et al. quality metrics (RSD and D-ratio), retaining flagged features in the table for downstream use.

## When to use

Use when a preprocessed LC-MS peak table (e.g. MS-DIAL output) must be quality-controlled before statistics: features detected in <70% of QC samples or with RSD ≥ 0.2 / D-ratio ≥ 0.4 need flagging.

## When NOT to use

- The data is still raw instrument output (e.g. mzML/NetCDF) and needs peak picking first — notame expects an already preprocessed peak table
- The goal is to permanently discard poor features rather than flag them — notame flags low-quality features because they may aid metabolite identification
- The dataset has no pooled QC samples, since flag_detection and flag_quality thresholds are computed on QC samples

## Inputs

- MS-DIAL-generated peak table spreadsheet (e.g. .xlsx) with sample metadata including a Group column distinguishing QC samples
- MetaboSet object with an exprs abundance matrix

## Outputs

- Per-feature quality flag table (feature_quality_flags.csv) with the Flag column, QC detection rates and quality metrics
- MetaboSet objects with updated featureData containing Flag, RSD* and D-ratio values

## How to apply

Load the MS-DIAL-derived peak table (e.g. sample_data_whole.xlsx) with read_from_excel and construct_metabosets(group_col = 'Group') to build MetaboSet objects, then call mark_nas(value = 0) so zero abundances become NA. Run flag_detection with qc_limit = 0.7 (group_limit = 0.8) to flag features not observed in at least 70% of QC samples. Then run flag_quality, which computes the robust RSD* (1.4826 × MAD / median of QC abundances) and D-ratio* (MAD_QC / MAD_biological) on QC samples and flags features with RSD* ≥ 0.2 or D-ratio* ≥ 0.4, per the Broadhurst et al. quality metrics; signals whose classic RSD, RSD* and D-ratio are all below 0.1 are kept unflagged. Export the Flag column together with detection-rate and quality metrics from fData to feature_quality_flags.csv; features are flagged rather than removed because low-quality signals may still aid metabolite identification.

## Related tools

- **notame** (R package providing mark_nas, flag_detection and flag_quality used to flag low-quality features using Broadhurst et al. quality metrics) — https://github.com/antonvsdata/notame
- **R** (Runtime environment in which the notame flagging workflow is executed)

## Examples

```
library(notame); set <- construct_metabosets(read_from_excel("sample_data_whole.xlsx", sheet = 1), group_col = "Group"); set <- mark_nas(set, value = 0); set <- flag_detection(set, qc_limit = 0.7); set <- flag_quality(set); write.csv(fData(set), "feature_quality_flags.csv")
```

## Evaluation signals

- All features detected in fewer than 70% of QC samples carry a detection flag in the output table
- Every flagged feature satisfies RSD* >= 0.2 or D-ratio* >= 0.4 on QC samples; no feature with RSD, RSD* and D-ratio all below 0.1 is flagged for quality
- RSD* values in the output equal 1.4826 × MAD / median of QC abundances, and D-ratio equals MAD_QC / MAD_biological
- feature_quality_flags.csv exists, contains one row per feature, and includes the Flag column plus detection-rate and quality metric columns
- Zero abundances in the original peak table appear as NA after mark_nas(value = 0)

## Limitations

- Flagging is advisory — flagged low-quality features are retained in the data and must be filtered explicitly downstream
- Thresholds 0.7 (QC detection), 0.2 (RSD*) and 0.4 (D-ratio) are defaults; group_limit = 0.8 applies to group-wise detection and may need adjustment
- The notame API is still quite experimental and breaking changes are possible; the original antonvsdata/notame repo is no longer maintained (active development moved to hanhineva-lab/notame)

## Evidence

- [methods] flag_quality: "```flag_quality``` is used to flag features based on the other quality metrics"
- [methods] flag_detection: "```flag_detection``` is used to flag features based on detection"
- [methods] mark_nas: "```mark_nas``` replaces given values in ```exprs``` part of the MetaboSet object with ```NA```."
- [readme] flagging low-quality features: [evidence span withheld — anchor status: external_doc]
- [intro] flag rather than remove rationale: "LC-MS metabolomics datasets contain a significant number of low-quality features"
