---
name: rs-annotation-library-alignment
description: Use when Rapid QC-MS has processed an ongoing LC-MS/MS run and you need
  to compare the RS annotations from the current analysis against historical RS annotations
  stored in a user-supplied .msp library file to detect drift in mass accuracy or
  retention time.
when_to_use_negative:
- The biological matrix has changed or no previously annotated reference specimens
  exist, so no gold-standard .msp library can be constructed from prior analyses.
- Only internal standard (IS) or evaluation specimen (ES) data need QC — the RS annotation
  alignment step is specific to reference standard annotations, not general IS RT/m/z/intensity
  monitoring.
- The input is already an aligned feature table rather than raw vendor files or mzML;
  MSConvert/MS-DIAL-based RS extraction and alignment would be redundant.
edam_operation: http://edamontology.org/operation_3741
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0218
tools:
- name: Rapid QC-MS
  role: Provides the Biological Standards interface to load the user-supplied .msp
    library and processes the current run's RS annotations for alignment against the
    gold-standard entries.
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: MSConvert
  role: Converts raw vendor data (e.g., Thermo .raw) into open mzML format before
    routing it to the MS-DIAL data processing pipeline that generates RS annotations.
  repo: https://proteowizard.sourceforge.io/tools/msconvert.html
- name: MS-DIAL
  role: Data processing and identification step that extracts the RS annotations produced
    from the current Rapid QC-MS run.
  repo: http://prime.psc.riken.jp/compms/msdial/main.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:b74c2493ef31aecc1335c05c580584905ac51345a485fb8adf46bcd7e637873b
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
  edam_operation: http://edamontology.org/operation_3741
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# rs-annotation-library-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark current LC-MS/MS instrument performance by aligning reference standard (RS) annotations from the ongoing Rapid QC-MS run against a previously curated gold-standard .msp library, computing mass accuracy drift and retention time shifts. This enables comparison of today's RS annotations with those from earlier analyses in the same well-defined biological matrix.

## When to use

Use when Rapid QC-MS has processed an ongoing LC-MS/MS run and you need to compare the RS annotations from the current analysis against historical RS annotations stored in a user-supplied .msp library file to detect drift in mass accuracy or retention time.

## When NOT to use

- The biological matrix has changed or no previously annotated reference specimens exist, so no gold-standard .msp library can be constructed from prior analyses.
- Only internal standard (IS) or evaluation specimen (ES) data need QC — the RS annotation alignment step is specific to reference standard annotations, not general IS RT/m/z/intensity monitoring.
- The input is already an aligned feature table rather than raw vendor files or mzML; MSConvert/MS-DIAL-based RS extraction and alignment would be redundant.

## Inputs

- User-supplied .msp library file containing annotations (analyte name, m/z, RT, MS2 information) from previously annotated reference specimens in a well-defined biological matrix
- Current run RS annotations extracted via MS-DIAL data processing of mzML files converted by MSConvert
- Rapid QC-MS job configuration with the .msp library loaded under Settings > Biological Standards

## Outputs

- rs_vs_msp_comparison.csv containing one row per aligned RS-to-library feature match
- Per-feature retention time shift values (current RS annotations relative to the gold-standard .msp library)
- Per-feature mass accuracy deviation values (current RS annotations relative to the gold-standard .msp library)

## How to apply

First, a previously annotated reference specimen in a well-defined biological matrix yields high-confidence annotations that are combined by analyte name, m/z, retention time, and MS2 information into a single .msp-format file, which is uploaded into Rapid QC-MS via Settings > Biological Standards. During the current run, Rapid QC-MS ingests raw vendor files (e.g., Thermo .raw), converts them to open mzML with MSConvert, and processes them with MS-DIAL to extract RS annotations. The current RS annotations are then aligned against the .msp library annotations by matching features between the two annotation sets. Deviation metrics—mass accuracy drift and retention time shifts—are computed for each aligned RS-to-library feature match, and the results are written to rs_vs_msp_comparison.csv. Correct application requires that the output file exists, contains one row per aligned match, and includes explicit fields for the RT shift and mass accuracy deviation relative to the gold-standard library entries.

## Related tools

- **Rapid QC-MS** (Provides the Biological Standards interface to load the user-supplied .msp library and processes the current run's RS annotations for alignment against the gold-standard entries.) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **MSConvert** (Converts raw vendor data (e.g., Thermo .raw) into open mzML format before routing it to the MS-DIAL data processing pipeline that generates RS annotations.) — https://proteowizard.sourceforge.io/tools/msconvert.html
- **MS-DIAL** (Data processing and identification step that extracts the RS annotations produced from the current Rapid QC-MS run.) — http://prime.psc.riken.jp/compms/msdial/main.html

## Examples

```
rapidqcms  # start Rapid QC-MS and configure the job's Biological Standards tab with the .msp library file
```

## Evaluation signals

- rs_vs_msp_comparison.csv exists after the run completes.
- The CSV contains one row per aligned RS-to-library feature match.
- The CSV includes explicit fields for retention time shift and mass accuracy deviation of current annotations relative to the gold-standard .msp library.
- Matched features align between the two annotation sets (current RS annotations vs. .msp library entries).
- Computed deviations fall within instrument-performance expectations when the LC-MS/MS is behaving normally; anomalies in the metrics correspond to known drifts or acute deviations.

## Limitations

- Rapid QC-MS has only been tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files, so other vendor formats may behave unexpectedly in the RS annotation pipeline.
- MSConvert and MS-DIAL must be installed manually; Python dependencies (Pandas, SQLAlchemy, Plotly Dash, etc.) are installed automatically, but missing manual dependencies will prevent RS annotation generation.
- The RS benchmarking relies on a well-defined biological matrix for the reference specimen; changes in matrix composition can confound the alignment-based comparison.
- Rapid QC-MS was designed to run on Windows platforms because of its dependency on MSConvert and MS-DIAL; MacOS users can only monitor/view instrument run data, not run the full RS annotation alignment pipeline.

## Evidence

- [methods] RS benchmarking workflow: [evidence span withheld — anchor status: not_found]
- [methods] validation of output: [evidence span withheld — anchor status: not_found]
