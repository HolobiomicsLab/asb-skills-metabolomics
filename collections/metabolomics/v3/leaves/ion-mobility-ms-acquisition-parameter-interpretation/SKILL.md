---
name: ion-mobility-ms-acquisition-parameter-interpretation
description: Use when a raw instrument-format IM-MS acquisition file (e.g. Agilent
  .d) needs its acquisition settings extracted for quality assessment of instrument
  performance relative to the collected data.
when_to_use_negative:
- Input is not an instrument-format raw acquisition file (e.g. it is already a preprocessed
  UIMF, a feature table, or Lipid Annotator output) — the metadata export module reads
  raw instrument data, not analysis results.
- The goal is signal enhancement (smoothing, saturation repair, demultiplexing) of
  LC-IM-MS data rather than interpreting acquisition parameters — use the PreProcessor's
  preprocessing modules instead.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PNNL PreProcessor
  role: Standalone software whose metadata export module reads the instrument-format
    data and writes a metadata text file with the acquisition settings and parameters.
  repo: https://omics.pnl.gov/software/pnnl-preprocessor
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00425
    title: A Preprocessing Tool for Enhanced Ion Mobility–Mass Spectrometry-Based
      Omics Workflows
  source_content_hash: sha256:9b3e63ae69a192fcb04a553ba0a05fecb08b0e52f10c53c35ac9c19b48d4d908
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PNNL PreProcessor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# ion-mobility-ms-acquisition-parameter-interpretation

## Summary

Use the PNNL PreProcessor's metadata export module to write a metadata text file capturing the settings and parameters used for IM-MS data acquisition (electric field strength, pressure, temperature, voltages), enabling thorough examination of instrument performance relative to the collected data.

## When to use

Use when a raw instrument-format IM-MS acquisition file (e.g. Agilent .d) needs its acquisition settings extracted for quality assessment of instrument performance relative to the collected data.

## When NOT to use

- Input is not an instrument-format raw acquisition file (e.g. it is already a preprocessed UIMF, a feature table, or Lipid Annotator output) — the metadata export module reads raw instrument data, not analysis results.
- The goal is signal enhancement (smoothing, saturation repair, demultiplexing) of LC-IM-MS data rather than interpreting acquisition parameters — use the PreProcessor's preprocessing modules instead.

## Inputs

- Original raw lipid-extract MS-file in Agilent .d instrument format (LC-IM-MS acquisition)

## Outputs

- acquisition_metadata.txt — metadata text file with instrument/acquisition parameter fields (electric field strength, pressure, temperature, voltages)

## How to apply

Retrieve the original raw MS-file (e.g. Agilent .d format, such as from ftp://massive.ucsd.edu/MSV000087295) and run the metadata export module of the PNNL PreProcessor (v3.0.2020.11.24) on it. The module reads the instrument-format data and writes a metadata text file recording the settings and parameters used for the IM-MS acquisition. Verify the export contains the key acquisition metadata fields reported for IM-MS acquisitions: electric field strength, pressure, temperature, and voltages. Rationale: these parameters let users examine instrument performance directly against the collected data, complementing the PreProcessor's preprocessing algorithms (e.g. demultiplexing with 75% pulses, intensity threshold of 50 counts).

## Related tools

- **PNNL PreProcessor** (Standalone software whose metadata export module reads the instrument-format data and writes a metadata text file with the acquisition settings and parameters.) — https://omics.pnl.gov/software/pnnl-preprocessor

## Evaluation signals

- The exported acquisition_metadata.txt exists after running the module on the raw .d file.
- The exported file contains the key IM-MS acquisition parameter fields: electric field strength, pressure, temperature, and voltages.
- The exported parameters correspond to the settings actually used for the acquisition being examined (instrument performance can be evaluated relative to the collected data).
- The module was run on the original instrument-format raw file (e.g. Agilent .d), not on a converted or preprocessed file.

## Limitations

- The article provides no explicit limitations for the metadata export module in the discussion text.
- Interpretability depends on the instrument writing acquisition parameters into the source file; data from other IM-MS instrumentation may need conversion to UIMF as an intermediate step.
- No tool version is specified in the discussion section for general use (version v3.0.2020.11.24 comes from the validated workflow).

## Evidence

- [full_text] The PreProcessor's metadata export function writes metadata text files containing the settings and parameters used for IM-MS data acquisition: [evidence span withheld — anchor status: not_found]
