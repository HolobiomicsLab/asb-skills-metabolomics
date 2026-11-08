---
name: feature-table-metadata-mapping
description: Use when a feature table must be partitioned by sample class (Blank,
  QC, Unknown/STD) for background-removal or QC steps, and the experiment's metadata
  CSV provides the per-sample sample_type annotations.
when_to_use_negative:
- The input is raw .raw or .mzML data that has not yet been preprocessed by Asari
  into a feature table — run preprocessing first
- No sample metadata CSV or sample-type field exists, so samples cannot be annotated
  as Blank/QC/Unknown
- The analysis goal is annotation (khipu pre-annotation, MS1/MS2 matching) rather
  than sample-class-based data-quality correction
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Python 3
  role: Runtime for the pandas-based mapping of feature-table sample columns to metadata
    sample_type annotations
- name: pandas
  role: Load the feature table TSV and metadata CSV, perform the sample-name to sample_type
    join and partitioning
- name: pcpfm
  role: Pipeline whose blank-mask operation consumes the sample partition defined
    by the query_field mapping
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: Asari
  role: Preprocessor that produced the feature table being mapped to metadata
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:743d1aaf55dc3bbcf8dd15f405bb82b26f5ae794f6d540c344eaba95e78ac519
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# feature-table-metadata-mapping

## Summary

Join an Asari-derived feature table (TSV) to the experiment's sample metadata CSV so each sample can be mapped to its sample-type annotation (e.g. 'Blank' vs 'Unknown'). This mapping is the prerequisite for pcpfm operations such as blank masking.

## When to use

Use when a feature table must be partitioned by sample class (Blank, QC, Unknown/STD) for background-removal or QC steps, and the experiment's metadata CSV provides the per-sample sample_type annotations.

## When NOT to use

- The input is raw .raw or .mzML data that has not yet been preprocessed by Asari into a feature table — run preprocessing first
- No sample metadata CSV or sample-type field exists, so samples cannot be annotated as Blank/QC/Unknown
- The analysis goal is annotation (khipu pre-annotation, MS1/MS2 matching) rather than sample-class-based data-quality correction

## Inputs

- Preferred Asari feature table TSV (e.g. preferred_Feature_table.tsv or full_feature_table.tsv)
- Experiment sample metadata CSV (sequence file with sample name, file path, and a sample-type field such as 'Sample Type')

## Outputs

- Per-sample sample_type mapping (pandas Series/dict keyed by sample name)
- Partitioned column lists: blank samples and study samples (e.g. 'Blank' vs 'Unknown')
- Mapped table that downstream pcpfm steps (blank masking, normalization, batch correction) operate on

## How to apply

Load the preferred Asari feature table (TSV) and the experiment sample metadata CSV into pandas, using the README-specified metadata layout (sample name/path fields plus a sample-type field such as 'Sample Type' or 'Type'). Use the user-specified query_field (e.g. sample type) to map each sample name in the feature table columns to its annotation, then partition columns into blank and study-sample groups. This partition is consumed downstream by the pcpfm blank-mask operation, which retains only features whose intensity in Unknown samples is at least 3x (blank_intensity_ratio = 3) their intensity in blanks, with zeros excluded. Verify the mapping resolved every feature-table column to exactly one sample_type before any filtering logic runs.

## Related tools

- **Python 3** (Runtime for the pandas-based mapping of feature-table sample columns to metadata sample_type annotations)
- **pandas** (Load the feature table TSV and metadata CSV, perform the sample-name to sample_type join and partitioning)
- **pcpfm** (Pipeline whose blank-mask operation consumes the sample partition defined by the query_field mapping) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **Asari** (Preprocessor that produced the feature table being mapped to metadata)

## Examples

```
import pandas as pd; ft = pd.read_csv('preferred_Feature_table.tsv', sep='\t'); meta = pd.read_csv('Sequence.csv'); types = dict(zip(meta['Name'], meta['Sample Type']))
```

## Evaluation signals

- Every sample column in the feature table resolves to exactly one sample_type entry in the metadata CSV (no unmatched or ambiguous names)
- Blank group contains only samples annotated as blank types (e.g. process or solvent blanks) and study group only Unknown samples, per the query_field
- Downstream blank-masked table retains only features whose sample intensity is at least 3x blank intensity with zeros excluded
- Mapped output preserves the same feature rows as the input Asari table — mapping changes sample grouping, not the feature set

## Limitations

- The metadata join depends on sample names matching; the README notes an issue where sample names did not match their mzML file names, fixed in pcpfm as of 2/28/24
- Different blank types (process vs solvent) may need separate partition passes to determine which criterion applies
- The mapping is only as good as the metadata CSV — minimally it needs sample names and file path, and a type field with consistent strings ('STD', 'BLANK', etc.)

## Evidence

- [readme] As a basic recommmendation, you should include a field for sample type (e.g., "Type") with strings for each type of sample: "As a basic recommmendation, you should include a field for sample type (e.g., "Type") with strings for each type of sample (i.e., standards are marked 'STD', blanks are 'BLANK', etc.)"
- [readme] Inputs should include a set of raw files (.raw or .mzML) and a csv file for metadata (minimal sample names and file path).: "Inputs should include a set of raw files (.raw or .mzML) and a csv file for metadata (minimal sample names and file path)."
- [intro] Operations to correct common data quality issues are provided including normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation.: "Operations to correct common data quality issues are provided including normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation."
- [readme] This includes feature tables that are optionally blank masked, normalized, batch corrected, annotated or otherwise curated by PCPFM: "This includes feature tables that are optionally blank masked, normalized, batch corrected, annotated or otherwise curated by PCPFM"
