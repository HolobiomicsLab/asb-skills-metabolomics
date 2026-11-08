---
name: metabolomics-quality-correction-workflow
description: Use when an Asari preferred_Feature_table.tsv from a pcpfm experiment
  shows per-sample TIC differences driven by feature counts, or samples span multiple
  batches, and the table must be normalized/batch-corrected before downstream analysis.
when_to_use_negative:
- Input is raw .mzML or .raw data — preprocessing via Asari must run first to produce
  the feature table.
- The table has already been TIC-normalized; re-normalization would double-scale the
  intensities.
- Only a small fraction of features are shared across samples, so the TIC_normalization_percentile
  filter would leave too few conserved features to compute robust factors.
tools:
- name: pcpfm
  role: Pipeline that provides the normalize command and the correction operations
    (normalization, blank masking, batch correction, removal of uncommon features,
    missing value imputation) over experiment feature tables
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: Asari
  role: Upstream preprocessor whose preferred feature table is the input to normalization
    and QC operations
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: pandas
  role: Loads the feature table TSV and computes per-sample TIC sums and median/mean
    normalization factors over conserved features
- name: numpy
  role: Array arithmetic for computing Z-scores (outlier cutoff |Z| > 2.5) and per-sample
    TIC scaling of the intensity matrix
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:fc14d36e658e238b24d2745e0ea73dc83db3e7a630b9cbfda4b9040054663125
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# metabolomics-quality-correction-workflow

## Summary

Applies pcpfm's data-quality correction operations — TIC-based normalization (one-pass or batch-aware two-pass), blank masking, batch correction, removal of uncommon/rare features, and missing value imputation — to Asari feature tables so they are fit for downstream statistical analysis.

## When to use

Use when an Asari preferred_Feature_table.tsv from a pcpfm experiment shows per-sample TIC differences driven by feature counts, or samples span multiple batches, and the table must be normalized/batch-corrected before downstream analysis.

## When NOT to use

- Input is raw .mzML or .raw data — preprocessing via Asari must run first to produce the feature table.
- The table has already been TIC-normalized; re-normalization would double-scale the intensities.
- Only a small fraction of features are shared across samples, so the TIC_normalization_percentile filter would leave too few conserved features to compute robust factors.

## Inputs

- Asari preferred feature table TSV (asari_results/preferred_Feature_table.tsv)
- pcpfm experiment directory with metadata CSV/sequence file (including a 'Batch' field for multi-batch studies)
- Full feature table TSV (asari_results/export/full_feature_table.tsv)

## Outputs

- Normalized feature table TSV (e.g. feature_tables/preferred_normalized.tsv)
- QA/QC figures (pca.png, tsne.png) per feature table in QAQC_figs/
- PDF QA/QC reports from the pipeline

## How to apply

Load the Asari feature table TSV (e.g. asari_results/preferred_Feature_table.tsv) with pandas and restrict factor calculations to features present in at least the TIC_normalization_percentile (0.90 in the paper's example) of samples, so only common features contribute. Compute each sample's total ion current over the retained features and derive its normalization factor using the median (default) or mean, then divide that sample's feature intensities by its factor to yield the one-pass corrected table. If samples were acquired across batches (--by_batch), first normalize each batch independently, then scale batches to one another using their median TIC values computed from conserved features (two-pass approach). Before normalizing, apply the pipeline's other correction operations as needed: blank masking, removal of uncommon/rare features, and outlier removal of failed injections (default |number-of-features Z-score| > 2.5). Rationale: restricting factors to common features prevents low-signal or blank-prone features from distorting per-sample scaling, and the two-pass median-TIC scheme removes between-batch intensity offsets. Write the result to a new moniker TSV (e.g. preferred_normalized.tsv) in the experiment's feature_tables directory.

## Related tools

- **pcpfm** (Pipeline that provides the normalize command and the correction operations (normalization, blank masking, batch correction, removal of uncommon features, missing value imputation) over experiment feature tables) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **Asari** (Upstream preprocessor whose preferred feature table is the input to normalization and QC operations) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **pandas** (Loads the feature table TSV and computes per-sample TIC sums and median/mean normalization factors over conserved features)
- **numpy** (Array arithmetic for computing Z-scores (outlier cutoff |Z| > 2.5) and per-sample TIC scaling of the intensity matrix)

## Examples

```
import pandas as pd; table = pd.read_csv('asari_results/preferred_Feature_table.tsv', sep='\t')
```

## Evaluation signals

- Output TSV (preferred_normalized.tsv) exists in feature_tables/ and has the same samples and features as the input table
- Post-normalization per-sample TIC values are consistent across samples rather than driven by per-sample feature counts
- Normalization factors were derived only from features present in at least TIC_normalization_percentile (e.g. 0.90) of samples
- In --by_batch mode, batches have consistent median TIC over conserved features after the two-pass scale step
- Samples with absolute number-of-features Z-score > 2.5 were dropped before normalization

## Limitations

- Features not present in at least the percentile threshold of samples are excluded from factor calculation and may require separate missing value imputation or removal of uncommon features
- The median (default) vs mean choice for the factor changes scaling; the article's example uses the median
- Batch-aware correction depends on conserved features being measurable in every batch to compute between-batch median-TIC scale factors
- Support for .raw input files is limited in pcpfm; mzML conversion is the recommended path
- Low signal-to-noise features that fail Asari's quality threshold remain missing and are not recovered by normalization

## Evidence

- [intro] Operations to correct common data quality issues are provided including normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation.: "Operations to correct common data quality issues are provided including normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation."
- [intro] by default, samples that have an absolute value for their number of features Z-score greater than 2.5 are dropped.: "by default, samples that have an absolute value for their number of features Z-score greater than 2.5 are dropped."
- [intro] including normalization, feature interpolation, removal of rare features, quality assurance, quality control evaluations, and generates PDF reports: "including normalization, feature interpolation, removal of rare features, quality assurance, quality control evaluations, and generates PDF reports"
- [readme] This includes feature tables that are optionally blank masked, normalized, batch corrected, annotated or otherwise curated by PCPFM: "This includes feature tables that are optionally blank masked, normalized, batch corrected, annotated or otherwise curated by PCPFM and empirical compounds as a JSON file"
