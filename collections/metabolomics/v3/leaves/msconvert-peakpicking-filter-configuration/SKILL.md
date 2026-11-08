---
name: msconvert-peakpicking-filter-configuration
description: Use when vendor-format raw LC-MS/MS files (e.g., Thermo, Waters, Agilent,
  Bruker, SCIEX) must be converted to an open format (.mzXML/.mzML/.mgf) before uploading
  to GNPS-MassIVE for molecular networking.
when_to_use_negative:
- Input data are already in an open format (.mzXML, .mzML, or .mgf) and only need
  to be uploaded to MassIVE via FTP
- The analysis goal is calibrated quantitative measurement — classic molecular networking
  considers only MS2 scans and provides only qualitative information
- The target workflow is a vendor-optimized proteomics search rather than GNPS metabolomics;
  vendor-specific converters such as AB SCIEX MS Data Converter (Beta 1.3) may be
  the protocol-recommended alternative
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MSConvert (ProteoWizard)
  role: Performs the vendor-to-open-format conversion, applying the peak-picking (centroiding)
    filter for MS levels 1–2, 32-bit precision, and zlib compression disabled
  repo: http://proteowizard.sourceforge.net/downloads.shtml
- name: OpenMS TOPPView
  role: Spectrum viewer used to validate that the converted file is readable and contains
    MS2-level scans
  repo: https://github.com/OpenMS/OpenMS/releases
- name: WinSCP
  role: FTP client used to download the vendor-format raw LC-MS/MS file from the GNPS-MassIVE
    repository before conversion
  repo: https://winscp.net/eng/download.php
- name: GNPS
  role: Downstream platform that consumes the converted open-format files for molecular
    networking job submission
  repo: https://github.com/CCMS-UCSD/GNPSDocumentation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41596-020-0317-5
    title: Reproducible molecular networking of untargeted mass spectrometry data
      using GNPS
  source_content_hash: sha256:dc5b1debc71f246750e81f8ac667d35c1a52fdcb1128bfa2571695320f77edfe
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
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# msconvert-peakpicking-filter-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configures ProteoWizard MSConvert to convert vendor raw LC-MS/MS files into GNPS-usable open formats (.mzXML, .mzML, .mgf), applying the peak-picking (centroiding) vendor filter for MS levels 1–2 with 32-bit precision and no zlib compression, as specified in the GNPS molecular networking protocol.

## When to use

Use when vendor-format raw LC-MS/MS files (e.g., Thermo, Waters, Agilent, Bruker, SCIEX) must be converted to an open format (.mzXML/.mzML/.mgf) before uploading to GNPS-MassIVE for molecular networking.

## When NOT to use

- Input data are already in an open format (.mzXML, .mzML, or .mgf) and only need to be uploaded to MassIVE via FTP
- The analysis goal is calibrated quantitative measurement — classic molecular networking considers only MS2 scans and provides only qualitative information
- The target workflow is a vendor-optimized proteomics search rather than GNPS metabolomics; vendor-specific converters such as AB SCIEX MS Data Converter (Beta 1.3) may be the protocol-recommended alternative

## Inputs

- Vendor-format raw LC-MS/MS data file (e.g., downloaded from GNPS-MassIVE accession MSV000083437)
- Vendor-specific reader software when applicable (e.g., AB SCIEX Analyst 1.7, Agilent MassHunter, Bruker Data Analysis, Shimadzu LabSolutions, Thermo Xcalibur, Waters MassLynx)

## Outputs

- Single centroided .mzXML file (or .mzML; .mgf also supported by the protocol)
- Conversion log captured from the MSConvert console output

## How to apply

First download the vendor-format raw LC-MS/MS file from the GNPS-MassIVE repository (e.g., MSV000083437, the GF/SPF mouse dataset) using an FTP client such as WinSCP or Cyberduck. Then run ProteoWizard MSConvert with the peak-picking vendor filter enabled for MS levels 1–2 (centroiding), 32-bit floating-point binary encoding precision, and zlib compression deliberately disabled. Write the converted spectra to a single centroided output file in .mzXML format (or .mzML), capturing the converter's console output as a conversion log. Finally, validate the output by opening it in a spectrum viewer such as OpenMS TOPPView and confirming it is readable and contains MS2-level scans suitable for downstream GNPS molecular networking upload. Rationale: molecular networking requires instrument-specific raw data in an open, centroided format so MS2 scans can be clustered and matched against GNPS spectral libraries.

## Related tools

- **MSConvert (ProteoWizard)** (Performs the vendor-to-open-format conversion, applying the peak-picking (centroiding) filter for MS levels 1–2, 32-bit precision, and zlib compression disabled) — http://proteowizard.sourceforge.net/downloads.shtml
- **OpenMS TOPPView** (Spectrum viewer used to validate that the converted file is readable and contains MS2-level scans) — https://github.com/OpenMS/OpenMS/releases
- **WinSCP** (FTP client used to download the vendor-format raw LC-MS/MS file from the GNPS-MassIVE repository before conversion) — https://winscp.net/eng/download.php
- **GNPS** (Downstream platform that consumes the converted open-format files for molecular networking job submission) — https://github.com/CCMS-UCSD/GNPSDocumentation

## Examples

```
msconvert MSV000083437_raw.D --filter "peakPicking true 1-2" --32 --mzXML --nozlib
```

## Evaluation signals

- The converted file opens successfully in OpenMS TOPPView and displays readable spectra
- The output contains MS2-level scans suitable for downstream GNPS molecular networking upload
- MS2 spectra are centroided (peak-picking filter applied to MS levels 1–2), not profile data
- Binary encoding is 32-bit floating-point precision and no zlib compression is applied
- A conversion log captured from the MSConvert console output is retained for reproducibility

## Limitations

- The protocol does not specify exact software tool versions (e.g., ProteoWizard release) for the conversion step
- This is the protocol's 'traditional' conversion workflow; some vendors (e.g., AB SCIEX) also provide their own converters (AB SCIEX MS Data Converter Beta 1.3, Analyst 1.7) as alternatives
- Converted data support qualitative molecular networking only; quantitation requires follow-up targeted LC-MS

## Evidence

- [other] Conversion settings (32-bit, no zlib, peak picking 1-2): [evidence span withheld — anchor status: not_found]
- [readme] GNPS handles raw and processed MS/MS data in open access: [evidence span withheld — anchor status: external_doc]
