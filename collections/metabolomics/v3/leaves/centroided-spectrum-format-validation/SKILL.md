---
name: centroided-spectrum-format-validation
description: Use when raw instrument-specific mass-spectrometry files (e.g. from Agilent
  MassHunter, Thermo Xcalibur, AB SCIEX, Bruker, Waters MassLynx, Shimadzu) must be
  converted to an open format for GNPS molecular networking.
when_to_use_negative:
- Input data is already in an open format (.mzXML/.mzML) or already centroided peak
  lists (.mgf) — no vendor conversion step is needed
- Vendor-specific converter routes are required instead of the 'traditional' MSConvert
  path, e.g. AB SCIEX Analyst/WIFF files handled with AB SCIEX MS Data Converter (Beta
  1.3) as listed in the protocol's software table
- The analysis goal is calibrated quantitation — classic molecular networking uses
  only MS2 scans and is qualitative, so conversion alone does not address quantitative
  follow-up
edam_operation: http://edamontology.org/operation_2409
tools:
- name: MSConvert (ProteoWizard)
  role: performs the vendor-to-open-format conversion with the peak-picking filter,
    32-bit precision, and no zlib compression
  repo: http://proteowizard.sourceforge.net/downloads.shtml
- name: OpenMS TOPPView
  role: spectrum viewer used to validate that the converted file opens and contains
    usable MS2 scans
  repo: https://github.com/OpenMS/OpenMS/releases
- name: WinSCP
  role: FTP client for downloading vendor raw files from the GNPS-MassIVE repository
    before conversion
  repo: https://winscp.net/eng/download.php
- name: GNPS
  role: target platform whose workflows and MassIVE uploads define the required converted,
    centroided file format
  repo: https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41596-020-0317-5
    title: Reproducible molecular networking of untargeted mass spectrometry data
      using GNPS
  source_content_hash: sha256:1953f016f25948b742e70bba58ebb455494f7f1e45a9a228dcb219b66ac4356e
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# centroided-spectrum-format-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Convert vendor raw LC-MS/MS files into centroided open formats (.mzXML/.mzML) via ProteoWizard MSConvert with GNPS-compatible settings, then validate the converted spectra before uploading to GNPS-MassIVE.

## When to use

Use when raw instrument-specific mass-spectrometry files (e.g. from Agilent MassHunter, Thermo Xcalibur, AB SCIEX, Bruker, Waters MassLynx, Shimadzu) must be converted to an open format for GNPS molecular networking.

## When NOT to use

- Input data is already in an open format (.mzXML/.mzML) or already centroided peak lists (.mgf) — no vendor conversion step is needed
- Vendor-specific converter routes are required instead of the 'traditional' MSConvert path, e.g. AB SCIEX Analyst/WIFF files handled with AB SCIEX MS Data Converter (Beta 1.3) as listed in the protocol's software table
- The analysis goal is calibrated quantitation — classic molecular networking uses only MS2 scans and is qualitative, so conversion alone does not address quantitative follow-up

## Inputs

- Vendor-format raw LC-MS/MS data file (e.g. Thermo, Agilent MassHunter, Waters MassLynx, AB SCIEX, Bruker, Shimadzu proprietary formats)
- Target open format choice (.mzXML, .mzML, or .mgf)

## Outputs

- Single centroided converted file in .mzXML or .mzML containing MS1 and MS2 scans
- MSConvert console output captured as a conversion log
- Validated, GNPS-upload-ready spectra file

## How to apply

Download a vendor-format raw file (e.g. MSV000083437 from GNPS-MassIVE) using WinSCP or Cyberduck, then run MSConvert applying the peak-picking vendor filter across MS levels 1–2 to centroid MS1 and MS2 scans. Configure 32-bit binary encoding precision and deliberately leave zlib compression disabled, then write the spectra to a single centroided output file in .mzXML (or .mzML, or .mgf for peak lists). Open the converted file in a spectrum viewer such as OpenMS TOPPView to confirm it is readable, and capture the MSConvert console output as a conversion log. Validation succeeds when MS2-level scans are present, since downstream GNPS library search and molecular networking jobs consume MS2 scans.

## Related tools

- **MSConvert (ProteoWizard)** (performs the vendor-to-open-format conversion with the peak-picking filter, 32-bit precision, and no zlib compression) — http://proteowizard.sourceforge.net/downloads.shtml
- **OpenMS TOPPView** (spectrum viewer used to validate that the converted file opens and contains usable MS2 scans) — https://github.com/OpenMS/OpenMS/releases
- **WinSCP** (FTP client for downloading vendor raw files from the GNPS-MassIVE repository before conversion) — https://winscp.net/eng/download.php
- **GNPS** (target platform whose workflows and MassIVE uploads define the required converted, centroided file format) — https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp

## Examples

```
msconvert MSV000083437_raw_sample.raw --mzXML --32 --filter "peakPicking true 1-2" -o converted/
```

## Evaluation signals

- Converted file opens without error in OpenMS TOPPView and displays readable spectra
- Converted file contains MS2-level scans suitable for downstream GNPS molecular networking upload
- Output is centroided (peak-picking vendor filter applied at MS levels 1-2), not profile data
- Binary encoding is 32-bit floating-point precision and zlib compression is absent, matching the protocol's specified settings
- A conversion log of the MSConvert console output has been captured alongside the converted file

## Limitations

- The GF/SPF mouse dataset (MSV000083437) used to demonstrate the tutorial was not shown in the paper itself
- Classic molecular networking after this conversion is qualitative — only MS2 scans are considered, so quantitation requires follow-up targeted LC-MS analysis
- After conversion and library matching, only ~5–6% of MS2 spectra in untargeted metabolomics are annotated; the remainder remains unannotated 'dark matter'
- MS2 spectra typically cannot differentiate regio- or stereoisomers regardless of conversion settings

## Evidence

- [methods] task card finding, data-conversion workflow: [evidence span withheld — anchor status: not_found]
- [intro] workflow step (ii): "(ii) converting instrument-speci fi c raw data fi les to an open format"
- [supplementary] data availability statement: [evidence span withheld — anchor status: not_found]
