---
name: annotation-alignment-separation-from-rs-module
description: Use when QC specimens include external standards acquired in a different
  biological matrix than the reference standard, and their features must be matched
  to a separate .msp target library independent of the RS module.
when_to_use_negative:
- The specimen's standards come from the same biological matrix as the reference standard
  — in that case use the RS (reference standard) module rather than a separate ES
  library
- Input is already a completed feature-abundance QC report and no m/z or MS/MS annotation
  alignment against a .msp library is required
- Working on a MacOS-only setup, since MSConvert and MS-DIAL (needed for mzML conversion
  and .msp alignment) are not installed there
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Rapid QC-MS
  role: Runs the ES annotation-alignment module as part of its automated LC-MS/MS
    quality-control pipeline and renders es_alignment_results.csv in the dashboard
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: MS-DIAL ver.4.9.221218
  role: Performs the m/z and MS/MS spectral similarity alignment of ES features against
    the ES-specific .msp library
  repo: http://prime.psc.riken.jp/compms/msdial/main.html
- name: MSConvert 3.0.23332-7faa7f422
  role: Converts vendor RAW acquisition files into open mzML format before the data
    processing pipeline routes ES features to annotation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:d713f41790af444df104d138d4f88b4dbcc6a2e535b78a36a23a1647d01be81d
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
  - Rapid QC-MS
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# annotation-alignment-separation-from-rs-module

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configure and run Rapid QC-MS's external standard (ES) annotation-alignment module, which matches ES features against a molecular-targets .msp library that is deliberately kept separate from the internal standard (IS) and reference standard (RS) libraries because ES and RS are often not from the same biological matrix.

## When to use

Use when QC specimens include external standards acquired in a different biological matrix than the reference standard, and their features must be matched to a separate .msp target library independent of the RS module.

## When NOT to use

- The specimen's standards come from the same biological matrix as the reference standard — in that case use the RS (reference standard) module rather than a separate ES library
- Input is already a completed feature-abundance QC report and no m/z or MS/MS annotation alignment against a .msp library is required
- Working on a MacOS-only setup, since MSConvert and MS-DIAL (needed for mzML conversion and .msp alignment) are not installed there

## Inputs

- ES feature annotations from the Rapid QC-MS processing pipeline (MS-DIAL output after mzML conversion)
- Predefined .msp spectral library for the ES analytical matrix
- Vendor RAW files converted to mzML via MSConvert

## Outputs

- es_alignment_results.csv containing ES feature annotations with match score, m/z error, and retention-time agreement
- ES-specific annotations rendered in the Rapid QC-MS dashboard

## How to apply

Load the ES feature annotations exported by the Rapid QC-MS processing pipeline — MS-DIAL ver.4.9.221218 output after MSConvert 3.0.23332-7faa7f422 has converted vendor RAW files to mzML — together with a predefined .msp spectral library matched to the ES analytical matrix. Match each ES feature's m/z and MS/MS or EI spectrum against the .msp entries by spectral similarity scoring in MS-DIAL, keeping this configuration distinct from the RS library setup. Record the best library hit's match score, m/z error, and retention-time agreement in an ES-specific results table, then write the aligned ES annotation results to es_alignment_results.csv for downstream dashboard visualization. The rationale is that ES and RS are often not from the same biological matrix, so allowing different molecular targets per module preserves QC granularity.

## Related tools

- **Rapid QC-MS** (Runs the ES annotation-alignment module as part of its automated LC-MS/MS quality-control pipeline and renders es_alignment_results.csv in the dashboard) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **MS-DIAL ver.4.9.221218** (Performs the m/z and MS/MS spectral similarity alignment of ES features against the ES-specific .msp library) — http://prime.psc.riken.jp/compms/msdial/main.html
- **MSConvert 3.0.23332-7faa7f422** (Converts vendor RAW acquisition files into open mzML format before the data processing pipeline routes ES features to annotation)

## Examples

```
rapidqcms  # configure the ES job to use a matrix-specific .msp library; results are written to es_alignment_results.csv
```

## Evaluation signals

- es_alignment_results.csv exists and contains alignment fields (match score, m/z error, retention-time agreement) for the queried ES features
- The ES module runs as a distinct configuration — its .msp library entries differ from those of the RS module when the matrices differ
- Each ES feature has exactly one best library hit recorded with a nonzero match score
- Dashboard renders the ES annotations separately from IS/RS annotations with correct matrix-specific molecular targets
- ES alignment does not alter or overwrite the RS alignment results table

## Limitations

- Rapid QC-MS has only been tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files, so ES alignment on other vendors may have bugs
- MSConvert and MS-DIAL must be installed manually — they are not bundled with the pip install
- Rapid QC-MS was designed for Windows platforms because of its MSConvert and MS-DIAL dependencies; MacOS users can only monitor/view run data
- ES and RS are often not from the same biological matrix, so reusing the RS library for ES features will misalign molecular targets

## Evidence

- [methods] ES workflow key distinction is the molecular targets library: [evidence span withheld — anchor status: not_found]
- [methods] MS-DIAL is the alignment engine: [evidence span withheld — anchor status: not_found]
- [methods] MSConvert produces mzML input: "MSConvert converts raw acquired data into open mzML format before routing it to the data processing pipeline"
- [other] External-standard output artifact name: [evidence span withheld — anchor status: not_found]
- [readme] Manual dependency installation is required: "Python dependencies are installed automatically, but dependencies such as MSConvert and MS-DIAL will need to be installed manually."
