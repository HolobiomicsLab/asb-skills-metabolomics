---
name: lcms-metadata-table-curation
description: Use when preparing a GNPS molecular networking (METABOLOMICS-SNETS-V2)
  or library-search job from LC-MS/MS data and a sample metadata table must be collated,
  standardized, and attached so the analysis is publishable and reproducible.
when_to_use_negative:
- Input is already a processed feature table or an existing GNPS job output (bucket
  table / .graphml network) — metadata curation belongs to the data-upload stage,
  not network interpretation
- The dataset is private and cannot be made public — living-data annotation propagation
  in GNPS requires publicly accessible data
- In silico annotation of unannotated spectra is the goal — the article states such
  tools are beyond the scope of this networking protocol
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: GNPS
  role: Platform where the dataset and sample metadata table are registered and attached
    to molecular networking jobs
  repo: https://github.com/CCMS-UCSD/GNPSDocumentation
- name: MassIVE
  role: Public repository where converted LC-MS files and the metadata table are uploaded
    and made public
- name: WinSCP
  role: FTP client for uploading data files and metadata to MassIVE from Windows
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41596-020-0317-5
    title: Reproducible molecular networking of untargeted mass spectrometry data
      using GNPS
  source_content_hash: sha256:06fd2ccb701b0c839e5846565ec13e929f0545fa3ea9c0a41032f30ce3b60f3e
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
  - GNPS
  - MassIVE
  - WinSCP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# lcms-metadata-table-curation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Collate, standardize, and attach a sample information (metadata) table for LC-MS/MS data uploaded to MassIVE/GNPS so that molecular networking jobs are reproducible and annotations propagate across public datasets.

## When to use

Use when preparing a GNPS molecular networking (METABOLOMICS-SNETS-V2) or library-search job from LC-MS/MS data and a sample metadata table must be collated, standardized, and attached so the analysis is publishable and reproducible.

## When NOT to use

- Input is already a processed feature table or an existing GNPS job output (bucket table / .graphml network) — metadata curation belongs to the data-upload stage, not network interpretation
- The dataset is private and cannot be made public — living-data annotation propagation in GNPS requires publicly accessible data
- In silico annotation of unannotated spectra is the goal — the article states such tools are beyond the scope of this networking protocol

## Inputs

- LC-MS/MS raw or converted data files in .mzML, .mzXML, or .mgf format
- Instrument sample list / file names for the acquired dataset
- Sample attribute information (e.g. organism, condition: GF vs SPF mouse duodenum)

## Outputs

- Standardized sample information (metadata) table attached to a public MassIVE dataset
- Public MassIVE dataset accession (e.g. MSV000083437) with metadata linked to uploaded files

## How to apply

Convert instrument-specific raw files to open formats (.mzML, .mzXML, or .mgf, e.g. via MSConvert from ProteoWizard) and upload them to the GNPS-MassIVE repository through an FTP client such as WinSCP (Windows) or Cyberduck (macOS) or via the browser, then make the dataset public. Separately collate a sample information (metadata) table that maps each input file to its sample attributes — e.g. germ-free (GF) versus specific-pathogen-free (SPF) mouse duodenum in the MSV000083437 example — using standardized column headers, and attach this table to the dataset and to the molecular networking job. Attaching the metadata lets GNPS keep a continuous record of annotations, propagate knowledge across all public data as a living dataset, and allows each job to be shared or cloned. For the MSV000083437 tutorial job, verify the metadata table is attached before launching the METABOLOMICS-SNETS-V2 run so the resulting network and bucket table can be attributed to the correct GF/SPF sample groups.

## Related tools

- **GNPS** (Platform where the dataset and sample metadata table are registered and attached to molecular networking jobs) — https://github.com/CCMS-UCSD/GNPSDocumentation
- **MassIVE** (Public repository where converted LC-MS files and the metadata table are uploaded and made public)
- **WinSCP** (FTP client for uploading data files and metadata to MassIVE from Windows)

## Evaluation signals

- Every file listed in the metadata table corresponds to an uploaded .mzML/.mgf file in the MassIVE dataset, with no orphan or missing entries
- The dataset is public on MassIVE and has an accession number (e.g. MSV000083437) resolvable through the GNPS-MassIVE repository
- The molecular networking job page shows the sample metadata file attached to the job before submission
- Sample grouping in downstream network visualization (e.g. GF vs SPF) matches the attribute columns in the metadata table
- The job can be shared or cloned with the metadata intact, propagating annotations across public data

## Limitations

- Metadata curation alone does not make spectra annotatable — only ~5–6% of MS2 spectra in untargeted metabolomics are annotated; most remain 'dark matter'
- Metadata-correct networks still only provide qualitative information, since classic molecular networking considers only MS2 scans and cannot differentiate regio- or stereoisomers
- Statistics were not performed for the tutorial protocol example, so the attached metadata enables grouping but no quantitative claims in the tutorial dataset

## Evidence

- [intro] workflow step to collate and upload metadata: [evidence span withheld — anchor status: not_found]
- [supplementary] public data availability via GNPS-MassIVE: [evidence span withheld — anchor status: not_found]
