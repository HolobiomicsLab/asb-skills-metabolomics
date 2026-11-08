---
name: metabolomics-blank-masking
description: Use when an Asari-derived feature table contains features plausibly attributable
  to blanks/contaminants and a sample metadata CSV annotates each sample's sample_type
  (e.g. 'Blank' vs 'Unknown').
when_to_use_negative:
- The input is already blank-masked or contains no blank injections in the metadata
  — there is no reference group to compare against.
- The goal is removing rare/uncommon features or missing value imputation; those are
  separate pcpfm data-quality operations, not blank masking.
- The metadata CSV lacks a usable sample_type/query_field, so blanks cannot be reliably
  partitioned from 'Unknown' samples.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: pcpfm
  role: Provides the blank masking operation that compares feature intensities between
    blanks and study samples using a query_field and drops features below the blank_intensity_ratio,
    saving the result under a new moniker
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: Asari
  role: Produces the feature table (e.g. preferred_Feature_table.tsv) that blank masking
    operates on
  repo: https://github.com/shuzhao-li-lab/asari_pcpfm_tutorials
- name: pandas
  role: Used to load the feature table TSV and sample metadata CSV and to partition
    samples by sample_type for the ratio comparison
- name: numpy
  role: Supports the per-feature intensity comparison between study samples and blanks
    with zeros excluded
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:be59386a223796a65087fcd26e0eb5595f3babd09b300487f352cd476e8ac615
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
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# Metabolomics blank masking

## Summary

Removes features likely arising from background ions and contaminants by comparing feature intensities between blank samples and study ('Unknown') samples, retaining only features whose sample intensity exceeds the blank intensity by a set ratio (pcpfm default blank_intensity_ratio = 3).

## When to use

Use when an Asari-derived feature table contains features plausibly attributable to blanks/contaminants and a sample metadata CSV annotates each sample's sample_type (e.g. 'Blank' vs 'Unknown').

## When NOT to use

- The input is already blank-masked or contains no blank injections in the metadata — there is no reference group to compare against.
- The goal is removing rare/uncommon features or missing value imputation; those are separate pcpfm data-quality operations, not blank masking.
- The metadata CSV lacks a usable sample_type/query_field, so blanks cannot be reliably partitioned from 'Unknown' samples.

## Inputs

- Preferred Asari feature table (TSV), e.g. asari_results/preferred_Feature_table.tsv
- Experiment sample metadata CSV with sample names and a sample_type field (e.g. 'Blank', 'Unknown')
- query_field specifying which metadata column defines blank vs study samples
- blank_intensity_ratio threshold (pcpfm default: 3)

## Outputs

- Blank-masked feature table saved under a new moniker (e.g. preferred_blank_masked.tsv) in the experiment's feature_tables directory
- Same feature columns as the input table minus dropped (contaminant/background) features

## How to apply

Load the preferred Asari feature table (TSV) and the experiment's sample metadata CSV with pandas, using the metadata to map each sample to its sample_type annotation. Partition samples into blank and study-sample groups according to the user-specified query_field. For each feature, compare intensity in study samples against intensity in blanks with zeros excluded, and drop any feature whose unknown-sample intensity is not at least 3x its blank intensity (blank_intensity_ratio = 3, the pcpfm default). Write the retained features to a new moniker (e.g. 'preferred_blank_masked') as a TSV in the experiment's feature_tables directory. If multiple blank types exist (e.g. process vs solvent blanks), repeat the comparison per blank type as needed.

## Related tools

- **pcpfm** (Provides the blank masking operation that compares feature intensities between blanks and study samples using a query_field and drops features below the blank_intensity_ratio, saving the result under a new moniker) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **Asari** (Produces the feature table (e.g. preferred_Feature_table.tsv) that blank masking operates on) — https://github.com/shuzhao-li-lab/asari_pcpfm_tutorials
- **pandas** (Used to load the feature table TSV and sample metadata CSV and to partition samples by sample_type for the ratio comparison)
- **numpy** (Supports the per-feature intensity comparison between study samples and blanks with zeros excluded)

## Examples

```
import pandas as pd; ft = pd.read_csv('asari_results/preferred_Feature_table.tsv', sep='\t'); meta = pd.read_csv('Sequence.csv'); blanks = meta[meta['Sample Type']=='Blank']
```

## Evaluation signals

- The output file (e.g. preferred_blank_masked.tsv) exists in the experiment's feature_tables directory and is valid TSV.
- The output retains the same feature columns as the input table minus only the dropped features (no new or renamed feature identifiers).
- Every retained feature satisfies the 3-fold sample-vs-blank intensity criterion (blank_intensity_ratio = 3) with zeros excluded from the comparison.
- Row-wise: any feature dropped from the input table should be verifiable as failing the sample-vs-blank ratio test under the specified query_field partition.
- Re-running with a different blank type (process vs solvent blank) should produce a distinguishable, explainable change in the retained feature set.

## Limitations

- The retention threshold is a fixed ratio (default 3x blank intensity); features near the cutoff may be retained or dropped depending on blank type chosen (process vs solvent blanks).
- The operation depends on correctly annotated sample types in the metadata CSV; mislabeled samples (e.g. a blank marked 'Unknown') will corrupt the comparison.
- PCPFM documentation notes sample names must match their mzML file names for correct sample mapping — a mismatch issue was fixed as of 2/28/24 but older runs may mis-associate samples with metadata.

## Evidence

- [intro] Operations to correct common data quality issues are provided including normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation.: "Operations to correct common data quality issues are provided including normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation."
- [readme] This includes feature tables that are optionally blank masked, normalized, batch corrected, annotated or otherwise curated by PCPFM and empirical compounds: "This includes feature tables that are optionally blank masked, normalized, batch corrected, annotated or otherwise curated by PCPFM and empirical compounds"
- [readme] you should include a field for sample type (e.g., "Type") with strings for each type of sample (i.e., standards are marked 'STD', blanks are 'BLANK', etc.): "you should include a field for sample type (e.g., "Type") with strings for each type of sample (i.e., standards are marked 'STD', blanks are 'BLANK', etc.)"
