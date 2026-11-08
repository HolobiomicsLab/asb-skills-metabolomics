---
name: msms-spectral-similarity-molecular-networking
description: Use when you have untargeted LC-MS/MS data (converted to open formats
  like .mzXML/.mzML/.mgf) and want to group related molecules, dereplicate knowns
  via library matching, and propagate annotations across a spectral network.
when_to_use_negative:
- You need calibrated quantitative results — classic molecular networking can provide
  only qualitative information about the experiment because only MS2 scans are considered;
  targeted follow-up LC-MS is required
- You need to differentiate regio- or stereoisomers — MS2 spectra typically cannot
  differentiate them
- You need comprehensive compound identification — only ~5–6% of MS2 spectra in untargeted
  metabolomics get annotated and most remain unannotated 'dark matter'; in silico
  annotation tools are beyond the scope of this networking protocol
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: Hosts the METABOLOMICS-SNETS-V2 molecular networking workflow that clusters
    MS2 spectra, scores cosine similarity, and performs library search
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: MSConvert (ProteoWizard)
  role: Converts instrument-specific raw data files to open formats (.mzXML, .mzML)
    before uploading to MassIVE
- name: Cytoscape
  role: Visualization of the downloaded GNPS job output tables as a node/edge network
    for subnetwork extraction and annotation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41596-020-0317-5
    title: Reproducible molecular networking of untargeted mass spectrometry data
      using GNPS
  source_content_hash: sha256:d3fb69a62fbffd859d0c261bc9065c25a2aaa7583d8cc45c0b6a3240419b5814
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/CCMS-UCSD/GNPS_Workflows
  repo_url: https://github.com/CCMS-UCSD/GNPS_Workflows
  tools:
  - Cytoscape
  - GNPS
  - MSConvert (ProteoWizard)
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# MS/MS spectral-similarity molecular networking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Clusters consensus MS2 spectra into molecular-network nodes connected by cosine spectral similarity, then annotates nodes by GNPS spectral-library search — the core GNPS molecular networking analysis that organizes tandem MS data to reveal structurally related molecules.

## When to use

Use when you have untargeted LC-MS/MS data (converted to open formats like .mzXML/.mzML/.mgf) and want to group related molecules, dereplicate knowns via library matching, and propagate annotations across a spectral network.

## When NOT to use

- You need calibrated quantitative results — classic molecular networking can provide only qualitative information about the experiment because only MS2 scans are considered; targeted follow-up LC-MS is required
- You need to differentiate regio- or stereoisomers — MS2 spectra typically cannot differentiate them
- You need comprehensive compound identification — only ~5–6% of MS2 spectra in untargeted metabolomics get annotated and most remain unannotated 'dark matter'; in silico annotation tools are beyond the scope of this networking protocol

## Inputs

- LC-MS/MS raw data converted to open formats (.mzXML, .mzML, .mgf), e.g. files from GNPS-MassIVE accession MSV000083359
- Standardized sample information (metadata) table linking each file to its sample origin
- GNPS/MassIVE account for job submission

## Outputs

- Structured molecular network data table reflecting the molecular diversity captured in the tandem MS experiment (node/edge tables from the GNPS job)
- Library-search annotations per consensus spectrum (putative known molecule assignments)
- Cytoscape visual network of nodes and cosine-similarity edges, e.g. an extracted quinolone subnetwork node table (quinolone_subnetwork_node_table.csv)

## How to apply

Collect MS2 spectra, convert instrument-specific raw files to open formats (.mzXML, .mzML, .mgf), and upload them to the MassIVE repository with a standardized sample information (metadata) table. Submit a METABOLOMICS-SNETS-V2 molecular networking job on GNPS with default spectral filters enabled: the precursor ion window filter deleting all peaks within a ±17 Da range around the precursor ion mass, and the 50 Da window filter keeping only the top six most intense peaks per ±50 Da window. The workflow clusters MS2 spectra into consensus spectra (nodes), connects them by pairwise cosine spectral similarity (since structurally related molecules share fragment ion patterns under MS2 fragmentation), and queries each consensus spectrum against the GNPS spectral library using the default empirical cutoff of cosine 0.7 with 6 matched MS2 ions. Download the job output tables, import the network into Cytoscape (or view it in the GNPS web browser), and extract subnetworks of interest, annotating nodes with sample-origin metadata to propagate library annotations across connected nodes.

## Related tools

- **GNPS** (Hosts the METABOLOMICS-SNETS-V2 molecular networking workflow that clusters MS2 spectra, scores cosine similarity, and performs library search) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **MSConvert (ProteoWizard)** (Converts instrument-specific raw data files to open formats (.mzXML, .mzML) before uploading to MassIVE)
- **Cytoscape** (Visualization of the downloaded GNPS job output tables as a node/edge network for subnetwork extraction and annotation)

## Evaluation signals

- Verify library-search hits meet the default empirical cutoff of cosine 0.7 with 6 matched MS2 ions
- Confirm the precursor ion window filter removed all peaks within ±17 Da of the precursor mass and the 50 Da filter kept only the top six most intense peaks per window
- For the MSV000083359 tutorial, check the extracted subnetwork contains HHQ, NHQ, NHQ-C9:1, and NQNO at m/z 244.1696, 270.1852, 272.2009, and 288.1958 with the correct orange (lung tissue + isolates) versus cyan (isolates only) origin classification
- Confirm the network renders correctly in Cytoscape with GNPS job output tables (nodes and cosine edges) consistent with the published tutorial result

## Limitations

- Classic molecular networking provides only qualitative information because only MS2 scans are considered; calibrated quantitation requires follow-up targeted LC-MS
- Only ~5–6% of MS2 spectra in untargeted metabolomics are annotated — most spectra remain unannotated 'dark matter'
- MS2 spectra typically cannot differentiate regio- or stereoisomers
- Application of in silico annotation tools is beyond the immediate scope of the networking protocol presented here

## Evidence

- [intro] Library-search cosine cutoff: [evidence span withheld — anchor status: not_found]
- [intro] Structural basis of networking: [evidence span withheld — anchor status: not_found]
