---
name: batch-aware-two-pass-normalization
description: Use when a feature table (e.g., the Asari preferred_feature_table.tsv
  from a pcpfm experiment) shows per-sample intensity differences unrelated to biology
  — e.
when_to_use_negative:
- The input is raw .mzML/.raw data — preprocessing via Asari must produce a feature
  table before normalization applies.
- True feature-level batch correction is needed (e.g., removing systematic batch-specific
  feature drift); this skill only rescales per-sample intensities via TIC and does
  not perform other batch correction operations pcpfm offers separately.
- Normalization must be based on internal standards or QC-sample drift correction
  rather than total ion current of common features.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: pcpfm
  role: Provides the 'normalize' command that computes TIC-based normalization factors,
    supports --by_batch two-pass mode, and writes the new moniker feature table into
    the experiment's feature_tables directory.
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: pandas
  role: Used to load and manipulate the input feature table TSV when reconstructing
    or applying the normalization logic.
- name: numpy
  role: Supports numeric aggregation (median/mean) for per-sample normalization factors.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:25d51f5c455beb87489c27d4a6da024125d12fd2fc0f817908d902f5b2187ec9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
  tools:
  - Python
  - Python 3
  - numpy
  - pandas
  - pcpfm
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# batch-aware-two-pass-normalization

## Summary

Computes per-sample TIC-based normalization factors for LC-MS feature tables using only features present in at least a user-specified percentile of samples (e.g., 90%), with an optional two-pass batch-aware mode that first normalizes within batches and then scales batches to each other via median TIC of conserved features. This is pcpfm's 'normalize' operation for correcting per-sample intensity bias before downstream analysis.

## When to use

Use when a feature table (e.g., the Asari preferred_feature_table.tsv from a pcpfm experiment) shows per-sample intensity differences unrelated to biology — e.g., total ion current varies with per-sample feature counts — and intensities must be normalized, optionally across multiple acquisition batches.

## When NOT to use

- The input is raw .mzML/.raw data — preprocessing via Asari must produce a feature table before normalization applies.
- True feature-level batch correction is needed (e.g., removing systematic batch-specific feature drift); this skill only rescales per-sample intensities via TIC and does not perform other batch correction operations pcpfm offers separately.
- Normalization must be based on internal standards or QC-sample drift correction rather than total ion current of common features.

## Inputs

- Asari preferred feature table TSV (or other pcpfm feature table) with per-sample feature intensities
- Sample metadata / Batch field (required for --by_batch mode to define batches)
- User parameters: TIC_normalization_percentile (e.g., 0.90), aggregation choice (median default or mean), --by_batch flag, output moniker

## Outputs

- Normalized feature table written as a new moniker TSV (e.g., preferred_normalized.tsv) in the experiment's feature_tables directory
- Per-sample normalization factors derived from TIC over conserved features

## How to apply

Load the input feature table TSV (e.g., the Asari preferred feature table) with pandas and restrict normalization-factor calculations to features present in at least the TIC_normalization_percentile of samples (0.90 in the paper's example), so only common features contribute to the factors. Compute each sample's total ion current (TIC) over the retained features and derive the per-sample normalization factor from the median value (default) or mean; dividing each sample's feature intensities by its factor yields the one-pass corrected table. If batch-aware mode is requested (--by_batch), first normalize each batch independently, then scale batches to one another using their median TIC values computed from the conserved features — this is the two-pass approach. Rationale: using only widely present features prevents rare or noisy features from distorting the TIC-based factor, and normalizing within batches before cross-batch scaling prevents batch-level intensity offsets from propagating into the factors.

## Related tools

- **pcpfm** (Provides the 'normalize' command that computes TIC-based normalization factors, supports --by_batch two-pass mode, and writes the new moniker feature table into the experiment's feature_tables directory.) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **pandas** (Used to load and manipulate the input feature table TSV when reconstructing or applying the normalization logic.)
- **numpy** (Supports numeric aggregation (median/mean) for per-sample normalization factors.)

## Examples

```
pcpfm normalize --input_feature_table preferred_feature_table.tsv --new_moniker preferred_normalized --by_batch
```

## Evaluation signals

- Output TSV exists at the expected new-moniker path (e.g., preferred_normalized.tsv) in the feature_tables directory.
- Output has the same samples and features as the input table (no rows or columns dropped by normalization).
- Post-normalization per-sample TIC values are consistent across samples rather than driven by per-sample feature counts.
- In batch-aware mode, within-batch TICs align after pass one and cross-batch median TICs align after pass two (scaling by conserved-feature median TIC).
- Normalization factors were computed from features present in at least the specified percentile of samples (e.g., ≥90%).

## Limitations

- Factor derivation depends on feature presence thresholds — features present in fewer than the specified percentile of samples (e.g., 10% of samples) are excluded from factor calculation, which can shift factors if many features are sparse.
- Median (default) vs mean aggregation changes the factors; results differ between the two choices.
- The two-pass batch-aware approach scales batches by median TIC of conserved features; it does not correct batch-specific feature-level artifacts beyond global intensity scaling.
- Sample names must match mzML file names — an earlier pcpfm issue regarding mismatched sample names was only fixed as of 2/28/24 per the README.

## Evidence

- [intro] normalization among pcpfm data-quality operations: "Operations to correct common data quality issues are provided including normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation."
- [intro] normalization within the pipeline's post-processing suite: "including normalization, feature interpolation, removal of rare features, quality assurance, quality control evaluations, and generates PDF reports"
- [readme] pcpfm outputs include normalized and batch-corrected feature tables: "This includes feature tables that are optionally blank masked, normalized, batch corrected, annotated or otherwise curated by PCPFM"
- [readme] pipeline overview lists normalization and batch correction: "- perform quality control
- data normalization and batch correction"
- [readme] default feature-table location for the normalized output: "      asari_results/
          preferred_Feature_table.tsv"
