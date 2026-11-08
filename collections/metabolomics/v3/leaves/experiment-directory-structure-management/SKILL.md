---
name: experiment-directory-structure-management
description: Use when setting up or curating the output layout for an LC-MS metabolomics
  processing run with pcpfm, where .raw/.mzML inputs, Asari feature tables, empirical
  compound JSON, and QA/QC figures must be organized under one Experiment Directory.
when_to_use_negative:
- Inputs are already downstream-analyzable feature tables or empirical compound JSON
  from another pipeline with their own layout — do not force them into the PCPFM tree
  without mapping their provenance.
- A single ad-hoc analysis of one .mzML file with asari alone — the full experiment
  tree (QAQC_figs, annotations, results) is unnecessary overhead when no pcpfm pipeline
  run is planned.
- Studies whose instrument metadata cannot be expressed in the sequence CSV (sample
  name + filepath + Type/Batch fields); the README notes manually crafted sequence
  files are used instead of the preprocess step in the manuscript examples.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: pcpfm
  role: Pipeline that creates and manages the standardized Experiment Directory layout
    (converted_acquisitions, asari_results, annotations, QAQC_figs, experiment.json)
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: Python
  role: Implementation language for the pipeline that writes and validates the directory
    structure
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: ThermoRawFileParser
  role: Produces the centroided .mzML files placed in the converted_acquisitions subdirectory
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:2e7f2187903f1255bf1d9d803f8d72ea3408db4a8e3431cd3279c1d9bdc23bb4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
  tools:
  - Python
  - ThermoRawFileParser
  - mono
  - pcpfm
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# Experiment directory structure management

## Summary

Organizes a PCPFM metabolomics experiment into a standardized directory tree (raw_acquisitions, converted_acquisitions, asari_results, feature_tables, annotations, results, QAQC_figs, experiment.json) so that raw files, intermediate features, and annotated outputs remain traceable and immediately reusable downstream.

## When to use

Use when setting up or curating the output layout for an LC-MS metabolomics processing run with pcpfm, where .raw/.mzML inputs, Asari feature tables, empirical compound JSON, and QA/QC figures must be organized under one Experiment Directory.

## When NOT to use

- Inputs are already downstream-analyzable feature tables or empirical compound JSON from another pipeline with their own layout — do not force them into the PCPFM tree without mapping their provenance.
- A single ad-hoc analysis of one .mzML file with asari alone — the full experiment tree (QAQC_figs, annotations, results) is unnecessary overhead when no pcpfm pipeline run is planned.
- Studies whose instrument metadata cannot be expressed in the sequence CSV (sample name + filepath + Type/Batch fields); the README notes manually crafted sequence files are used instead of the preprocess step in the manuscript examples.

## Inputs

- Experiment metadata CSV mapping sample names to .raw or .mzML file paths (with optional Type and Batch fields)
- Thermo .raw acquisition files and/or centroided .mzML files
- User-created feature table .tsv files
- pcpfm experiment.json metadata file

## Outputs

- Experiment Directory with raw_acquisitions/ (sample1.raw, ...)
- converted_acquisitions/ (sample1.mzML, sample2.mzML, ...)
- asari_results/preferred_Feature_table.tsv and asari_results/export/full_feature_table.tsv
- feature_tables/user_created_table_1.tsv, ...
- annotations/empCpd.json
- results/annotation_table, results/feature_table, results/sample_annotation_table
- QAQC_figs/<table_name>/pca.png, tsne.png, ...
- experiment.json

## How to apply

Create an Experiment Directory per study and populate subdirectories per the PCPFM convention: raw_acquisitions/ holds sample1.raw ... sampleN.raw; converted_acquisitions/ holds the centroided .mzML files (sample1.mzML, ...) produced by ThermoRawFileParser; asari_results/ holds preferred_Feature_table.tsv and export/full_feature_table.tsv; feature_tables/ stores user-created curated tables; annotations/ holds empCpd.json; results/ holds annotation_table, feature_table and sample_annotation_table; QAQC_figs/ holds per-feature-table plots such as pca.png and tsne.png; experiment.json records the experiment metadata. The experiment metadata CSV (sample names mapped to file paths, with recommended 'Type' fields like BLANK/QC/STD and a 'Batch' field for multi-batch studies) drives which files land in raw_acquisitions and converted_acquisitions. Validation follows the pipeline's file_exists checks — e.g. one .mzML in converted_acquisitions per .raw listed in the metadata CSV, and a preferred_Feature_table.tsv in asari_results after processing.

## Related tools

- **pcpfm** (Pipeline that creates and manages the standardized Experiment Directory layout (converted_acquisitions, asari_results, annotations, QAQC_figs, experiment.json)) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **Python** (Implementation language for the pipeline that writes and validates the directory structure) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **ThermoRawFileParser** (Produces the centroided .mzML files placed in the converted_acquisitions subdirectory)

## Examples

```
pcpfm preprocess -s ./Sequence.csv --new_csv_path ./NewSequence.csv --name_field='Name' --path_field='Path' --preprocessing_config ./pcpfm/prerpocessing.json
```

## Evaluation signals

- One centroided .mzML file exists in converted_acquisitions for every .raw acquisition listed in the metadata CSV (file_exists check)
- raw_acquisitions/ contains exactly the .raw files named by the metadata CSV (e.g. sample1.raw, sample2.raw)
- asari_results/preferred_Feature_table.tsv and asari_results/export/full_feature_table.tsv exist after Asari preprocessing
- annotations/ contains empCpd.json and results/ contains annotation_table, feature_table, and sample_annotation_table
- QAQC_figs/<user_created_table_name>/ contains the expected plot artifacts (pca.png, tsne.png) for each feature table

## Limitations

- Sample names that do not match their .mzML file names caused issues prior to the 2/28/24 fix — ensure the metadata CSV sample names match converted filenames.
- Working directly with .raw files has limited support; .mzML inputs skip the conversion step and no converted_acquisitions content is generated.
- The organization assumes a metadata CSV with sample name and file path at minimum; extra fields like Type and Batch are recommended but the minimal schema constrains what QA/QC and batch correction steps can use.

## Evidence

- [readme] The organization of the outputs is as such: Experiment Directory/ annotations/ empCpd.json ... asari_results/ preferred_Feature_table.tsv export/ full_feature_table.tsv converted_acquisitions sample1.mzML sample2.mzML: "The organization of the outputs is as such: Experiment Directory/ annotations/ empCpd.json ... asari_results/ preferred_Feature_table.tsv export/ full_feature_table.tsv converted_acquisitions"
- [readme] Inputs should include a set of raw files (.raw or .mzML) and a csv file for metadata (minimal sample names and file path).: "Inputs should include a set of raw files (.raw or .mzML) and a csv file for metadata (minimal sample names and file path)."
- [abstract] Our pipeline takes experimental data in .mzML or .raw format and outputs annotated feature tables for subsequent biological interpretation.: "Our pipeline takes experimental data in .mzML or .raw format and outputs annotated feature tables for subsequent biological interpretation."
- [readme] you should include a field for sample type (e.g., "Type") with strings for each type of sample (i.e., standards are marked 'STD', blanks are 'BLANK', etc.) and a "Batch" field if your samples were collected in multiple: "you should include a field for sample type (e.g., "Type") with strings for each type of sample (i.e., standards are marked 'STD', blanks are 'BLANK', etc.) and a "Batch" field if your samples were"
