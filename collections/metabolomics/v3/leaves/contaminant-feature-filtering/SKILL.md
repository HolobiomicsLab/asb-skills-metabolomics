---
name: contaminant-feature-filtering
description: Use when an Asari-derived feature table (TSV) contains features driven
  by background ions or contaminants and sample metadata distinguishes blank injections
  (e.g. 'Blank' vs 'Unknown' sample_type) so features can be filtered by sample-vs-blank
  intensity ratio.
when_to_use_negative:
- The input is already blank-masked or downstream of a blank-masked table — re-masking
  with the same blanks would double-filter features.
- No blank injections exist in the experiment metadata (no 'Blank' sample_type values),
  so no blank-vs-sample comparison is possible.
- The goal is removal of rare/low-prevalence features or outlier samples (|number-of-features
  Z-score| > 2.5), which are separate pcpfm data-quality operations rather than blank
  masking.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: pcpfm
  role: Provides the blank masking command that compares feature intensities between
    blanks and study samples and saves the result under a new moniker
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: Asari
  role: Produces the upstream preferred feature table (TSV) that blank masking operates
    on
- name: pandas
  role: Loads the feature table and sample metadata CSV and applies the 3x sample-vs-blank
    intensity comparison with zeros excluded
- name: Python 3
  role: Execution environment for the blank-masking implementation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:970ea4ed63ae241e290ec9c983c9fe884d0fb34c1ae3c6d45ad94f6fdbaed8a5
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

# contaminant-feature-filtering

## Summary

Blank masking of LC-MS feature tables in the pcpfm pipeline: features whose intensity in study samples is not at least 3x their intensity in blank samples are dropped as background/contaminant signals, producing a new 'preferred_blank_masked' feature table.

## When to use

Use when an Asari-derived feature table (TSV) contains features driven by background ions or contaminants and sample metadata distinguishes blank injections (e.g. 'Blank' vs 'Unknown' sample_type) so features can be filtered by sample-vs-blank intensity ratio.

## When NOT to use

- The input is already blank-masked or downstream of a blank-masked table — re-masking with the same blanks would double-filter features.
- No blank injections exist in the experiment metadata (no 'Blank' sample_type values), so no blank-vs-sample comparison is possible.
- The goal is removal of rare/low-prevalence features or outlier samples (|number-of-features Z-score| > 2.5), which are separate pcpfm data-quality operations rather than blank masking.

## Inputs

- Asari preferred feature table TSV (e.g. asari_results/preferred_Feature_table.tsv)
- Experiment sample metadata CSV with a sample-type field (e.g. Sample Type: Blank/QC/Unknown)
- query_field and query_field_values specifying which samples are blanks
- blank_intensity_ratio threshold (default 3)

## Outputs

- Blank-masked feature table saved as preferred_blank_masked.tsv under the experiment's feature_tables directory
- New feature-table moniker ('preferred_blank_masked') registered in the experiment for downstream QA/QC, normalization or annotation

## How to apply

Load the preferred Asari feature table (TSV) and the experiment sample metadata CSV with pandas, mapping each sample to its sample_type annotation (e.g. 'Blank' vs 'Unknown') via the user-specified query_field. Partition samples into blank and study-sample groups, then for each feature compare median/aggregate intensity in study samples against blanks with zeros excluded, retaining only features whose sample intensity is at least blank_intensity_ratio = 3 (the pcpfm default) times the blank intensity. Rationale: blank samples capture carryover, solvents and process contaminants, so any feature not appreciably enriched over blanks is unlikely to originate from the biological samples. Repeat with different blank types (process vs solvent blanks) if the blank composition differs. Write the retained features under a new moniker ('preferred_blank_masked') as a TSV in the experiment's feature_tables directory.

## Related tools

- **pcpfm** (Provides the blank masking command that compares feature intensities between blanks and study samples and saves the result under a new moniker) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **Asari** (Produces the upstream preferred feature table (TSV) that blank masking operates on)
- **pandas** (Loads the feature table and sample metadata CSV and applies the 3x sample-vs-blank intensity comparison with zeros excluded)
- **Python 3** (Execution environment for the blank-masking implementation)

## Examples

```
table = pd.read_csv('asari_results/preferred_Feature_table.tsv', sep='\t'); meta = pd.read_csv('Sequence.csv'); blanks = meta[meta['Sample Type']=='Blank']; unknowns = meta[meta['Sample Type']=='Unknown']
```

## Evaluation signals

- Output file preferred_blank_masked.tsv exists and is a valid TSV in the experiment's feature_tables directory.
- Output contains exactly the same feature columns as the input table minus the dropped features (no columns added, renamed or reordered).
- Every retained feature satisfies the 3-fold sample-vs-blank intensity criterion (blank_intensity_ratio = 3) with zero-intensity values excluded from the comparison.
- Applying the filter with different blank types (process vs solvent blanks) yields the expected differences in retained features; blanks-only features are all dropped.
- PCA/t-SNE QA/QC figures generated for the 'preferred_blank_masked' table render successfully from the new moniker.

## Limitations

- The ratio test depends on accurate sample_type annotation in the metadata; mislabeled blanks or unknowns will incorrectly drop or retain features.
- Zeros are excluded from the blank-intensity comparison, so features absent from blanks but genuinely near-noise in samples may survive masking.
- The default ratio of 3 is a fixed heuristic; different blank types (process vs solvent) may require re-running with adjusted thresholds.

## Evidence

- [intro] article: blank masking as a pcpfm operation: "Operations to correct common data quality issues are provided including normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation."
- [readme] README: blank-masked feature tables are PCPFM outputs: "This includes feature tables that are optionally blank masked, normalized, batch corrected, annotated or otherwise curated by PCPFM and empirical compounds as a JSON file"
- [full_text] task card: blank masking rule and default ratio: [evidence span withheld — anchor status: not_found]
- [full_text] task card: pandas-based workflow with sample_type partitioning: [evidence span withheld — anchor status: not_found]
