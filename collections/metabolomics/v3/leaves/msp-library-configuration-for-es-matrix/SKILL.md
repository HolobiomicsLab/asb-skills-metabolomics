---
name: msp-library-configuration-for-es-matrix
description: Use when running Rapid QC-MS QC on LC-MS/MS data whose external standards
  (ES) come from a different biological matrix than the reference standards (RS),
  and ES feature annotations must be matched against a separate .
when_to_use_negative:
- When the ES and RS originate from the same biological matrix and share the same
  molecular-target library — in that case a single RS library configuration suffices
  and no separate ES module is needed.
- When the input is already a curated feature table or annotation result that does
  not require spectral alignment against a new .msp library.
- When the instrument data is not from a Thermo RAW file or another vendor format
  convertible via MSConvert, since Rapid QC-MS has only been tested extensively on
  Thermo Fisher RAW files.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Rapid QC-MS
  role: 'Hosts the ES module: loads ES feature annotations, coordinates library matching,
    writes es_alignment_results.csv, and surfaces results on the interactive dashboard'
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: MS-DIAL ver.4.9.221218
  role: Performs spectral library matching of ES features' m/z and MS/MS or EI spectra
    against the .msp library via spectral similarity scoring
  repo: http://prime.psc.riken.jp/compms/msdial/main.html
- name: MSConvert 3.0.23332-7faa7f422
  role: Converts vendor raw acquisition data into open mzML format before routing
    to the data processing pipeline that generates ES feature annotations
  repo: https://proteowizard.sourceforge.io/tools/msconvert.html
- name: Python
  role: Runtime environment (versions 3.8–3.11) in which Rapid QC-MS and its pandas/SQLAlchemy/Plotly
    Dash dependencies execute to orchestrate the ES library workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:084751211600034441e0eeb2a2d5f79c544626baa37bfa56d3bb5bfc02b54d96
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
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# msp-library-configuration-for-es-matrix

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configure and apply a matrix-matched .msp spectral library inside Rapid QC-MS's external standards (ES) module so that ES features are aligned and annotated independently of the reference standard (RS) library, preserving QC granularity for molecular targets specific to the ES biological matrix.

## When to use

Use when running Rapid QC-MS QC on LC-MS/MS data whose external standards (ES) come from a different biological matrix than the reference standards (RS), and ES feature annotations must be matched against a separate .msp library with m/z, MS/MS/EI spectra, match score, m/z error, and retention-time agreement recorded per feature.

## When NOT to use

- When the ES and RS originate from the same biological matrix and share the same molecular-target library — in that case a single RS library configuration suffices and no separate ES module is needed.
- When the input is already a curated feature table or annotation result that does not require spectral alignment against a new .msp library.
- When the instrument data is not from a Thermo RAW file or another vendor format convertible via MSConvert, since Rapid QC-MS has only been tested extensively on Thermo Fisher RAW files.

## Inputs

- ES feature annotations exported by the Rapid QC-MS processing pipeline (MS-DIAL output)
- Predefined .msp spectral library matched to the ES analytical matrix
- Vendor RAW files (converted via MSConvert 3.0.23332-7faa7f422 to mzML) for ES runs

## Outputs

- es_alignment_results.csv — ES alignment table containing match score, m/z error, and retention-time agreement per queried feature
- ES-specific annotation results (best library hit per feature) for downstream dashboard visualization in Rapid QC-MS

## How to apply

Build the ES job in Rapid QC-MS nearly identically to the RS workflow but keep the two in distinct modules, supplying an ES-specific .msp library that corresponds to the ES analytical matrix, since ES and RS are often not from the same biological matrix and thus target different molecular species. In MS-DIAL ver.4.9.221218, match each ES feature's m/z and MS/MS (or EI) spectrum against the ES .msp library entries using spectral similarity scoring, retaining match score, m/z error, and retention-time agreement in an ES-specific results table. Load the ES feature annotations exported by the Rapid QC-MS processing pipeline (MS-DIAL output) alongside the predefined .msp library to complete the alignment. Write the aligned ES annotation results to es_alignment_results.csv so the dashboard can visualize them. Decision point: if the ES and RS matrices differ, configure a separate .msp library rather than reusing the RS library, because matrix type dictates which molecular targets are searched. MSConvert 3.0.23332-7faa7f422 is used upstream to convert vendor RAW into open mzML format before routing to the pipeline.

## Related tools

- **Rapid QC-MS** (Hosts the ES module: loads ES feature annotations, coordinates library matching, writes es_alignment_results.csv, and surfaces results on the interactive dashboard) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **MS-DIAL ver.4.9.221218** (Performs spectral library matching of ES features' m/z and MS/MS or EI spectra against the .msp library via spectral similarity scoring) — http://prime.psc.riken.jp/compms/msdial/main.html
- **MSConvert 3.0.23332-7faa7f422** (Converts vendor raw acquisition data into open mzML format before routing to the data processing pipeline that generates ES feature annotations) — https://proteowizard.sourceforge.io/tools/msconvert.html
- **Python** (Runtime environment (versions 3.8–3.11) in which Rapid QC-MS and its pandas/SQLAlchemy/Plotly Dash dependencies execute to orchestrate the ES library workflow)

## Examples

```
python -m rapidqcms  # start Rapid QC-MS; configure a new QC job supplying an ES .msp library matched to the ES analytical matrix, with MS-DIAL aligning features and writing es_alignment_results.csv
```

## Evaluation signals

- es_alignment_results.csv exists and contains alignment fields (match score, m/z error, retention time) for the queried ES features
- Each ES feature row carries a non-null best-library-hit assignment with a recorded match score and an m/z error within the library's tolerance window
- Retention-time agreement between matched ES features and library entries is consistent across runs, showing no systematic drift over hours/days as flagged by the Rapid QC-MS dashboard
- The ES module's alignment results remain distinct from the RS module's output, confirming the two workflows are separated as required for QC granularity
- The .msp library used in the ES job corresponds to the ES analytical matrix (different molecular targets from the RS library when matrices differ)

## Limitations

- Rapid QC-MS has only been tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files; other vendor formats may contain bugs.
- MacOS users can monitor/view instrument run data but cannot run the full ES processing pipeline because of the MSConvert dependency.
- MSConvert and MS-DIAL must be installed manually as they are not bundled with Rapid QC-MS.
- ES and RS are often not from the same biological matrix, so the ES library must be reconfigured rather than reused from the RS module.

## Evidence

- [methods] Source card task_006: [evidence span withheld — anchor status: not_found]
