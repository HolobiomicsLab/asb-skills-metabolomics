---
name: reference-specimen-qc-comparison
description: Use when a Rapid QC-MS run includes a reference specimen and a prior
  library of high-confidence RS annotations (.msp) exists, and you need to quantify
  current RT shifts and mass accuracy drift versus previous analyses.
when_to_use_negative:
- The analysis contains no reference specimen or no prior library of reference-specimen
  annotations has been generated in a .msp file — there is nothing to align against.
- The goal is quality control of endogenous biological specimens rather than benchmarking
  instrument performance against a reference standard (use the standard IS/ES QC checks
  instead).
- The user is on macOS and only wants to monitor/view instrument run data without
  the MS-DIAL-based processing that RS comparison depends on.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Rapid QC-MS
  role: Runs the QC job, stores the uploaded .msp Biological Standard, and writes
    the RS-vs-library comparison output
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: MSConvert
  role: Converts vendor raw reference-specimen data into open mzML format before routing
    to the MS-DIAL processing pipeline
- name: MS-DIAL
  role: Processes the mzML files to extract RS annotations (m/z, RT, MS2) from the
    current run for alignment against the library
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:1efffff9b985397f258350cd545d6925c6084f563aaea0108b344e0df10097cd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/czbiohub-sf/Rapid-QC-MS
  repo_url: https://github.com/czbiohub-sf/Rapid-QC-MS
  tools:
  - MS-DIAL
  - MSConvert
  - Rapid QC-MS
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# reference-specimen-qc-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks current LC-MS/MS instrument performance by comparing annotations extracted from a reference specimen (RS) in the active Rapid QC-MS run against a user-supplied gold-standard .msp library of previously annotated reference specimens, reporting mass accuracy drift and retention time shifts.

## When to use

Use when a Rapid QC-MS run includes a reference specimen and a prior library of high-confidence RS annotations (.msp) exists, and you need to quantify current RT shifts and mass accuracy drift versus previous analyses.

## When NOT to use

- The analysis contains no reference specimen or no prior library of reference-specimen annotations has been generated in a .msp file — there is nothing to align against.
- The goal is quality control of endogenous biological specimens rather than benchmarking instrument performance against a reference standard (use the standard IS/ES QC checks instead).
- The user is on macOS and only wants to monitor/view instrument run data without the MS-DIAL-based processing that RS comparison depends on.

## Inputs

- Vendor raw LC-MS/MS data file (e.g. Thermo .raw) containing the reference specimen acquired in the current run
- User-supplied .msp-format library file combining analyte name, m/z, RT, and MS2 information from previously annotated reference specimens
- mzML files produced by MSConvert from the current run
- Rapid QC-MS QC job configured with the .msp library uploaded under Settings > Biological Standards

## Outputs

- rs_vs_msp_comparison.csv containing one row per aligned RS-to-library feature match, with fields for retention time shift and mass accuracy deviation of current annotations relative to the gold-standard .msp library
- Benchmark metrics (mass accuracy drift, RT shift) viewable on the Rapid QC-MS dashboard

## How to apply

First generate the gold-standard library: collect high-confidence annotations of the reference specimen in a well-defined biological matrix, then combine analyte name, m/z, RT, and MS2 information into an .msp-format file and upload it via Rapid QC-MS (Settings > Biological Standards). During the run, Rapid QC-MS converts vendor raw data to mzML with MSConvert and processes it with MS-DIAL to extract RS annotations from the current run. The skill then aligns the current RS annotations against the .msp library entries, matching features between the two annotation sets, and computes deviation metrics — mass accuracy drift and retention time shifts — for each matched feature. Alignment results are written to rs_vs_msp_comparison.csv with one row per RS-to-library feature match, so users can benchmark current instrument performance against previous analyses of the same reference standard.

## Related tools

- **Rapid QC-MS** (Runs the QC job, stores the uploaded .msp Biological Standard, and writes the RS-vs-library comparison output) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **MSConvert** (Converts vendor raw reference-specimen data into open mzML format before routing to the MS-DIAL processing pipeline)
- **MS-DIAL** (Processes the mzML files to extract RS annotations (m/z, RT, MS2) from the current run for alignment against the library)

## Examples

```
rapidqcms
```

## Evaluation signals

- rs_vs_msp_comparison.csv exists after the run and contains one row per aligned RS-to-library feature match
- The CSV includes explicit fields for retention time shift and mass accuracy deviation relative to the gold-standard .msp entries
- Matched features trace back to entries in the uploaded .msp library (analyte name, m/z, RT, MS2 fields agree with library records)
- Deviation magnitudes are consistent with expected instrument stability — no unexplained large mass-accuracy or RT shifts
- The .msp library itself was built from high-confidence annotations of the RS in a well-defined biological matrix before upload via Settings > Biological Standards

## Limitations

- Rapid QC-MS was designed for Windows because of its MSConvert and MS-DIAL dependencies; macOS users can only monitor/view instrument run data, not run the full RS comparison pipeline.
- The workflow has only been tested extensively on Thermo Fisher instruments, Thermo acquisition sequences, and Thermo RAW files; other vendor formats may still have processing bugs.
- Support for other vendors (Agilent, Bruker, Sciex, Waters) is under development and requires collaboration with the authors.
- A valid benchmark requires that the .msp library was generated from well-annotated reference specimens in a well-defined biological matrix; poorly annotated library entries will degrade the comparison.

## Evidence

- [supplementary] Generate library: generate a list of high confidence annotations in a well-defined biological matrix, then combine analyte name, m/z, RT, and MS2 information in a .msp-format file which they upload into Rapid QC-MS (Settings > Biological Standards).: [evidence span withheld — anchor status: not_found]
- [supplementary] RS feature alignment and deviation metrics: [evidence span withheld — anchor status: not_found]
