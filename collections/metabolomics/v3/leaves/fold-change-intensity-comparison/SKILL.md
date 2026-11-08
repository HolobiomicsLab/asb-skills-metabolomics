---
name: fold-change-intensity-comparison
description: Use when a feature table from LC-MS preprocessing (e.g. an Asari preferred_Feature_table.tsv)
  contains features likely arising from background ions or contaminants, and a sample_type
  annotation (e.g. 'Blank' vs 'Unknown') is available to separate blanks from study
  samples.
when_to_use_negative:
- The input data is raw .mzML or .raw — preprocessing into a feature table (e.g. with
  Asari) must happen first
- The goal is correcting batch effects or missing values rather than background contamination
  — pcpfm provides separate batch correction and missing value imputation operations
- No blank samples (or sample_type annotations distinguishing blanks) exist in the
  experiment metadata, so no blank-vs-sample intensity comparison is possible
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: pcpfm
  role: Provides the blank masking command that compares feature intensities between
    blanks and study samples and drops features below the blank_intensity_ratio, saving
    the result under a new moniker
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: Asari
  role: Produces the preferred feature table (TSV) that is the input to blank masking
- name: Python
  role: Language in which the pipeline and the pandas/numpy-based comparison of blank
    vs sample intensities are implemented
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:3d26e636b342247c3ea3aeba45816f0208548cd5794cc54307ebf9ebe017c44c
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
yes_rate: 0.8666666666666667
n_claims: 15
---

# fold-change-intensity-comparison

## Summary

A blank-masking filter that removes background/contaminant features by comparing feature intensities between blank and study samples, retaining only features whose sample intensity is at least a set ratio (default 3x) above the blank intensity. This is the blank masking operation of the pcpfm metabolomics pipeline.

## When to use

Use when a feature table from LC-MS preprocessing (e.g. an Asari preferred_Feature_table.tsv) contains features likely arising from background ions or contaminants, and a sample_type annotation (e.g. 'Blank' vs 'Unknown') is available to separate blanks from study samples.

## When NOT to use

- The input data is raw .mzML or .raw — preprocessing into a feature table (e.g. with Asari) must happen first
- The goal is correcting batch effects or missing values rather than background contamination — pcpfm provides separate batch correction and missing value imputation operations
- No blank samples (or sample_type annotations distinguishing blanks) exist in the experiment metadata, so no blank-vs-sample intensity comparison is possible

## Inputs

- Preferred Asari feature table (TSV) from the experiment's asari_results directory
- Experiment sample metadata CSV with a sample_type field (e.g. 'Blank', 'QC', 'Unknown') used via query_field
- blank_intensity_ratio parameter (pcpfm default: 3)

## Outputs

- Blank-masked feature table TSV saved under a new moniker ('preferred_blank_masked') in the experiment's feature_tables directory, containing the same feature columns as the input minus dropped features

## How to apply

Load the preferred Asari feature table (TSV) and the experiment sample metadata CSV with pandas, using the metadata to map each sample to its sample_type annotation (e.g. 'Blank' vs 'Unknown') via the user-specified query_field. Partition samples into blank and study-sample groups, then for each feature compare intensity in study samples against intensity in blanks with zeros excluded. Retain only features whose sample intensity is at least blank_intensity_ratio times the blank intensity — the pcpfm default is a 3-fold ratio; repeat with different blank types (process vs solvent blanks) if needed. Write the retained features to a new moniker ('preferred_blank_masked') saved as a TSV in the experiment's feature_tables directory.

## Related tools

- **pcpfm** (Provides the blank masking command that compares feature intensities between blanks and study samples and drops features below the blank_intensity_ratio, saving the result under a new moniker) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **Asari** (Produces the preferred feature table (TSV) that is the input to blank masking)
- **Python** (Language in which the pipeline and the pandas/numpy-based comparison of blank vs sample intensities are implemented)

## Evaluation signals

- The output file preferred_blank_masked.tsv exists in the experiment's feature_tables directory
- The output is valid TSV with the same feature columns as the input table minus the dropped features
- Every retained feature satisfies the 3-fold sample-vs-blank intensity criterion (blank_intensity_ratio = 3) with zeros excluded
- Features present in blanks but not enriched in study samples are absent from the output table

## Limitations

- The default 3-fold ratio is a heuristic; features near the cutoff may be dropped or retained incorrectly, and the ratio may need adjustment per experiment
- Different blank types (process blanks vs solvent blanks) give different masking results and may need to be tried separately
- Zeros are excluded from the intensity comparison, so features absent from blanks are judged only on their presence in study samples

## Evidence

- [intro] blank masking operation rationale: "Operations to correct common data quality issues are provided including normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation."
- [readme] README on blank-masked feature tables as outputs: "This includes feature tables that are optionally blank masked, normalized, batch corrected, annotated or otherwise curated by PCPFM"
- [readme] README on sample type metadata fields used for grouping: "you should include a field for sample type (e.g., "Type") with strings for each type of sample (i.e., standards are marked 'STD', blanks are 'BLANK', etc.)"
- [other] task card finding on the comparison rule: [evidence span withheld — anchor status: not_found]
