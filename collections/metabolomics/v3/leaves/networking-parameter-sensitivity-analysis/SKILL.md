---
name: networking-parameter-sensitivity-analysis
description: Use when re-running or reproducing a GNPS METABOLOMICS-SNETS-V2 molecular
  networking job on a public dataset (e.g. MassIVE MSV000083381) and you need to characterize
  how clustering or tolerance parameters alter node counts and library annotations.
when_to_use_negative:
- Input is already a processed feature table or quantified molecular network output
  — this skill concerns the networking job parameters themselves
- The goal is calibrated quantitative analysis — classic molecular networking provides
  only qualitative information because only MS2 scans are considered
- Application of in silico annotation tools (e.g. structure prediction beyond library
  matching) is required — this is beyond the scope of the networking protocol
tools:
- name: GNPS
  role: Platform used to submit the METABOLOMICS-SNETS-V2 molecular networking jobs
    and to count nodes/annotations per parameter setting
  repo: https://ccms-ucsd.github.io/GNPSDocumentation
- name: MSConvert (ProteoWizard)
  role: Optional converter of instrument-specific raw data to open formats (.mzXML,
    .mzML, .mgf) prior to upload
- name: WinSCP
  role: FTP client for uploading the LC-MS/MS files to the MassIVE public repository
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41596-020-0317-5
    title: Reproducible molecular networking of untargeted mass spectrometry data
      using GNPS
  source_content_hash: sha256:bf7b03ec1447125177dc6feb2a9dec65519223bf253ee5b1231d0a15b6095aed
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
  - WinSCP
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# networking-parameter-sensitivity-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A systematic sweep of GNPS molecular networking parameters (Minimum Consensus Cluster Size, PIMT/FIMT precursor/fragment mass tolerances, and spectrum filtering options) on a fixed LC-MS/MS dataset to quantify how each setting changes subnetwork size and annotation yield.

## When to use

Use when re-running or reproducing a GNPS METABOLOMICS-SNETS-V2 molecular networking job on a public dataset (e.g. MassIVE MSV000083381) and you need to characterize how clustering or tolerance parameters alter node counts and library annotations.

## When NOT to use

- Input is already a processed feature table or quantified molecular network output — this skill concerns the networking job parameters themselves
- The goal is calibrated quantitative analysis — classic molecular networking provides only qualitative information because only MS2 scans are considered
- Application of in silico annotation tools (e.g. structure prediction beyond library matching) is required — this is beyond the scope of the networking protocol

## Inputs

- Public LC-MS/MS raw/converted data files (mzXML, mzML, or mgf) from MassIVE dataset MSV000083381
- GNPS METABOLOMICS-SNETS-V2 job parameter sets (MCC 1-4, PIMT/FIMT 2.0/0.5 vs 0.03, filtering on/off)
- Sample information (metadata) table uploaded to MassIVE

## Outputs

- fig7_sensitivity_table.csv with per-setting node and consensus-spectrum counts for the stenothricin subnetwork
- Per-job GNPS molecular networking outputs (node count, total consensus spectra, library annotations)

## How to apply

Retrieve the LC-MS/MS files for the target MassIVE accession (here MSV000083381) and submit one GNPS METABOLOMICS-SNETS-V2 (classic Molecular Networking) job per parameter setting. Sweep the Minimum Consensus Cluster Size (MCC) from 1 to 4 while holding Precursor Ion Mass Tolerance at 2.0 and Fragment Ion Mass Tolerance at 0.5 with spectrum filtering disabled, then run two comparison jobs: one with the advanced spectrum filters enabled and one at high-resolution PIMT/FIMT of 0.03. For each completed job, count the nodes and consensus spectra in the subnetwork of interest (e.g. the stenothricin-containing subnetwork, identified via library spectral matches with the default cosine > 0.7 and 6 matched MS2 ions) and the total spectra in the full network. Assemble per-setting counts into a single sensitivity table (fig7_sensitivity_table.csv). The rationale is that MCC acts as a clustering threshold collapsing structurally related spectra, while PIMT/FIMT tolerances control edge formation, so low-resolution tolerances (2.0/0.5) can preserve annotations that stringent 0.03 settings lose.

## Related tools

- **GNPS** (Platform used to submit the METABOLOMICS-SNETS-V2 molecular networking jobs and to count nodes/annotations per parameter setting) — https://ccms-ucsd.github.io/GNPSDocumentation
- **MSConvert (ProteoWizard)** (Optional converter of instrument-specific raw data to open formats (.mzXML, .mzML, .mgf) prior to upload)
- **WinSCP** (FTP client for uploading the LC-MS/MS files to the MassIVE public repository)

## Examples

```
https://gnps.ucsd.edu/ProteoSAFe/index.jsp?params=%7B%22workflow%22:%22METABOLOMICS-SNETS-V2%22
```

## Evaluation signals

- Stenothricin subnetwork node/spectrum counts must match the reported values: MCC 1-4 give 52/169, 29/144, 12/89, 7/73 nodes/spectra
- Advanced-filtering job must yield 9 nodes in the stenothricin subnetwork vs 26 without filtering
- PIMT/FIMT 0.03 job at MCC 1 must give 2 stenothricin nodes of 7,642 total spectra, and at MCC 2 no stenothricin subnetwork among 192 nodes
- Every parameter sweep uses identical inputs and holds non-swept parameters fixed (e.g. tolerances 2.0/0.5 with filtering disabled across MCC 1-4)
- Counts are assembled into a single complete table (fig7_sensitivity_table.csv) with one row per submitted job

## Limitations

- Stringent 0.03 mass tolerances at MCC 1 annotate only 2 stenothricin nodes of 7,642 total spectra, and at MCC 2 lose the stenothricin subnetwork entirely — low-resolution tolerances are needed to retain these analogs
- Classic molecular networking provides only qualitative information because only MS2 scans are considered
- Only ~5-6% of MS2 spectra in untargeted metabolomics are annotated; most spectra remain unannotated 'dark matter', limiting annotation-yield comparisons
- MS2 spectra typically cannot differentiate regio- or stereo-isomers, so subnetwork annotations are putative

## Evidence

- [intro] MCC and mass tolerances control network clustering: [evidence span withheld — anchor status: not_found]
- [intro] Default library-match thresholds used to identify annotated subnetwork nodes: [evidence span withheld — anchor status: not_found]
- [supplementary] Input dataset provenance: [evidence span withheld — anchor status: not_found]
- [readme] GNPS platform description supporting job submission: [evidence span withheld — anchor status: external_doc]
