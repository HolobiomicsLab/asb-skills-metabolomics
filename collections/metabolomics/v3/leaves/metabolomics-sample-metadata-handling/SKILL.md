---
name: metabolomics-sample-metadata-handling
description: Use when building or validating the sample metadata/sequence file for
  a pcpfm experiment — e.g. assigning Type (STD/BLANK/QC/Unknown) and Batch fields
  from an instrument sequence file, or preparing metadata that batch correction will
  consume via --by_batch.
when_to_use_negative:
- The metadata CSV already exists in the correct format with Name/Path (and Type/Batch
  where relevant) — go straight to pcpfm processing commands rather than re-preprocessing.
- The task is batch-effect correction itself rather than metadata preparation — that
  is performed by the pcpfm batch_correct command wrapping pyCombat, which merely
  consumes the Batch field produced here.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: pcpfm
  role: Pipeline whose `preprocess` command generates a standardized metadata CSV
    from a sequence file, and whose operations (e.g. batch_correct --by_batch) consume
    the metadata fields
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: Python
  role: Language of the pipeline and metadata preprocessing; pandas is used to load
    batch-annotated feature tables and confirm batch designation fields exist
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: pyComBat
  role: Downstream consumer of the user-specified batch metadata field for empirical-Bayes
    batch-effect correction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:18ccd50858330f00d06480c3b28901ecf5e2192d2f211b3c1a537e451f13f313
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
  tools:
  - Python
  - Python 3
  - pandas
  - pcpfm
  - pyComBat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# metabolomics-sample-metadata-handling

## Summary

Prepares and standardizes sample metadata (sequence/metadata CSV with Name, Path, sample Type, and Batch fields) for the pcpfm LC-MS metabolomics pipeline, so downstream operations such as blank masking, QC, and pyCombat-based batch correction (--by_batch) can key off user-specified metadata fields.

## When to use

Use when building or validating the sample metadata/sequence file for a pcpfm experiment — e.g. assigning Type (STD/BLANK/QC/Unknown) and Batch fields from an instrument sequence file, or preparing metadata that batch correction will consume via --by_batch.

## When NOT to use

- The metadata CSV already exists in the correct format with Name/Path (and Type/Batch where relevant) — go straight to pcpfm processing commands rather than re-preprocessing.
- The task is batch-effect correction itself rather than metadata preparation — that is performed by the pcpfm batch_correct command wrapping pyCombat, which merely consumes the Batch field produced here.

## Inputs

- Instrument sequence file (CSV, e.g. ./Sequence.csv) with sample names and file paths for .raw or .mzML acquisitions
- Preprocessing configuration JSON (e.g. ./pcpfm/prerpocessing.json) describing column rules
- Optionally, a manually crafted metadata CSV with minimal sample names and file paths plus Type and Batch columns

## Outputs

- A new metadata CSV (e.g. ./NewSequence.csv) with standardized Name, Path, sample Type (STD/BLANK/QC/Unknown) and Batch fields
- Experiment-level metadata consumed by pcpfm operations (e.g. --by_batch for batch correction, blank/QC designation for masking and QC)

## How to apply

Organize metadata into a CSV file, either by hand for small studies or by running the `pcpfm preprocess` command on an instrument-exported Sequence.csv to generate a new CSV using rules from a preprocessing JSON config, mapping the --name_field and --path_field columns. Include a field for sample type (e.g. 'Type') using the pipeline's conventions — standards 'STD', blanks 'BLANK', QCs and Unknowns — plus a 'Batch' field whenever samples were collected in multiple batches, since the batch-correction operation later determines batches from this user-specified metadata field. Ensure each sample name matches its .raw/.mzML file name, because a mismatch caused a known pipeline issue (fixed as of 2/28/24). The rationale is the paper's call to standardize terminologies covering sample preparation, experimental protocols, software processing steps and metadata within larger biomedical projects; for large studies use the preprocessing step, but manual crafting is still recommended for full flexibility.

## Related tools

- **pcpfm** (Pipeline whose `preprocess` command generates a standardized metadata CSV from a sequence file, and whose operations (e.g. batch_correct --by_batch) consume the metadata fields) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **Python** (Language of the pipeline and metadata preprocessing; pandas is used to load batch-annotated feature tables and confirm batch designation fields exist) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **pyComBat** (Downstream consumer of the user-specified batch metadata field for empirical-Bayes batch-effect correction)

## Examples

```
pcpfm preprocess -s ./Sequence.csv --new_csv_path ./NewSequence.csv --name_field='Name' --path_field='Path' --preprocessing_config ./pcpfm/prerpocessing.json
```

## Evaluation signals

- The output CSV (e.g. NewSequence.csv) exists and carries the designated --name_field and --path_field columns with resolvable .raw/.mzML file paths.
- Every sample row has a Type designation using the pipeline conventions ('STD' for standards, 'BLANK' for blanks, plus QC and Unknown designations).
- A 'Batch' field is present whenever samples were collected in multiple batches, so the downstream batch_correct --by_batch operation can group samples correctly.
- Each sample name matches its mzML file name — mismatches were a known pipeline bug fixed as of 2/28/24.
- Loading the batch-annotated table with pandas confirms a batch designation field exists before any metadata-dependent operation.

## Limitations

- Some instruments do not allow all values for all fields in a sequence file, which is why the preprocessing step exists; manual crafting is still recommended for full flexibility.
- Sample names that do not match their mzML file names caused a reported issue (fixed as of 2/28/24); verify name-to-file correspondence.
- Batch correction via the metadata Batch field is sensitive to missing features, which should be removed or imputed beforehand.

## Evidence

- [intro] there is an urgent need to standardize terminologies that cover sample preparation, experimental protocols, steps of software processing and metadata.: "there is an urgent need to standardize terminologies that cover sample preparation, experimental protocols, steps of software processing and metadata."
- [readme] you should include a field for sample type (e.g., "Type") with strings for each type of sample (i.e., standards are marked 'STD', blanks are 'BLANK', etc.) and a "Batch" field: "you should include a field for sample type (e.g., "Type") with strings for each type of sample (i.e., standards are marked 'STD', blanks are 'BLANK', etc.) and a "Batch" field"
- [readme] Inputs should include a set of raw files (.raw or .mzML) and a csv file for metadata (minimal sample names and file path).: "Inputs should include a set of raw files (.raw or .mzML) and a csv file for metadata (minimal sample names and file path)."
- [readme] Notably there was an issue regarding sample names that do not match their mzML file names. This has been fixed as of 2/28/24.: "Notably there was an issue regarding sample names that do not match their mzML file names. This has been fixed as of 2/28/24."
