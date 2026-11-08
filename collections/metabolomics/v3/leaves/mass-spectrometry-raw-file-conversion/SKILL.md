---
name: mass-spectrometry-raw-file-conversion
description: Use when newly acquired vendor-format raw files (e.g., Thermo .raw from
  an Orbitrap sequence) must be made pipeline-ready in open mzML format before MS-DIAL
  feature extraction and internal-standard QC.
when_to_use_negative:
- Input data are already in open mzML format and need no vendor-format conversion
- Data are from Agilent, Bruker, Sciex, or Waters instruments whose acquisition sequences
  and files have not been extensively tested with this pipeline
- The host platform is not Windows and vendor conversion is required (macOS users
  can monitor/view data but the conversion pipeline depends on MSConvert)
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MSConvert 3.0.23332-7faa7f422
  role: Converts raw vendor data into open mzML format before routing it to the Rapid
    QC-MS data processing pipeline; must be installed manually
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: Rapid QC-MS
  role: 'Orchestrates conversion: detects newly written raw files and invokes MSConvert
    as the first pipeline stage before MS-DIAL processing'
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: MS-DIAL ver.4.9.221218
  role: Receives the converted mzML files and performs data processing/identification;
    downloaded and installed separately
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:3e2c221600bfd95701016040a6f60a41d9d90088f21197952f07cd5324b937f6
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
  - MSConvert
  - MSConvert 3.0.23332-7faa7f422
  - Rapid QC-MS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# mass-spectrometry-raw-file-conversion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Convert vendor-proprietary LC-MS/MS raw files (e.g., Thermo .raw) into the open mzML format with MSConvert as the first stage of the Rapid QC-MS pipeline, enabling downstream MS-DIAL processing and quality-control dashboarding regardless of instrument vendor.

## When to use

Use when newly acquired vendor-format raw files (e.g., Thermo .raw from an Orbitrap sequence) must be made pipeline-ready in open mzML format before MS-DIAL feature extraction and internal-standard QC.

## When NOT to use

- Input data are already in open mzML format and need no vendor-format conversion
- Data are from Agilent, Bruker, Sciex, or Waters instruments whose acquisition sequences and files have not been extensively tested with this pipeline
- The host platform is not Windows and vendor conversion is required (macOS users can monitor/view data but the conversion pipeline depends on MSConvert)

## Inputs

- Vendor-proprietary LC-MS/MS raw data files (e.g., Thermo .raw files from an Orbitrap acquisition sequence)
- Thermo acquisition sequence .csv (context: example files from Zenodo 10.5281/zenodo.10525183)

## Outputs

- One mzML file per raw input file, routed to the MS-DIAL data processing pipeline
- Downstream IS QC metrics (retention time, intensity, Δm/z per specimen) derived from the converted mzML

## How to apply

Launch the Rapid QC-MS pipeline with the 'rapidqcms' command; MSConvert (version 3.0.23332–7faa7f422) runs as a dependency to convert each completed raw file into open mzML before routing it to the data processing pipeline. Ensure MSConvert and MS-DIAL (ver.4.9.221218) are installed manually alongside Python 3.8–3.11, since these dependencies are not installed by pip. Because conversion to mzML is the vendor-agnostic entry point, newly written files are detected automatically (Watchdog file watching) and converted within minutes so that MS-DIAL can extract internal-standard RT, intensity, and Δm/z metrics. The decision point is vendor support: only Thermo Fisher acquisition sequences and RAW files have been extensively tested, so treat other vendor conversions as unvalidated.

## Related tools

- **MSConvert 3.0.23332-7faa7f422** (Converts raw vendor data into open mzML format before routing it to the Rapid QC-MS data processing pipeline; must be installed manually) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **Rapid QC-MS** (Orchestrates conversion: detects newly written raw files and invokes MSConvert as the first pipeline stage before MS-DIAL processing) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **MS-DIAL ver.4.9.221218** (Receives the converted mzML files and performs data processing/identification; downloaded and installed separately)

## Examples

```
rapidqcms
```

## Evaluation signals

- Every completed raw file in the run yields a corresponding mzML file in the processing directory within minutes of file writing
- Each mzML file is accepted by MS-DIAL ver.4.9.221218 and yields internal-standard peak metrics (RT, intensity, Δm/z) per specimen
- Conversion occurs upstream of data processing: no IS QC results appear in the dashboard for a specimen until its mzML is produced
- The conversion path works for any vendor format in principle because mzML is the common interchange — verify at least Thermo .raw conversions, the only extensively tested case
- MSConvert version matches the article's stated version (3.0.23332–7faa7f422) and was installed manually alongside MS-DIAL

## Limitations

- Rapid QC-MS has only been tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files; other vendor formats may contain bugs
- The pipeline depends on MSConvert, so it was designed to run on Windows platforms; macOS users can monitor/view data but not run the full conversion pipeline
- MSConvert and MS-DIAL must be installed manually — they are not installed automatically with the rapid-qc-ms pip package

## Evidence
