---
name: instrument-parameter-qc-inspection
description: Use when you hold a raw LC-IM-MS file (e.g., Agilent .d) and need the
  instrument settings and parameters used for the acquisition — electric field strength,
  pressure, temperature, voltages — to check instrument performance relative to the
  collected data.
when_to_use_negative:
- Input is already a preprocessed or converted file (e.g., mzML or a UIMF-converted
  intermediate) rather than an original instrument-format acquisition — the module
  is intended to read the raw instrument-format data
- The goal is signal enhancement (smoothing, saturation repair, demultiplexing) rather
  than inspection of acquisition parameters — those are separate PreProcessor modules
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PNNL PreProcessor
  role: Standalone software whose metadata export module reads instrument-format IM-MS
    data and writes the acquisition settings/parameters text file (v3.0.2020.11.24)
  repo: https://omics.pnl.gov/software/pnnl-preprocessor
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00425
    title: A Preprocessing Tool for Enhanced Ion Mobility–Mass Spectrometry-Based
      Omics Workflows
  source_content_hash: sha256:d911a4dd2c06293a78daa22747386b81def8276e558c4d6a126094b26a76371e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PNNL PreProcessor
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# instrument-parameter-qc-inspection

## Summary

Exports IM-MS acquisition metadata from raw instrument-format files (e.g., Agilent .d) using the PNNL PreProcessor's metadata export utility, producing a text file with acquisition settings and parameters for examining instrument performance.

## When to use

Use when you hold a raw LC-IM-MS file (e.g., Agilent .d) and need the instrument settings and parameters used for the acquisition — electric field strength, pressure, temperature, voltages — to check instrument performance relative to the collected data.

## When NOT to use

- Input is already a preprocessed or converted file (e.g., mzML or a UIMF-converted intermediate) rather than an original instrument-format acquisition — the module is intended to read the raw instrument-format data
- The goal is signal enhancement (smoothing, saturation repair, demultiplexing) rather than inspection of acquisition parameters — those are separate PreProcessor modules

## Inputs

- Original raw lipid-extract MS-file in Agilent .d format (instrument-format acquisition data, e.g., from ftp://massive.ucsd.edu/MSV000087295)

## Outputs

- acquisition_metadata.txt — a metadata text file containing the IM-MS acquisition settings and parameters (electric field strength, pressure, temperature, voltages)

## How to apply

Run the metadata export module of the PNNL PreProcessor (v3.0.2020.11.24) on the original raw MS-file; the module reads the instrument-format data and writes a metadata text file with the acquisition settings and parameters. Verify that the exported acquisition_metadata.txt exists and contains the key IM-MS acquisition fields — electric field strength, pressure, temperature, and voltages. Because the tool operates on Agilent ".d" files (the same input Lipid Annotator reads), no prior conversion is needed for Agilent acquisitions; for other vendors, the article notes data can be brought into the tool by converting to UIMF as an intermediate step. The exported file enables thorough examination of instrument performance relative to the collected data, so compare exported values against expected instrument settings before downstream preprocessing.

## Related tools

- **PNNL PreProcessor** (Standalone software whose metadata export module reads instrument-format IM-MS data and writes the acquisition settings/parameters text file (v3.0.2020.11.24)) — https://omics.pnl.gov/software/pnnl-preprocessor

## Evaluation signals

- acquisition_metadata.txt exists after the run and is non-empty
- Exported file contains the key IM-MS acquisition fields: electric field strength, pressure, temperature, and voltages
- Exported parameter values are consistent with the instrument settings expected for the deposited acquisition (e.g., ftp://massive.ucsd.edu/MSV000087295 raw file)
- Export succeeds directly on the original Agilent .d file without requiring preprocessing steps such as LC compression or demultiplexing

## Limitations

- The PreProcessor is oriented at LC-IM-MS data (LC-DT IM-MS in the article); applicability to other IM-MS instrumentation is noted as something vendors could extend 'through converting to UIMF as an intermediate step'
- No explicit limitations are stated in the article's discussion text for the metadata export functionality itself

## Evidence

- [results] The PreProcessor's metadata export function writes metadata text files containing the settings and parameters used for IM-MS data acquisition — including electric field strength, pressure, temperature, and voltages: [evidence span withheld — anchor status: not_found]
