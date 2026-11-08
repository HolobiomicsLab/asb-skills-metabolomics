---
name: percentile-based-common-feature-filtering
description: Use when a pcpfm feature table contains rare/uncommon features that should
  not influence per-sample normalization factors, or when 'removal of uncommon features'
  is needed before QC, normalization, or batch correction.
when_to_use_negative:
- The goal is blank masking, batch correction, or missing value imputation rather
  than restricting to/ removing uncommon features — pcpfm provides these as separate
  operations.
- The input is raw data (.mzML or .raw) rather than an already-preprocessed feature
  table; filtering operates after Asari preprocessing.
- The analysis legitimately requires rare features (e.g., xenobiotic-focused workflows
  like the Bowen et al 2023 sunitinib study) where dropping low-prevalence features
  would discard the signal of interest.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: pcpfm
  role: Pipeline whose 'normalize' operation applies the percentile-based common-feature
    restriction when computing per-sample TIC normalization factors
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: pandas
  role: Loads and filters the input feature table TSV by per-feature sample-presence
    percentile
- name: numpy
  role: Computes per-sample TIC and median/mean aggregation over retained features
- name: Asari
  role: Produces the upstream preferred feature table on which the percentile filter
    operates
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:e541780a5cb9e8aff7aa5546a63790d420bf26db62480e2276e65b27e15fdf2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
  tools:
  - Asari
  - Python
  - Python 3
  - numpy
  - pandas
  - pcpfm
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9545454545454546
n_claims: 22
---

# percentile-based-common-feature-filtering

## Summary

Restricts a pcpfm/Asari feature table to features present in at least a user-specified percentile of samples (e.g., TIC_normalization_percentile = 0.90) so that only common features contribute to downstream operations such as TIC-based normalization factor calculation.

## When to use

Use when a pcpfm feature table contains rare/uncommon features that should not influence per-sample normalization factors, or when 'removal of uncommon features' is needed before QC, normalization, or batch correction.

## When NOT to use

- The goal is blank masking, batch correction, or missing value imputation rather than restricting to/ removing uncommon features — pcpfm provides these as separate operations.
- The input is raw data (.mzML or .raw) rather than an already-preprocessed feature table; filtering operates after Asari preprocessing.
- The analysis legitimately requires rare features (e.g., xenobiotic-focused workflows like the Bowen et al 2023 sunitinib study) where dropping low-prevalence features would discard the signal of interest.

## Inputs

- Asari preferred feature table TSV (e.g., asari_results/preferred_Feature_table.tsv or full_feature_table.tsv)
- Sample metadata / sequence CSV (including sample type and, for multi-batch experiments, a 'Batch' field)
- Percentile threshold (TIC_normalization_percentile, e.g. 0.90) and aggregation choice (median default or mean)

## Outputs

- Percentile-filtered feature table restricted to features present in >= the specified percentile of samples
- Per-sample TIC-based normalization factors derived from the retained common features
- Normalized table written to a new moniker TSV (e.g., preferred_normalized.tsv) in the experiment's feature_tables directory

## How to apply

Load the Asari preferred feature table TSV with pandas and compute, for each feature, the fraction of samples in which it is present. Retain only features present in at least the configured percentile (the paper's example uses TIC_normalization_percentile = 0.90), so only common features contribute to the subsequent total ion current (TIC) computation. In the normalization workflow, compute each sample's TIC over only these retained features and derive the per-sample factor from the median (default) or mean; dividing sample intensities by that factor yields the one-pass corrected table. If batch-aware mode is enabled (--by_batch), apply the percentile restriction and per-batch normalization within each batch first, then scale batches to one another using median TIC values from conserved features (two-pass). The rationale is that low-prevalence features would otherwise skew per-sample TIC and thus the normalization factor, so restricting to common features keeps the factor representative across samples.

## Related tools

- **pcpfm** (Pipeline whose 'normalize' operation applies the percentile-based common-feature restriction when computing per-sample TIC normalization factors) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **pandas** (Loads and filters the input feature table TSV by per-feature sample-presence percentile)
- **numpy** (Computes per-sample TIC and median/mean aggregation over retained features)
- **Asari** (Produces the upstream preferred feature table on which the percentile filter operates)

## Evaluation signals

- The retained feature set contains only features present in at least the configured percentile (e.g., 90%) of samples — check per-feature presence counts before and after filtering.
- The output TSV exists in the feature_tables directory (e.g., preferred_normalized.tsv) and has the same samples and features as the input table.
- Post-normalization per-sample TIC values computed over the retained features are consistent across samples rather than driven by per-sample feature counts.
- In --by_batch mode, per-batch median TIC values over conserved features converge after the two-pass cross-batch scaling step.

## Limitations

- The percentile cutoff is user-specified (0.90 in the paper's example); too strict a threshold on small cohorts can leave few features to compute stable TIC normalization factors.
- The filter restricts which features contribute to normalization factors; it is not itself an annotation-quality step — low signal-to-noise features are separately excluded by the Asari quality threshold.
- Batch-aware two-pass normalization depends on conserved common features across batches; if batches share few percentile-passing features, cross-batch scaling may be unreliable.

## Evidence

- [intro] Operations to correct common data quality issues are provided including normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation.: "Operations to correct common data quality issues are provided including normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation."
- [intro] including normalization, feature interpolation, removal of rare features, quality assurance, quality control evaluations, and generates PDF reports: "including normalization, feature interpolation, removal of rare features, quality assurance, quality control evaluations, and generates PDF reports"
- [readme] This includes feature tables that are optionally blank masked, normalized, batch corrected, annotated or otherwise curated by PCPFM: "This includes feature tables that are optionally blank masked, normalized, batch corrected, annotated or otherwise curated by PCPFM"
