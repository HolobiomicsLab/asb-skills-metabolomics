---
name: im-ms-acquisition-metadata-extraction
description: Use when starting from an original raw LC-IM-MS instrument file (e.g.,
  Agilent .d from a MassIVE deposit) and you need the acquisition settings — electric
  field strength, pressure, temperature, voltages — before or alongside downstream
  processing.
when_to_use_negative:
- The goal is signal processing (smoothing, saturation repair, demultiplexing, LC
  compression/IM interpolation) rather than extracting acquisition parameters — use
  the PreProcessor's preprocessing modules instead.
- The data is already in an open, processed format without the vendor instrument file,
  so the acquisition settings cannot be read from the source.
- The instrument data is not in a format the PreProcessor ingests (e.g., non-Agilent,
  non-UIMF vendor formats); the article notes conversion to UIMF as an intermediate
  step may be needed for other IM-MS instrumentation.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: PNNL PreProcessor
  role: 'Standalone software whose metadata export module reads the instrument-format
    IM-MS data and writes the acquisition-parameter text file (version used: v3.0.2020.11.24).'
  repo: https://omics.pnl.gov/software/pnnl-preprocessor
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00425
    title: A Preprocessing Tool for Enhanced Ion Mobility–Mass Spectrometry-Based
      Omics Workflows
  source_content_hash: sha256:0d1b507210fb4b8904d9ed1d18e397eec8f9c15fc608def36e6a627ed1b653b0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PNNL PreProcessor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# im-ms-acquisition-metadata-extraction

## Summary

Exports the IM-MS acquisition settings and parameters (electric field strength, pressure, temperature, voltages) embedded in raw instrument files using the PNNL PreProcessor's metadata export module, producing a metadata text file that lets users examine instrument performance relative to the collected data.

## When to use

Use when starting from an original raw LC-IM-MS instrument file (e.g., Agilent .d from a MassIVE deposit) and you need the acquisition settings — electric field strength, pressure, temperature, voltages — before or alongside downstream processing.

## When NOT to use

- The goal is signal processing (smoothing, saturation repair, demultiplexing, LC compression/IM interpolation) rather than extracting acquisition parameters — use the PreProcessor's preprocessing modules instead.
- The data is already in an open, processed format without the vendor instrument file, so the acquisition settings cannot be read from the source.
- The instrument data is not in a format the PreProcessor ingests (e.g., non-Agilent, non-UIMF vendor formats); the article notes conversion to UIMF as an intermediate step may be needed for other IM-MS instrumentation.

## Inputs

- Original raw LC-IM-MS MS-file in instrument/vendor format, e.g., an Agilent .d directory from a lipid-extract run
- Raw TOF ion-mobility data readable by the PNNL PreProcessor (the tool's native instrument-format ingestion)

## Outputs

- Metadata text file (acquisition_metadata.txt) containing the settings and parameters used for IM-MS data acquisition: electric field strength, pressure, temperature, and voltages

## How to apply

Retrieve one original raw MS-file (e.g., an Agilent .d lipid-extract file from ftp://massive.ucsd.edu/MSV000087295) and run the metadata export module of the PNNL PreProcessor (v3.0.2020.11.24) on it. The module reads the instrument-format data — the same raw LC-IM-MS TOF data that the PreProcessor's smoothing, saturation-repair, and demultiplexing algorithms operate on — and writes a metadata text file rather than a processed MS-file. Verify the exported acquisition_metadata.txt exists and contains the key acquisition parameter fields reported for IM-MS acquisitions: electric field strength, pressure, temperature, and voltages. The rationale is that these settings govern instrument performance, so having them alongside the data enables thorough examination of how acquisition conditions relate to signal quality (e.g., low-abundance peak definition and detector saturation issues the PreProcessor targets). The PreProcessor User Guide, distributed with the software executable, documents the tool's usage.

## Related tools

- **PNNL PreProcessor** (Standalone software whose metadata export module reads the instrument-format IM-MS data and writes the acquisition-parameter text file (version used: v3.0.2020.11.24).) — https://omics.pnl.gov/software/pnnl-preprocessor

## Evaluation signals

- The exported acquisition_metadata.txt file exists after running the module on the raw file.
- The exported file contains the key IM-MS acquisition fields: electric field strength, pressure, temperature, and voltages.
- The values in the metadata file are consistent with the acquisition described for the source dataset (e.g., the LC-DT IM-MS runs deposited at MSV000087295).
- The metadata corresponds to the same file used for downstream preprocessing (a single original raw .d file was retrieved per the workflow).
- The PreProcessor version used is recorded (v3.0.2020.11.24) so the export behavior is reproducible.

## Limitations

- The export reflects only the acquisition settings readable from the instrument-format data the PreProcessor ingests; for other IM-MS instrumentation the article notes vendors or conversion to UIMF as an intermediate step would be required.
- The article does not specify a tool version in the main text, and no CLI invocation is documented — the module is used via the standalone PreProcessor executable and its User Guide.
- The output is a static metadata text file; it does not validate or interpret the parameters against data quality metrics.

## Evidence

- [full_text] metadata export writes acquisition settings: [evidence span withheld — anchor status: not_found]
- [full_text] acquisition parameter fields: [evidence span withheld — anchor status: not_found]
- [full_text] module run on raw file: [evidence span withheld — anchor status: not_found]
- [results] standalone software tool: "The PreProcessor is a standalone software tool that provides algorithms and utilities for MS data"
- [results] user guide availability: "The PreProcessor User Guide is freely available along with the software executable at https://omics.pnl.gov/software/pnnl-preprocessor."
