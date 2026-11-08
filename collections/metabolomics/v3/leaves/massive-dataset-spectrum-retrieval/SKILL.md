---
name: massive-dataset-spectrum-retrieval
description: 'Use when a task provides an mzspec: USI referencing a MassIVE accession
  (MSV...) with a file path and scan index (e.g. :scan:62886) and the underlying spectrum
  peaks, precursor m/z, or MS level must be retrieved.'
when_to_use_negative:
- The input is already a local spectrum file or feature table and no USI resolution
  against a remote repository is needed
- The data source is not one the dashboard supports (e.g. a repository lacking a _resolve_usi_remotelink
  implementation in download.py)
- The goal is chromatographic/XIC comparison across multiple files rather than extracting
  a single scan's spectrum
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GNPS_LCMSDashboard
  role: Provides the USI resolution and download logic (_resolve_usi_remotelink in
    download.py) used to fetch the spectrum file from MassIVE
  repo: https://github.com/Wang-Bioinformatics-Lab/GNPS_LCMSDashboard
- name: Universal Spectrum Identifier (USI)
  role: Addressing scheme encoding repository, dataset, file path, and scan index
    of the target spectrum
- name: mzML format (open MS data format)
  role: File format of the centroided proteomics run (e.g. 01697_A01_P018020_S00_N01_R2.mzML)
    parsed to extract the scan record
provenance:
  source_task_ids:
  - task_007
  - task_001
  source_papers:
  - doi: 10.1038/s41592-021-01339-5
    title: 'GNPS Dashboard: collaborative exploration of mass spectrometry data in
      the web browser'
  source_content_hash: sha256:31ccda710db53c371048b8e7cd325008c8e9910c4f7ea6b58aa9c459b0a676b9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Wang-Bioinformatics-Lab/GNPS_LCMSDashboard
  repo_url: https://github.com/Wang-Bioinformatics-Lab/GNPS_LCMSDashboard
  tools:
  - GNPS_LCMSDashboard
  - Universal Spectrum Identifier (USI)
  - mzML format (open MS data format)
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# massive-dataset-spectrum-retrieval

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Resolves a Universal Spectrum Identifier (USI) pointing at a MassIVE-hosted mzML/mzXML file to the underlying spectrum record at a specific scan index, even deep inside large centroided proteomics runs, using the GNPS LC-MS Dashboard's USI download-resolution logic.

## When to use

Use when a task provides an mzspec: USI referencing a MassIVE accession (MSV...) with a file path and scan index (e.g. :scan:62886) and the underlying spectrum peaks, precursor m/z, or MS level must be retrieved.

## When NOT to use

- The input is already a local spectrum file or feature table and no USI resolution against a remote repository is needed
- The data source is not one the dashboard supports (e.g. a repository lacking a _resolve_usi_remotelink implementation in download.py)
- The goal is chromatographic/XIC comparison across multiple files rather than extracting a single scan's spectrum

## Inputs

- USI string of the form mzspec:MSV000083508:ccms_peak_centroided/pituitary_hypophysis/Trypsin_HCD_QExactiveplus/01697_A01_P018020_S00_N01_R2.mzML:scan:62886
- Remote mzML or mzXML file hosted on MassIVE (centroided proteomics run)

## Outputs

- Spectrum record at the specified scan index: MS level, precursor m/z, charge, and centroided peak list (e.g. serialized as scan_62886.json)
- Resolution status and resolved remote download URL for the USI

## How to apply

Parse the USI string into its repository prefix (MassIVE MSV accession), relative file path, and scan specifier. Resolve the remote download URL for the USI following the GNPS_LCMSDashboard logic implemented in download.py, specifically _resolve_usi_remotelink, then fetch the referenced mzML (or mzXML) file. Parse the file with pyteomics or pymzML and extract the spectrum record at the cited scan index (e.g. 62886), capturing MS level, precursor m/z, charge, and the centroided peak list. Validate that the extracted record is an MS2 spectrum with a non-empty peak array before serializing the result.

## Related tools

- **GNPS_LCMSDashboard** (Provides the USI resolution and download logic (_resolve_usi_remotelink in download.py) used to fetch the spectrum file from MassIVE) — https://github.com/Wang-Bioinformatics-Lab/GNPS_LCMSDashboard
- **Universal Spectrum Identifier (USI)** (Addressing scheme encoding repository, dataset, file path, and scan index of the target spectrum)
- **mzML format (open MS data format)** (File format of the centroided proteomics run (e.g. 01697_A01_P018020_S00_N01_R2.mzML) parsed to extract the scan record)

## Examples

```
usi = 'mzspec:MSV000083508:ccms_peak_centroided/pituitary_hypophysis/Trypsin_HCD_QExactiveplus/01697_A01_P018020_S00_N01_R2.mzML:scan:62886'; spectra = list(pyteomics.mzml.read(mzml_file)); rec = spectra[62886]
```

## Evaluation signals

- The USI parses cleanly into repository prefix (MSV accession), file path, and scan index components
- The resolved remote download URL returns a parseable mzML/mzXML file (not an error page)
- The extracted record at the cited scan index is an MS2 spectrum with a non-empty peak array
- Precursor m/z, charge, and MS level are captured in the serialized output (e.g. scan_62886.json)
- Retrieval succeeds on a scan deep into a large file (scan 62886 of the pituitary mzML), confirming correct index-based extraction rather than truncation

## Limitations

- Only data sources with an implemented USI resolution path are supported; new sources require updating download.py in _resolve_usi_remotelink
- Very large proteomics mzML files must be fully downloaded before scan-index extraction, which can be expensive
- The scan specifier is an index into the file, so correctness depends on the file's internal spectrum ordering

## Evidence

- [supplementary] The article lists mzspec:MSV000083508:ccms_peak_centroided/pituitary_hypophysis/Trypsin_HCD_QExactiveplus/01697_A01_P018020_S00_N01_R2.mzML:scan:62886 as an example USI that the dashboard serves: "mzspec:MSV000083508:ccms_peak_centroided/pituitary_hypophysis/Trypsin_HCD_QExactiveplus/01697_A01_P018020_S00_N01_R2.mzML:scan:62886"
- [readme] The README instructs implementers to add data sources by modifying the USI remote-link resolver in download.py.: "Update the code in ```download.py```, specifically in ```_resolve_usi_remotelink``` to implement how to get the remote URL for your new USI."
- [intro] The dashboard supports proteomics and metabolomics datasets from GNPS/MassIVE and MetaboLights via USI references.: [evidence span withheld — anchor status: not_found]
