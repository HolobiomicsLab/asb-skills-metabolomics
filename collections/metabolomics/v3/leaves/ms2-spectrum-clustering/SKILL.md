---
name: ms2-spectrum-clustering
description: Use when processing raw MS2 peak lists (e.g. .mzXML) for classic molecular
  networking and a reduced set of representative consensus spectra is needed instead
  of redundant duplicate scans.
when_to_use_negative:
- Input is already a processed feature table or quantified abundance matrix — MS-Cluster
  operates on raw MS2 spectra only.
- 'Quantitation is required: classic molecular networking considers only MS2 scans
  and is qualitative, so calibrated quantitation needs follow-up targeted LC-MS or
  feature-based workflows.'
- Running Feature-Based Molecular Networking, where features rather than MS-Cluster
  consensus spectra define the nodes.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GNPS
  role: Hosts the METABOLOMICS-SNETS-V2 molecular networking workflow whose MS-Cluster
    step collapses identical MS2 spectra into consensus spectra (nodes)
  repo: http://github.com/CCMS-UCSD/GNPS_Workflows
- name: MSConvert (ProteoWizard)
  role: Converts instrument-specific raw data files to open formats (.mzXML, .mzML,
    .mgf) suitable for clustering
- name: GNPS-MassIVE repository
  role: Source of the public LC-MS input datasets (e.g. MSV000083437) to be clustered
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41596-020-0317-5
    title: Reproducible molecular networking of untargeted mass spectrometry data
      using GNPS
  source_content_hash: sha256:11b635488b0facebe012ea1177c1a973104df770262df1b2152d81e9c16450b4
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
  - GNPS-MassIVE repository
  - MSConvert (ProteoWizard)
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# ms2-spectrum-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs the MS-Cluster consensus-clustering step of GNPS classic molecular networking, in which nearly identical MS2 spectra are collapsed into a single consensus spectrum (node) that becomes the input for network construction and spectral library searching.

## When to use

Use when processing raw MS2 peak lists (e.g. .mzXML) for classic molecular networking and a reduced set of representative consensus spectra is needed instead of redundant duplicate scans.

## When NOT to use

- Input is already a processed feature table or quantified abundance matrix — MS-Cluster operates on raw MS2 spectra only.
- Quantitation is required: classic molecular networking considers only MS2 scans and is qualitative, so calibrated quantitation needs follow-up targeted LC-MS or feature-based workflows.
- Running Feature-Based Molecular Networking, where features rather than MS-Cluster consensus spectra define the nodes.

## Inputs

- MS2 peak-list files in open formats (.mzXML, .mzML, .mgf), e.g. from dataset MSV000083437 on the GNPS-MassIVE repository
- GNPS molecular networking job parameters (parent-ion mass tolerance, Minimum Cluster Size, 'Run MSCluster' flag)

## Outputs

- Consensus-cluster table with columns for cluster index, precursor m/z, member-spectrum count, and consensus peak list (e.g. MSV000083437_consensus_clusters.tsv)
- Consensus MS2 spectra serving as nodes for molecular networking and library searching

## How to apply

Parse the MS2 spectra from the input peak-list files (e.g. .mzXML downloaded from GNPS-MassIVE) and first apply the GNPS pre-clustering spectrum filters: delete all peaks in a ±17 Da range around the precursor ion mass, keep only the top six most intense peaks within each ±50 Da window, and delete fragment ions below the minimum peak intensity threshold. Then group nearly identical MS2 spectra whose precursor ion m/z values fall within the parent-ion mass tolerance, as implemented in the GNPS METABOLOMICS-SNETS-V2 workflow with 'Run MSCluster' set to yes. Apply the user-set Minimum Cluster Size filter, discarding clusters with too few member spectra: use 5 or more for hundreds of files, 1–2 for one or two files (for the tutorial dataset MSV000083437 a value of 4 was used). For each retained cluster, merge the fragment peaks of its member spectra into a consensus spectrum and write the consensus-cluster table (cluster index, precursor m/z, member-spectrum count, consensus peak list). The rationale is that each consensus spectrum (node) is subsequently queried against spectral library databases for annotation and connected by pairwise spectral similarity in the network.

## Related tools

- **GNPS** (Hosts the METABOLOMICS-SNETS-V2 molecular networking workflow whose MS-Cluster step collapses identical MS2 spectra into consensus spectra (nodes)) — http://github.com/CCMS-UCSD/GNPS_Workflows
- **MSConvert (ProteoWizard)** (Converts instrument-specific raw data files to open formats (.mzXML, .mzML, .mgf) suitable for clustering)
- **GNPS-MassIVE repository** (Source of the public LC-MS input datasets (e.g. MSV000083437) to be clustered)

## Evaluation signals

- The consensus-cluster table should be comparable (cluster count, precursor m/z values, member counts) to the 'view all clusters with IDs' output of the corresponding GNPS molecular networking job on MSV000083437.
- Every output consensus spectrum must have a member-spectrum count ≥ the chosen Minimum Cluster Size (4 for the MSV000083437 tutorial; clusters below the size are discarded).
- Consensus spectra must contain no peaks within ±17 Da of the precursor ion mass and at most the top six most intense peaks per ±50 Da window, reflecting the pre-clustering filters.
- Each consensus cluster's precursor m/z must be consistent with the parent-ion mass tolerance used to group member spectra.
- Member-spectrum counts should scale with dataset size (larger values, 5+, for hundreds of files; 1–2 for one or two files), so spot-check counts against the number of input files.

## Limitations

- Classic molecular networking provides only qualitative information because only MS2 scans are considered; calibrated quantitation requires follow-up targeted LC-MS.
- The Minimum Cluster Size trades sensitivity for noise control — too large a value on a small dataset (e.g. 1–2 files) discards real consensus spectra, while too small a value on large datasets admits noise.
- Consensus spectra inherit the limits of MS2: typically ~5–6% of spectra in untargeted metabolomics are annotated and most remain unannotated 'dark matter'.

## Evidence

- [intro] classic molecular networking can provide only qualitative information about the experiment because only MS2 scans are considered: [evidence span withheld — anchor status: not_found]
