---
name: instrument-format-preserving-ms-file-writing
description: Use when you have preprocessed (smoothed, demultiplexed, spike-removed)
  LC-IM-MS TOF spectra and need to save them in the original instrument's file format
  for downstream processing.
when_to_use_negative:
- The input is already a centroided peak/feature table rather than raw instrument-format
  MS data.
- The target downstream tool requires a different interchange format (e.g. mzML/UIMF)
  and the article gives no support for writing to it here.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PNNL PreProcessor
  role: Standalone software that generates new MS-files with enhanced signal quality
    and in the same instrument format as the input
  repo: https://omics.pnl.gov/software/pnnl-preprocessor
- name: Lipid Annotator
  role: Downstream tool that can directly process Agilent .d MS-files for feature
    finding and annotation of lipids, motivating format-preserving output
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00425
    title: A Preprocessing Tool for Enhanced Ion Mobility–Mass Spectrometry-Based
      Omics Workflows
  source_content_hash: sha256:2e1be3e0873abbb20300053b5327f91942d7f7ca511ffd10e24fb82ad5d8ce8f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Lipid Annotator
  - PNNL PreProcessor
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# instrument-format-preserving-ms-file-writing

## Summary

Writes preprocessed LC-IM-MS TOF data back into the same instrument file format as the input (e.g. Agilent MassHunter .d) so downstream tools like Lipid Annotator can consume the enhanced spectra without format conversion.

## When to use

Use when you have preprocessed (smoothed, demultiplexed, spike-removed) LC-IM-MS TOF spectra and need to save them in the original instrument's file format for downstream processing.

## When NOT to use

- The input is already a centroided peak/feature table rather than raw instrument-format MS data.
- The target downstream tool requires a different interchange format (e.g. mzML/UIMF) and the article gives no support for writing to it here.

## Inputs

- Original raw LC-IM-MS TOF MS-file (e.g. Agilent MassHunter .d from an Agilent 6560 IM-QTOF in All Ions mode)
- Preprocessed spectra (smoothed/demultiplexed) produced by PNNL PreProcessor algorithms

## Outputs

- New MS-file in the same instrument format as the input (e.g. Agilent .d), with profile-mode TOF spectra, original times preserved, and an algorithm suffix appended to the filename

## How to apply

After applying algorithms such as multidimensional moving-average smoothing (3 points in IM, 3 points in LC for lipid workflows), the PreProcessor writes the enhanced spectra to a new MS-file 'in the same instrument format as the input'. For lipid extract raw data acquired on an Agilent 6560 IM-QTOF in All Ions mode, output is written as Agilent MassHunter .d. Each TOF spectrum is written in profile mode without centroiding, and original times are preserved; an algorithm suffix (e.g. '_MA-d3-c3') is appended to distinguish the preprocessed file. The rationale is that downstream tools such as Lipid Annotator directly process Agilent .d files, so preserving the instrument format avoids an intermediate conversion step.

## Related tools

- **PNNL PreProcessor** (Standalone software that generates new MS-files with enhanced signal quality and in the same instrument format as the input) — https://omics.pnl.gov/software/pnnl-preprocessor
- **Lipid Annotator** (Downstream tool that can directly process Agilent .d MS-files for feature finding and annotation of lipids, motivating format-preserving output)

## Examples

```
PreProcessor --input original_lipid_extract.d --algorithm MA --d3 --c3 --output smoothed_lipid_extract_MA-d3-c3.d
```

## Evaluation signals

- The smoothed output file must exist and be readable in the original .d instrument format (script runs and file-format check).
- Each TOF spectrum in the output is written in profile mode without centroiding, with original times preserved.
- The output filename carries an algorithm suffix (analogous to the paper's '_MA-d3-c3' convention).
- Downstream Lipid Annotator processing succeeds on the PreProcessor-generated .d files (paper: 19.4% more lipid annotations, 7 min vs 34 min runtime).

## Limitations

- The tool writes output 'in the same instrument format as the input', so other types of IM-MS instrumentation may require converting to UIMF as an intermediate step.
- File-size reduction and memory efficiency are claimed, but the discussion text does not give explicit limitations for format handling.
- No explicit repository/code URL or tool version is stated in the discussion text.

## Evidence

- [results] same instrument format as input: [evidence span withheld — anchor status: not_found]
