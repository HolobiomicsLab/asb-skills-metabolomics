---
name: massive-repository-data-retrieval
description: Use when an analysis requires obtaining publicly available raw mass-spectrometry
  data from a GNPS-MassIVE accession (e.g., MSV000083437) so it can be converted to
  open formats for downstream GNPS workflows.
when_to_use_negative:
- The data are already available locally in an open format (.mzXML, .mzML, .mgf) and
  only conversion or molecular networking is needed
- The goal is submitting or making public a new dataset on MassIVE rather than retrieving
  an existing public one
- The task requires targeted quantitative follow-up LC-MS, since classic molecular
  networking 'can provide only qualitative information about the experiment because
  only MS2 scans are considered'
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: WinSCP
  role: FTP client used on Windows to download vendor-format raw LC-MS/MS data files
    from the GNPS-MassIVE repository
- name: Cyberduck
  role: FTP client used on Macintosh to retrieve datasets from GNPS-MassIVE
- name: GNPS-MassIVE repository
  role: Public repository hosting the raw LC-MS/MS datasets (e.g., MSV000083437) addressed
    by accession number
  repo: https://github.com/CCMS-UCSD/GNPSDocumentation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41596-020-0317-5
    title: Reproducible molecular networking of untargeted mass spectrometry data
      using GNPS
  source_content_hash: sha256:e521ebe3e23f0e85bdb42f22be33685803c7eaa43bd12769dab68bc4dac20fac
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/CCMS-UCSD/GNPS_Workflows
  repo_url: https://github.com/CCMS-UCSD/GNPSDocumentation
  tools:
  - Cyberduck
  - GNPS-MassIVE repository
  - OpenMS
  - WinSCP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# massive-repository-data-retrieval

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieves publicly deposited LC-MS/MS datasets from the GNPS-MassIVE repository by accession number using an FTP client, providing the vendor-format raw files needed before open-format conversion and molecular networking.

## When to use

Use when an analysis requires obtaining publicly available raw mass-spectrometry data from a GNPS-MassIVE accession (e.g., MSV000083437) so it can be converted to open formats for downstream GNPS workflows.

## When NOT to use

- The data are already available locally in an open format (.mzXML, .mzML, .mgf) and only conversion or molecular networking is needed
- The goal is submitting or making public a new dataset on MassIVE rather than retrieving an existing public one
- The task requires targeted quantitative follow-up LC-MS, since classic molecular networking 'can provide only qualitative information about the experiment because only MS2 scans are considered'

## Inputs

- GNPS-MassIVE accession identifier (e.g., MSV000083437)
- FTP client installed locally (WinSCP for Windows or Cyberduck for Macintosh)
- Local working directory for downloaded raw files

## Outputs

- Vendor-format raw LC-MS/MS data files downloaded from the MassIVE dataset
- Dataset file listing from the MassIVE accession used to confirm completeness

## How to apply

Locate the target dataset by its MassIVE accession (e.g., MSV000083437, the germ-free (GF) and specific-pathogen-free (SPF) mouse LC-MS/MS dataset used in the GNPS protocol). Install and configure an FTP client — WinSCP for Windows or Cyberduck for Macintosh — and point it at the MassIVE repository entry to download the vendor-format raw LC-MS/MS data files to a local working directory. Keep the downloaded vendor files intact, since the downstream step in this protocol converts them with ProteoWizard MSConvert (peak-picking for MS levels 1-2, 32-bit precision, zlib compression disabled) into .mzXML or .mzML. Rationale: the protocol deliberately uses public GNPS-MassIVE data ('No new data was collected for the tutorial example'), so correct retrieval and verification of the exact accession is what makes the molecular-networking tutorial reproducible.

## Related tools

- **WinSCP** (FTP client used on Windows to download vendor-format raw LC-MS/MS data files from the GNPS-MassIVE repository)
- **Cyberduck** (FTP client used on Macintosh to retrieve datasets from GNPS-MassIVE)
- **GNPS-MassIVE repository** (Public repository hosting the raw LC-MS/MS datasets (e.g., MSV000083437) addressed by accession number) — https://github.com/CCMS-UCSD/GNPSDocumentation

## Evaluation signals

- The downloaded files match the file listing published under the MassIVE accession (e.g., MSV000083437 for the GF/SPF mouse dataset)
- The retrieved vendor raw files open successfully in downstream tools (MSConvert conversion completes and TOPPView can display the converted spectra)
- The downloaded data are confirmed to be MS2-containing LC-MS/MS runs suitable for molecular networking upload
- The accession resolves on the public GNPS-MassIVE repository rather than a private or unpublished location

## Limitations

- The example dataset MSV000083437 (GF and SPF mice) is noted as 'data not shown' in the paper, so retrieval cannot be visually cross-checked against published figures
- Versions of the software tools (including FTP clients) are not specified in the protocol
- The protocol relies on public data availability; no new data was collected for the tutorial example, so failures in repository access block the entire workflow

## Evidence

- [methods] Download a vendor-format raw LC-MS/MS data file from the GNPS-MassIVE repository (e.g., MSV000083437, the GF/SPF mouse dataset) using an FTP client such as WinSCP or Cyberduck.: [evidence span withheld — anchor status: not_found]
- [supplementary] All LC-MS data publicly available at GNPS-MassIVE: [evidence span withheld — anchor status: not_found]
