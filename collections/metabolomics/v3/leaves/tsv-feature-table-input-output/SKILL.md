---
name: tsv-feature-table-input-output
description: Use when a metabolomics workflow step (normalization, missing-value imputation,
  blank masking, batch correction, annotation) must read a feature table from or write
  one to a TSV file with samples as rows and features as columns.
when_to_use_negative:
- The data are still raw instrument files (.raw or .mzML) that have not yet been converted
  to mzML and processed by Asari into a feature table.
- The output must be consumed by a tool that does not parse tab-separated text (e.g.
  a JSON-only consumer such as empCpd.json); use the JSON serialization of empirical
  compounds instead.
- The table already contains annotation columns or non-numeric metadata that would
  break a strict numeric feature-intensity layout.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: pcpfm
  role: Produces, curates, and writes the TSV feature tables (e.g. preferred_drop_missing,
    preferred_interpolated.tsv) inside the experiment directory.
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: Asari
  role: Generates the initial TSV feature tables (preferred_Feature_table.tsv, export/full_feature_table.tsv)
    that downstream steps read.
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: Python (pandas)
  role: Loads and writes the TSV feature tables when applying imputation or rare-feature
    removal filters.
- name: MetDataModel
  role: Defines the Feature and Empirical compound concepts that correspond to the
    columns of the TSV table.
  repo: https://github.com/shuzhao-li-lab/metDataModel
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:da019364d905bec6f9cc19381d2f369b71422b9ac08c190268c5f1b9f2a70873
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
  tools:
  - Asari
  - MetDataModel
  - Python
  - Python (pandas)
  - pcpfm
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# tsv-feature-table-input-output

## Summary

This skill covers the PCPFM convention of persisting curated feature tables as tab-separated value (TSV) files—each row is a sample and each column is a feature (empirical compound or Asari feature)—so that downstream statistical tools such as MetaboAnalyst or R/Python workflows can consume them directly.

## When to use

Use when a metabolomics workflow step (normalization, missing-value imputation, blank masking, batch correction, annotation) must read a feature table from or write one to a TSV file with samples as rows and features as columns.

## When NOT to use

- The data are still raw instrument files (.raw or .mzML) that have not yet been converted to mzML and processed by Asari into a feature table.
- The output must be consumed by a tool that does not parse tab-separated text (e.g. a JSON-only consumer such as empCpd.json); use the JSON serialization of empirical compounds instead.
- The table already contains annotation columns or non-numeric metadata that would break a strict numeric feature-intensity layout.

## Inputs

- Feature table TSV (samples as rows, features as columns), e.g. preferred_Feature_table.tsv or full_feature_table.tsv exported by Asari
- Sample metadata CSV with at least sample names, file path, and a 'Type' field (BLANK/STD/Unknown) and optional 'Batch' field
- experiment.json describing the feature_tables directory

## Outputs

- TSV feature tables written to the experiment's feature_tables directory (e.g. preferred_drop_missing, preferred_interpolated.tsv)
- Sample-annotation table and feature table under the results directory in .txt/TSV format ready for MetaboAnalyst or R/Python analysis

## How to apply

Represent the feature table as a TSV in which each row corresponds to a sample (including blanks and QCs identified by a 'Type' field such as 'BLANK' or 'STD') and each column corresponds to a feature from the Asari or khipu preprocessing step. Load the table with pandas before applying filters such as removal of uncommon features or missing-value imputation, then write the result back as a TSV—e.g. the workflow that drops infrequent features produces a 'preferred_drop_missing' table, and the imputation step that fills missing entries with 0.5 × the per-feature minimum writes 'preferred_interpolated.tsv', preserving the original sample rows and feature columns. Place user-created tables in the experiment's 'feature_tables' directory (e.g. user_created_table_1.tsv) and reference them in the experiment.json so QA/QC figures (PCA, t-SNE) and annotation tables can be generated per table. The rationale is that TSV output is intended to be 'immediately usable for downstream analysis (e.g. MetaboAnalyst or common tools in R, Python etc.)', so file format and layout must be preserved across every intermediate step.

