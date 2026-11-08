---
name: raw-mass-spectrometry-file-conversion
description: Use when you have instrument-vendor raw MS/MS files that must be converted
  to .mzXML/.mzML/.mgf before uploading to GNPS-MassIVE or submitting a molecular
  networking job.
when_to_use_negative:
- Data are already in an open format (.mzXML, .mzML, .mgf) and only need upload to
  MassIVE
- You need feature detection/quantitation (e.g., MZmine2 feature tables for Feature-Based
  Molecular Networking) rather than raw spectrum conversion
- The vendor requires instrument-specific software conversion (e.g., AB SCIEX Analyst
  1.7) that MSConvert does not support for that file type
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MSConvert (ProteoWizard)
  role: Primary converter applying the peak-picking filter (MS1-2), 32-bit precision,
    zlib disabled, writing .mzXML/.mzML
  repo: http://proteowizard.sourceforge.net/downloads.shtml
- name: WinSCP
  role: FTP client for downloading the vendor raw file from the GNPS-MassIVE repository
  repo: https://winscp.net/eng/download.php
- name: OpenMS
  role: TOPPView spectrum viewer used to validate the converted centroided file
  repo: https://github.com/OpenMS/OpenMS/releases
- name: AB SCIEX MS Data Converter (Beta 1.3)
  role: Alternative vendor-specific converter for AB SCIEX raw data
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41596-020-0317-5
    title: Reproducible molecular networking of untargeted mass spectrometry data
      using GNPS
  source_content_hash: sha256:e3465f207982e53ff85db894dcc507ba24dd872987839c60d6435b5b95562834
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/CCMS-UCSD/GNPS_Workflows
  tools:
  - AB SCIEX MS Data Converter (Beta 1.3)
  - MSConvert (ProteoWizard)
  - OpenMS
  - WinSCP
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# raw-mass-spectrometry-file-conversion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts vendor-specific raw LC-MS/MS files (e.g., Thermo, AB SCIEX, Agilent, Waters, Bruker, Shimadzu formats) into open formats (.mzXML, .mzML, .mgf) required for upload and analysis in GNPS-MassIVE, using ProteoWizard MSConvert with peak picking, 32-bit precision, and zlib compression disabled.

## When to use

Use when you have instrument-vendor raw MS/MS files that must be converted to .mzXML/.mzML/.mgf before uploading to GNPS-MassIVE or submitting a molecular networking job.

## When NOT to use

- Data are already in an open format (.mzXML, .mzML, .mgf) and only need upload to MassIVE
- You need feature detection/quantitation (e.g., MZmine2 feature tables for Feature-Based Molecular Networking) rather than raw spectrum conversion
- The vendor requires instrument-specific software conversion (e.g., AB SCIEX Analyst 1.7) that MSConvert does not support for that file type

## Inputs

- vendor-format raw LC-MS/MS data file (e.g., Thermo Xcalibur, AB SCIEX Analyst, Agilent MassHunter, Waters MassLynx, Bruker Data Analysis, Shimadzu LabSolutions formats)
- MSConvert peak-picking vendor filter settings for MS levels 1-2

## Outputs

- single centroided open-format file in .mzXML (or .mzML/.mgf)
- conversion log (MSConvert console output)

## How to apply

Download a vendor-format raw LC-MS/MS file (e.g., MSV000083437, the GF/SPF mouse dataset) from the GNPS-MassIVE repository via an FTP client such as WinSCP or Cyberduck. Run ProteoWizard MSConvert on the raw file with the peak-picking vendor filter enabled for MS levels 1-2 (centroiding), 32-bit floating-point binary encoding precision, and zlib compression left unchecked/disabled. Write the spectra to a single centroided output file in .mzXML (or .mzML), capturing the converter's console output as a conversion log. The rationale is that GNPS workflows require open, centroided formats; the protocol also lists vendor tools like AB SCIEX MS Data Converter (Beta 1.3) and AB SCIEX Analyst 1.7 as alternative converters for specific instruments. Validate the output by opening it in OpenMS TOPPView and confirming it is readable and contains MS2-level scans suitable for downstream GNPS molecular networking.

## Related tools

- **MSConvert (ProteoWizard)** (Primary converter applying the peak-picking filter (MS1-2), 32-bit precision, zlib disabled, writing .mzXML/.mzML) — http://proteowizard.sourceforge.net/downloads.shtml
- **WinSCP** (FTP client for downloading the vendor raw file from the GNPS-MassIVE repository) — https://winscp.net/eng/download.php
- **OpenMS** (TOPPView spectrum viewer used to validate the converted centroided file) — https://github.com/OpenMS/OpenMS/releases
- **AB SCIEX MS Data Converter (Beta 1.3)** (Alternative vendor-specific converter for AB SCIEX raw data)

## Examples

```
msconvert raw.D --filter "peakPicking true 1-2" --mzXML --32 --nozlib -o converted/ > msconvert.log
```

## Evaluation signals

- Converted file opens and renders spectra in OpenMS TOPPView without errors
- Output file is centroided (peaks appear as discrete m/z/intensity points, not profile data) and contains MS2-level scans
- Binary encoding is 32-bit and zlib compression was not applied (checked in the file's spectrum encoding metadata)
- Console output captured during conversion is retained as a conversion log for reproducibility

## Limitations

- MS2 spectra alone provide only qualitative information; classic molecular networking cannot supply calibrated quantitation
- Only ~5-6% of MS2 spectra in untargeted metabolomics are annotated, so conversion alone does not solve identification 'dark matter'
- Software versions of the conversion tools are not specified in the protocol

## Evidence

- [other] vendor mass-spectrometry files converted to .mzXML are configured with 32-bit binary encoding precision, with zlib compression deliberately not applied: [evidence span withheld — anchor status: not_found]
- [supplementary] All LC-MS data used in this paper are publicly available at the GNPS-MassIVE repository: [evidence span withheld — anchor status: not_found]
