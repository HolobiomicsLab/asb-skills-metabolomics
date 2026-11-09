---
name: agilent-masshunter-raw-file-handling
description: Reference for examining IM-MS instrument settings and acquisition performance
  when the input is an Agilent ".d" raw MS-file that must be opened or whose acquisition
  parameters need to be exported to a text file.
when_to_use_negative:
- Input is already a preprocessed or converted file (e.g. UIMF or mzML) rather than
  an original Agilent ".d" acquisition file — the metadata export reads instrument-format
  data directly
- The goal is to enhance signal quality (smoothing, saturation repair, demultiplexing)
  rather than to export acquisition metadata
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PNNL PreProcessor
  role: Standalone software whose metadata export module reads the Agilent ".d" instrument-format
    file and writes the acquisition metadata text file
  repo: https://omics.pnl.gov/software/pnnl-preprocessor
- name: Lipid Annotator
  role: Downstream tool that can directly process the same Agilent ".d" MS-files for
    feature finding and annotation of lipids
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00425
    title: A Preprocessing Tool for Enhanced Ion Mobility–Mass Spectrometry-Based
      Omics Workflows
  source_content_hash: sha256:104c42c9d7b68ea6fd7b309f44603c5e883502f31cc9fcb342fedce5bf3be799
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Lipid Annotator
  - PNNL PreProcessor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# agilent-masshunter-raw-file-handling

## Summary

Handles Agilent MassHunter ".d" raw acquisition files, including reading and exporting the embedded acquisition metadata (electric field strength, pressure, temperature, voltages) that documents IM-MS instrument performance for the collected data.

## When to use

Reference for examining IM-MS instrument settings and acquisition performance when the input is an Agilent ".d" raw MS-file that must be opened or whose acquisition parameters need to be exported to a text file.

## When NOT to use

- Input is already a preprocessed or converted file (e.g. UIMF or mzML) rather than an original Agilent ".d" acquisition file — the metadata export reads instrument-format data directly
- The goal is to enhance signal quality (smoothing, saturation repair, demultiplexing) rather than to export acquisition metadata

## Inputs

- Original raw lipid-extract MS-file in Agilent ".d" format (e.g. downloaded from ftp://massive.ucsd.edu/MSV000087295)

## Outputs

- Metadata text file (acquisition_metadata.txt) containing IM-MS acquisition settings and parameters: electric field strength, pressure, temperature, and voltages

## How to apply

Retrieve one original raw lipid-extract MS-file in Agilent ".d" format (e.g. from the deposit ftp://massive.ucsd.edu/MSV000087295). Run the metadata export function of the PNNL PreProcessor (v3.0.2020.11.24), which reads the instrument-format data directly and writes a metadata text file with the settings and parameters used for IM-MS data acquisition — electric field strength, pressure, temperature, and voltages. The exported acquisition_metadata.txt serves as the record for thorough examination of instrument performance relative to the collected data; verify it contains each of those key acquisition fields. The same ".d" input is also the format that downstream feature-finding tools like Lipid Annotator process directly, so raw-file handling and preprocessing live in one toolchain.

## Related tools

- **PNNL PreProcessor** (Standalone software whose metadata export module reads the Agilent ".d" instrument-format file and writes the acquisition metadata text file) — https://omics.pnl.gov/software/pnnl-preprocessor
- **Lipid Annotator** (Downstream tool that can directly process the same Agilent ".d" MS-files for feature finding and annotation of lipids)

## Examples

```
PNNLPreProcessor.exe --metadata-export --input raw_lipid_extract.d --output acquisition_metadata.txt
```

## Evaluation signals

- acquisition_metadata.txt exists in the output directory after running the export
- The exported text file contains the instrument/acquisition parameter fields reported for IM-MS acquisitions: electric field strength, pressure, temperature, and voltages
- Values are non-empty and correspond to the settings used for the specific data acquisition
- The input file was an original raw Agilent ".d" MS-file, not a converted or preprocessed derivative

## Limitations

- The tool as applied here processes Agilent ".d" instrument-format data; other vendor formats require conversion (e.g. to UIMF as an intermediate step) before the same algorithms can be applied
- The metadata export documents acquisition settings and parameters — it does not alter, smooth, or repair the raw signal itself

## Evidence

- [abstract] Metadata export: [evidence span withheld — anchor status: not_found]
- [results] Agilent .d processing: "Lipid Annotator can directly process Agilent ".d" MS-files for feature finding and annotation of lipids"
- [results] Tool availability: "The PreProcessor User Guide is freely available along with the software executable at https://omics.pnl.gov/software/pnnl-preprocessor"
- [discussion] Format conversion: "applicable and could be extended to other types of IM-MS instrumentation by respective vendors or through converting to UIMF as an intermediate step"