## Related tools

- **pcpfm** (Produces, curates, and writes the TSV feature tables (e.g. preferred_drop_missing, preferred_interpolated.tsv) inside the experiment directory.) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **Asari** (Generates the initial TSV feature tables (preferred_Feature_table.tsv, export/full_feature_table.tsv) that downstream steps read.) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **Python (pandas)** (Loads and writes the TSV feature tables when applying imputation or rare-feature removal filters.)
- **MetDataModel** (Defines the Feature and Empirical compound concepts that correspond to the columns of the TSV table.) — https://github.com/shuzhao-li-lab/metDataModel

## Examples

```
pcpfm preprocess -s ./Sequence.csv --new_csv_path ./NewSequence.csv --name_field='Name' --path_field='Path' --preprocessing_config ./pcpfm/prerpocessing.json
```

## Evaluation signals

- The output file exists as a valid TSV with the same sample rows and feature columns as its input table.
- After imputation the table contains no remaining missing (NaN or zero) entries, and every imputed entry equals 0.5 × that feature's minimum observed non-zero intensity.
- After rare-feature removal, each remaining feature appears in at least the user-specified fraction of samples (e.g. the preferred_drop_missing table).
- The file is placed in the experiment's feature_tables directory and is listed in experiment.json so QA/QC figures (PCA, t-SNE) can be generated per table.
- Row count matches the number of samples in the metadata CSV (including BLANK and STD types if they were retained).

## Limitations

- Sample names that do not match their mzML file names caused an issue fixed as of 2/28/24; ensure sample identifiers in the TSV match the metadata CSV.
- Manually crafted sequence/metadata CSVs are recommended over the automated preprocessing step for full flexibility; mis-formatted metadata will propagate into the TSV output.
- Annotation sources such as the HMDB are not redistributed—only the processed TSV tables are portable, not the annotation databases themselves.

## Evidence

- [readme] Missing value imputation replaces each missing entry with a scalar multiple of that feature's minimum observed intensity in the dataframe; the default imputation ratio is 0.5: [evidence span withheld — anchor status: not_found]
- [readme] Validation: confirm the output file exists as a TSV, contains no remaining missing values, and every imputed entry equals 0.5 × that feature's minimum observed non-zero intensity.: [evidence span withheld — anchor status: not_found]
- [readme] Outputs are intended to be immediately usable for downstream analysis (e.g. MetaboAnalyst or common tools in R, Python etc.). This includes feature tables that are optionally blank masked, normalized, batch corrected, annotated or otherwise curated by PCPFM: "Outputs are intended to be immediately usable for downstream analysis (e.g. MetaboAnalyst or common tools in R, Python etc.)."
- [readme] pcpfm preprocess -s ./Sequence.csv --new_csv_path ./NewSequence.csv --name_field='Name' --path_field='Path' --preprocessing_config ./pcpfm/prerpocessing.json: "pcpfm preprocess -s ./Sequence.csv --new_csv_path ./NewSequence.csv --name_field='Name' --path_field='Path' --preprocessing_config ./pcpfm/prerpocessing.json"
- [readme] you should include a field for sample type (e.g., "Type") with strings for each type of sample (i.e., standards are marked 'STD', blanks are 'BLANK', etc.): "you should include a field for sample type (e.g., "Type") with strings for each type of sample (i.e., standards are marked 'STD', blanks are 'BLANK', etc.)"
- [intro] including normalization, feature interpolation, removal of rare features, quality assurance, quality control evaluations, and generates PDF reports: "including normalization, feature interpolation, removal of rare features, quality assurance, quality control evaluations, and generates PDF reports"
- [intro] Operations to correct common data quality issues are provided including normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation.: "Operations to correct common data quality issues are provided including normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation."
- [readme] including a field for sample type (e.g., "Type") with strings for each type of sample (i.e., standards are marked 'STD', blanks are 'BLANK', etc.) and a "Batch" field if your samples were collected in multiple: "a "Batch" field if your samples were collected in multiple"
