---
name: consensus-spectrum-generation
description: Use when raw MS2 peak lists (.mzXML/.mzML) must be deduplicated into
  consensus spectra before molecular networking or library search, and the Minimum
  Cluster Size and precursor mass tolerance need to be justified.
when_to_use_negative:
- Input is already a feature table or quantified peak area table — consensus clustering
  operates on MS2 scans only and yields no quantitative information (classic networking
  is qualitative)
- The goal is isomer resolution — MS2 consensus spectra typically cannot differentiate
  regio- or stereoisomers
- Feature-Based Molecular Networking with in-source markers needed — this skill targets
  classic MS-Cluster node formation, not FBMN feature detection
edam_operation: http://edamontology.org/operation_3635
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GNPS
  role: Hosts the METABOLOMICS-SNETS-V2 molecular networking workflow in which MS-Cluster
    performs consensus clustering of filtered MS2 spectra
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: MSConvert (ProteoWizard)
  role: Converts instrument-specific raw files to open peak-list formats (.mzXML,
    .mzML) prior to clustering
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41596-020-0317-5
    title: Reproducible molecular networking of untargeted mass spectrometry data
      using GNPS
  source_content_hash: sha256:0c115decc73ed9df6c9b49bd6b2107bfb46fd031c5305f322c42f7161c01ad5f
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
  - GNPS
  - MSConvert (ProteoWizard)
  edam_operation: http://edamontology.org/operation_3635
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# consensus-spectrum-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct the MS-Cluster consensus-clustering step of GNPS molecular networking: collapse nearly identical MS2 spectra into consensus spectra (nodes) after GNPS peak filtering, so downstream network construction and spectral library searching operate on one representative spectrum per cluster.

## When to use

Use when raw MS2 peak lists (.mzXML/.mzML) must be deduplicated into consensus spectra before molecular networking or library search, and the Minimum Cluster Size and precursor mass tolerance need to be justified.

## When NOT to use

- Input is already a feature table or quantified peak area table — consensus clustering operates on MS2 scans only and yields no quantitative information (classic networking is qualitative)
- The goal is isomer resolution — MS2 consensus spectra typically cannot differentiate regio- or stereoisomers
- Feature-Based Molecular Networking with in-source markers needed — this skill targets classic MS-Cluster node formation, not FBMN feature detection

## Inputs

- Peak-list files (.mzXML, .mzML, .mgf) from tandem MS runs, e.g. the MSV000083437 germ-free vs. specific-pathogen-free C57Bl/6J mouse LC-MS/MS dataset
- GNPS job parameters: Minimum Cluster Size, parent-ion mass tolerance, and 'Run MSCluster' flag

## Outputs

- Consensus-cluster table with cluster index, precursor m/z, member-spectrum count, and merged consensus peak list (e.g. MSV000083437_consensus_clusters.tsv)
- Consensus spectra (nodes) that serve as input to network construction and spectral library searching

## How to apply

First apply the GNPS MS2 spectrum filters that precede clustering: delete all peaks within a ±17 Da window around the precursor ion mass, retain only the top six most intense peaks in each ±50 Da window, and delete fragment ions below the minimum peak intensity threshold. Then run MS-Cluster as implemented in the GNPS METABOLOMICS-SNETS-V2 workflow, grouping nearly identical spectra whose precursor ion m/z values fall within the parent-ion mass tolerance. Apply the user-set Minimum Cluster Size filter, discarding clusters with too few member spectra: use 1–2 for one or two files, 5 or more for hundreds of files (a value of 4 was used for the tutorial dataset MSV000083437, with 'Run MSCluster' set to yes for classic molecular networking). Compute a consensus spectrum per retained cluster by merging fragment peaks of its member spectra, and emit a consensus-cluster table (cluster index, precursor m/z, member count, consensus peak list). The rationale is that structurally related molecules share fragment ion patterns, so collapsing replicate spectra reduces redundancy before pairwise spectral-similarity networking.

## Related tools

- **GNPS** (Hosts the METABOLOMICS-SNETS-V2 molecular networking workflow in which MS-Cluster performs consensus clustering of filtered MS2 spectra) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **MSConvert (ProteoWizard)** (Converts instrument-specific raw files to open peak-list formats (.mzXML, .mzML) prior to clustering)

## Evaluation signals

- Consensus-cluster table contains the required columns: cluster index, precursor m/z, member-spectrum count, and consensus peak list
- Cluster counts, precursor m/z values, and member counts are comparable to the 'view all clusters with IDs' output of the corresponding GNPS molecular networking job on MSV000083437
- Every emitted consensus cluster has at least the chosen Minimum Cluster Size member spectra (e.g. 4 for the tutorial dataset)
- Consensus spectra should show removal of peaks in the ±17 Da precursor window and retain at most the top six peaks per ±50 Da window per member scan
- Clusters whose precursor m/z values differ by more than the parent-ion mass tolerance must not be merged

## Limitations

- Classic molecular networking including consensus clustering provides only qualitative information because only MS2 scans are considered; calibrated quantitation requires follow-up targeted LC-MS
- Only ~5–6% of MS2 spectra in untargeted metabolomics end up annotated, so most consensus spectra remain uncharacterized 'dark matter'
- Minimum Cluster Size must scale with dataset size (1–2 for one or two files, 5+ for hundreds of files); a poorly chosen value over- or under-represents clusters

## Evidence

- [intro] Filter Precursor Ion Window: "All peaks in a ±17 Da range around the precursor ion mass are deleted"
- [intro] Filter Peaks in 50 Da Window: "Removes peaks that are not one of the top six most intense within a ±50 Da window"
- [intro] Minimum Cluster Size guidance: [evidence span withheld — anchor status: not_found]
- [intro] Consensus spectra as nodes: "Each consensus spectrum (node) is then queried against spectral library databases to assign it to a putative known molecule within a network"
- [methods] Workflow availability: "Molecular Networking  | [Workflow](https://gnps.ucsd.edu/ProteoSAFe/index.jsp?params=%7B%22workflow%22:%22METABOLOMICS-SNETS-V2%22"
- [intro] Classic networking qualitative only: [evidence span withheld — anchor status: not_found]
