---
name: ms2-scan-content-quality-checking
description: Use when a vendor-format raw LC-MS/MS file (e.g., from GNPS-MassIVE dataset
  MSV000083437) has been converted to .mzXML or .mzML and must be confirmed readable
  and MS2-scan-containing before molecular networking upload.
when_to_use_negative:
- Input is still an unconverted vendor raw file — run the MSConvert conversion step
  first before quality-checking scan content
- Analysis requires calibrated quantitative information rather than qualitative MS2-based
  networking, since classic molecular networking uses only MS2 scans
- Input is already a processed feature table (e.g., MZmine2 output) rather than a
  spectrum-level converted data file
tools:
- name: MSConvert (ProteoWizard)
  role: Performs the vendor-to-open-format conversion (peak-picking MS levels 1-2,
    32-bit precision, zlib compression disabled) whose output is being quality-checked
  repo: http://proteowizard.sourceforge.net/downloads.shtml
- name: OpenMS TOPPView
  role: Spectrum viewer used to validate that the converted file is readable and contains
    MS2-level scans
  repo: https://github.com/OpenMS/OpenMS/releases
- name: WinSCP
  role: FTP client used to download the vendor raw LC-MS/MS file from the GNPS-MassIVE
    repository prior to conversion and checking
  repo: https://winscp.net/eng/download.php
- name: GNPS
  role: Downstream platform whose molecular networking workflow requires validated
    MS2-scan-containing open-format input files
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41596-020-0317-5
    title: Reproducible molecular networking of untargeted mass spectrometry data
      using GNPS
  source_content_hash: sha256:3c7982d6a2b25b954c7e6a067b2b7b93d781675d1ac277ab18fc2fe01551b4e6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/CCMS-UCSD/GNPS_Workflows
  tools:
  - GNPS
  - MSConvert (ProteoWizard)
  - OpenMS
  - OpenMS TOPPView
  - WinSCP
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# ms2-scan-content-quality-checking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates that vendor-raw LC-MS/MS data converted to open formats (.mzXML, .mzML, .mgf) via ProteoWizard MSConvert actually contain readable MS2-level scans, which are the required input for downstream GNPS molecular networking.

## When to use

Use when a vendor-format raw LC-MS/MS file (e.g., from GNPS-MassIVE dataset MSV000083437) has been converted to .mzXML or .mzML and must be confirmed readable and MS2-scan-containing before molecular networking upload.

## When NOT to use

- Input is still an unconverted vendor raw file — run the MSConvert conversion step first before quality-checking scan content
- Analysis requires calibrated quantitative information rather than qualitative MS2-based networking, since classic molecular networking uses only MS2 scans
- Input is already a processed feature table (e.g., MZmine2 output) rather than a spectrum-level converted data file

## Inputs

- Vendor-format raw LC-MS/MS data file (downloaded from GNPS-MassIVE, e.g., MSV000083437)
- Converted centroided .mzXML or .mzML output file
- MSConvert console output / conversion log

## Outputs

- Validated centroided .mzXML (or .mzML) file containing MS2-level scans
- Conversion log capturing MSConvert console output
- Pass/fail confirmation of readability in OpenMS TOPPView

## How to apply

Run ProteoWizard MSConvert on the downloaded vendor raw file with the peak-picking vendor filter enabled for MS levels 1-2 (centroiding), 32-bit floating-point binary precision, and zlib compression disabled, writing a single centroided output file in .mzXML (or .mzML). Capture the converter's console output as a conversion log. Then open the converted file in a spectrum viewer such as OpenMS TOPPView and confirm it loads without error and contains MS2-level scans, since classic molecular networking relies exclusively on MS2 spectra. Only a file that passes both the readability and MS2-content checks is suitable for uploading to GNPS-MassIVE and submitting to a molecular networking job.

## Related tools

- **MSConvert (ProteoWizard)** (Performs the vendor-to-open-format conversion (peak-picking MS levels 1-2, 32-bit precision, zlib compression disabled) whose output is being quality-checked) — http://proteowizard.sourceforge.net/downloads.shtml
- **OpenMS TOPPView** (Spectrum viewer used to validate that the converted file is readable and contains MS2-level scans) — https://github.com/OpenMS/OpenMS/releases
- **WinSCP** (FTP client used to download the vendor raw LC-MS/MS file from the GNPS-MassIVE repository prior to conversion and checking) — https://winscp.net/eng/download.php
- **GNPS** (Downstream platform whose molecular networking workflow requires validated MS2-scan-containing open-format input files)

## Examples

```
msconvert MSV000083437_raw.d --filter "peakPicking true 1-2" --32 --mzXML -o converted/
```

## Evaluation signals

- Converted file opens without error in OpenMS TOPPView
- File contains MS2-level scans (not MS1 only), suitable for GNPS molecular networking upload
- Binary encoding precision is 32-bit floating point with zlib compression not applied
- Output was written to a single centroided file in .mzXML (or .mzML) format
- MSConvert console output was captured as a conversion log for provenance

## Limitations

- Classic molecular networking provides only qualitative information because only MS2 scans are considered; an MS2-content check cannot substitute for quantitative validation
- Conversion quality depends on MSConvert vendor reader support for the specific instrument format; some vendors require vendor-specific software (e.g., AB SCIEX MS Data Converter, Bruker DataAnalysis, Waters MassLynx) before open-format export
- If the peak-picking vendor filter for MS levels 1-2 is misconfigured, the output may silently lack usable MS2 scans — hence the mandatory TOPPView inspection

## Evidence

- [intro] two structurally related molecules share fragment ion patterns when subjected to MS2 fragmentation methods: [evidence span withheld — anchor status: not_found]
- [supplementary] All LC–MS data used in this paper are publicly available at the GNPS-MassIVE repository under the following accession numbers.: [evidence span withheld — anchor status: not_found]
- [readme] aims to be an open-access knowledge base for community-wide organization and sharing of raw, processed or identified tandem mass (MS/MS) spectrometry data: [evidence span withheld — anchor status: external_doc]
