---
name: feature-annotation-qc-granularity
description: Use when building or running untargeted LC-MS/MS QC with Rapid QC-MS
  and you must annotate reference-standard and external-standard features separately
  because RS and ES derive from different biological matrices, so each needs its own
  .msp target library and its own module.
when_to_use_negative:
- Input consists only of internal-standard (IS) features — IS, RS and ES are evaluated
  together in Rapid QC-MS, but the separate RS/ES module distinction described here
  applies only when you have matrix-mismatched standard types.
- Input data are already an annotated, matrix-matched feature table for both RS and
  ES from the same biological matrix — a single combined library search suffices and
  the separate ES module adds no QC granularity.
- You are running on macOS and expecting vendor-format conversion and MS-DIAL processing
  — the README notes MacOS users can only monitor/view run data, not perform the MSConvert/MS-DIAL-dependent
  annotation pipeline.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: Rapid QC-MS
  role: Hosts the separate ES and RS annotation modules and renders ES annotations
    on the QC dashboard distinct from RS
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: MS-DIAL ver.4.9.221218
  role: Performs the m/z and MS/MS or EI spectral library matching against the matrix-matched
    .msp for each standard module
- name: MSConvert 3.0.23332-7faa7f422
  role: Converts vendor .raw data into open mzML format before routing to the data
    processing pipeline
- name: Python
  role: Runtime for Rapid QC-MS (versions 3.8–3.11 compatible) executing the module
    logic and writing es_alignment_results.csv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:554c146a3d2faaf04e409c74dbf1e853a6000de9ea5a00546c29cf4194af90fa
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
  - MS-DIAL ver.4.9.221218
  - MSConvert 3.0.23332-7faa7f422
  - Python
  - Rapid QC-MS
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Feature annotation with QC granularity (separate RS and ES modules)

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

This skill covers Rapid QC-MS's practice of keeping reference-standard (RS) and external-standard (ES) feature annotation as distinct modules, each aligned against its own matrix-matched .msp spectral library, so that QC metrics remain interpretable when internal standards (IS), reference standards, and external standards come from different biological matrices.

## When to use

Use when building or running untargeted LC-MS/MS QC with Rapid QC-MS and you must annotate reference-standard and external-standard features separately because RS and ES derive from different biological matrices, so each needs its own .msp target library and its own module.

## When NOT to use

- Input consists only of internal-standard (IS) features — IS, RS and ES are evaluated together in Rapid QC-MS, but the separate RS/ES module distinction described here applies only when you have matrix-mismatched standard types.
- Input data are already an annotated, matrix-matched feature table for both RS and ES from the same biological matrix — a single combined library search suffices and the separate ES module adds no QC granularity.
- You are running on macOS and expecting vendor-format conversion and MS-DIAL processing — the README notes MacOS users can only monitor/view run data, not perform the MSConvert/MS-DIAL-dependent annotation pipeline.

## Inputs

- Rapid QC-MS ES feature annotations exported by the processing pipeline (MS-DIAL output)
- Predefined .msp spectral library matched to the ES analytical matrix
- Vendor raw data converted to open mzML format via MSConvert
- Thermo acquisition sequence .csv and example files from Zenodo (10.5281/zenodo.10525183)

## Outputs

- ES-specific feature annotation results (best library hit per ES feature with match score, m/z error, retention-time agreement)
- es_alignment_results.csv, kept distinct from RS alignment output
- ES annotations rendered in the Rapid QC-MS dashboard separately from RS QC results

## How to apply

Set up the external-standard (ES) workflow almost identically to the reference-standard (RS) workflow, but configure the two as distinct modules rather than a single combined annotation pass. The key decision point is the molecular-targets library: because ES and RS are often not from the same biological matrix, supply a .msp spectral library matched to the ES analytical matrix so searches can target a different set of compounds than the RS library. Convert vendor .raw data to open mzML with MSConvert (version 3.0.23332–7faa7f422), then process with MS-DIAL ver.4.9.221218, matching each ES feature's m/z and MS/MS or EI spectrum against the ES .msp entries. Record each ES feature's best library hit with match score, m/z error, and retention-time agreement in an ES-specific results table (es_alignment_results.csv) kept separate from RS alignment output, preserving QC granularity in the dashboard.

## Related tools

- **Rapid QC-MS** (Hosts the separate ES and RS annotation modules and renders ES annotations on the QC dashboard distinct from RS) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **MS-DIAL ver.4.9.221218** (Performs the m/z and MS/MS or EI spectral library matching against the matrix-matched .msp for each standard module)
- **MSConvert 3.0.23332-7faa7f422** (Converts vendor .raw data into open mzML format before routing to the data processing pipeline)
- **Python** (Runtime for Rapid QC-MS (versions 3.8–3.11 compatible) executing the module logic and writing es_alignment_results.csv)

## Examples

```
python rapidqcms --config es_workflow.yaml  # or simply: rapidqcms
```

## Evaluation signals

- es_alignment_results.csv exists and contains alignment fields (match score, m/z error, retention time) for the queried ES features.
- ES alignment output is written as a separate module from RS, with no mixed RS/ES library hits in the ES results table.
- The ES .msp library entries correspond to the ES analytical matrix (different molecular targets from the RS library where matrices differ).
- MS-DIAL ver.4.9.221218 is the processing engine used for the spectral matching step, as documented in the article's tool list.
- Dashboard shows ES annotations visually distinct from RS QC metrics, preserving per-standard-type granularity.

## Limitations

- Rapid QC-MS is only extensively tested on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files; other vendor formats may exhibit bugs in the processing pipeline, including annotation.
- MSConvert and MS-DIAL must be installed manually — Python dependencies install automatically but these two do not, so the ES workflow cannot run without explicit setup.
- Rapid QC-MS was designed to run on Windows platforms because of MSConvert and MS-DIAL dependencies; MacOS users can only monitor/view run data.
- No changelog was found for the tool (per the article's discussion).

## Evidence

- [intro] ES and RS libraries differ by matrix: "Evaluating IS, RS and ES together"
- [intro] ES workflow mirror of RS but distinct modules: "Evaluating IS, RS and ES together"
- [readme] Matrix-type target library requirement: "MSConvert converts raw acquired data into open mzML format before routing it to the data processing pipeline"
- [readme] Separate module per standard type: "MSConvert converts raw acquired data into open mzML format before routing it to the data processing pipeline"
- [readme] MS-DIAL processing dependency: "dependencies such as MSConvert and MS-DIAL will need to be installed manually."
- [readme] Vendor-format support limitation: "Rapid QC-MS has only been tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files."
- [readme] Windows design rationale: [evidence span withheld — anchor status: not_found]
