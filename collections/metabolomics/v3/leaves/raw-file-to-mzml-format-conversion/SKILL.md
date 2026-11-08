---
name: raw-file-to-mzml-format-conversion
description: Use when input acquisitions are proprietary Thermo .raw files and downstream
  tools (e.g. Asari) require .mzML; skip entirely when .mzML files are already provided.
when_to_use_negative:
- The input acquisitions are already in .mzML format — the pipeline skips this step
  entirely when .mzML files are provided as input.
- The raw files are not Thermo .raw (e.g. other vendors' proprietary formats) — ThermoRawFileParser
  targets Thermo acquisitions, and the README notes non-mzML support ('You can work
  with .raw files but support is limited').
- You already have a feature table — conversion operates on raw instrument data, not
  processed tables.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: ThermoRawFileParser
  role: Converts Thermo .raw files to centroided .mzML; invoked via a configurable
    command with -f=1 (centroid) and $RAW_PATH/$OUT_PATH placeholders
- name: mono
  role: Runtime used to execute the ThermoRawFileParser.exe command
- name: pcpfm
  role: Pipeline that orchestrates the conversion step, writes results to converted_acquisitions,
    and validates file existence against the metadata CSV
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: Python
  role: Language in which the pipeline implements the conversion orchestration and
    file_exists validation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:86b551f6adf37df8ae1405d6221d0f243b57669cfd7235fe8a77dc84d66f4b59
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
  tools:
  - Python
  - ThermoRawFileParser
  - mono
  - pcpfm
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# raw-file-to-mzml-format-conversion

## Summary

Converts proprietary Thermo .raw LC-MS acquisitions into centroided .mzML files using ThermoRawFileParser, so that downstream preprocessing (e.g. Asari in the pcpfm pipeline) operates on an open format.

## When to use

Use when input acquisitions are proprietary Thermo .raw files and downstream tools (e.g. Asari) require .mzML; skip entirely when .mzML files are already provided.

## When NOT to use

- The input acquisitions are already in .mzML format — the pipeline skips this step entirely when .mzML files are provided as input.
- The raw files are not Thermo .raw (e.g. other vendors' proprietary formats) — ThermoRawFileParser targets Thermo acquisitions, and the README notes non-mzML support ('You can work with .raw files but support is limited').
- You already have a feature table — conversion operates on raw instrument data, not processed tables.

## Inputs

- Thermo .raw acquisition files (.raw)
- Experiment metadata CSV mapping sample names to .raw file paths (e.g. Name and Filepath columns)

## Outputs

- One centroided .mzML file per .raw acquisition, written to the experiment's converted_acquisitions subdirectory (sample1.mzML, sample2.mzML, ...)

## How to apply

Build the conversion command from a user-configurable template containing $RAW_PATH and $OUT_PATH placeholders, defaulting to 'mono ThermoRawFileParser.exe -f=1 -i $RAW_PATH -b $OUT_PATH', where -f=1 selects centroiding. Load the experiment metadata CSV that maps each sample name to its .raw acquisition filepath, and for each .raw file substitute the input and output paths, writing the resulting centroided .mzML into the experiment's converted_acquisitions subdirectory (e.g. sample1.mzML, sample2.mzML). If .mzML files are already supplied as input, skip this step entirely. Validate that one centroided .mzML exists in converted_acquisitions for every .raw listed in the metadata CSV (file_exists check); note the pipeline had a fixed issue where sample names not matching mzML file names caused errors.

## Related tools

- **ThermoRawFileParser** (Converts Thermo .raw files to centroided .mzML; invoked via a configurable command with -f=1 (centroid) and $RAW_PATH/$OUT_PATH placeholders)
- **mono** (Runtime used to execute the ThermoRawFileParser.exe command)
- **pcpfm** (Pipeline that orchestrates the conversion step, writes results to converted_acquisitions, and validates file existence against the metadata CSV) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **Python** (Language in which the pipeline implements the conversion orchestration and file_exists validation)

## Examples

```
mono ThermoRawFileParser.exe -f=1 -i $RAW_PATH -b $OUT_PATH
```

## Evaluation signals

- One centroided .mzML file exists in converted_acquisitions for every .raw acquisition listed in the metadata CSV (file_exists check).
- Each output .mzML filename corresponds to its source sample name (sample names must match mzML file names; a mismatch bug was fixed 2/28/24).
- The -f=1 flag was applied, so spectra in the .mzML are centroided rather than profile.
- When input was already .mzML, the conversion step was skipped and converted_acquisitions handling was bypassed.

## Limitations

- Support for .raw (non-mzML) inputs is limited: 'You can work with .raw files but support is limited' (README).
- Sample names that do not match their mzML file names caused an issue, fixed as of 2/28/24 — verify filename/sample-name correspondence.
- Conversion is Thermo-specific; non-Thermo proprietary formats are not addressed in the article or README.
- Running ThermoRawFileParser.exe requires mono.

## Evidence

- [other] The pipeline optionally converts raw Thermo files to centroided .mzML using ThermoRawFileParser, driven by a user-supplied conversion command with $RAW_PATH and $OUT_PATH placeholders: [evidence span withheld — anchor status: not_found]
- [other] default command: "defaulting to 'mono ThermoRawFileParser.exe -f=1 -i $RAW_PATH -b $OUT_PATH'"
- [other] validation rule: [evidence span withheld — anchor status: not_found]
- [readme] README workflow capability: "convert Thermo .raw to mzML (ThermoRawFileParser)"
- [readme] README input/output spec: "Inputs should include a set of raw files (.raw or .mzML) and a csv file for metadata (minimal sample names and file path)."
- [readme] README limited raw support: "You can work with .raw files but support is limited."
- [readme] README filename matching fix: "there was an issue regarding sample names that do not match their mzML file names. This has been fixed as of 2/28/24."
- [abstract] article abstract input formats: "Our pipeline takes experimental data in .mzML or .raw format and outputs annotated feature tables for subsequent biological interpretation."
