---
name: precursor-mz-tolerance-grouping
description: Use when raw MS2 peak lists (.mzXML/.mzML/.mgf) from multiple LC-MS files
  must be collapsed into consensus spectra to build GNPS molecular network nodes,
  or when choosing a Minimum Cluster Size for a dataset.
when_to_use_negative:
- Input is already a feature table or feature annotations (use Feature-Based Molecular
  Networking instead of classic MS-Cluster clustering)
- Only MS1 scans are available — classic molecular networking considers only MS2 scans
- 'The goal is calibrated quantitation: classic molecular networking provides only
  qualitative information because only MS2 scans are considered'
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: Hosts the METABOLOMICS-SNETS-V2 workflow whose MS-Cluster step performs the
    precursor-m/z-tolerance grouping and consensus spectrum computation
  repo: http://github.com/CCMS-UCSD/GNPS_Workflows
- name: GNPS-MassIVE repository
  role: Source of the public LC-MS dataset files (e.g. MSV000083437) that are downloaded
    as .mzXML before clustering
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41596-020-0317-5
    title: Reproducible molecular networking of untargeted mass spectrometry data
      using GNPS
  source_content_hash: sha256:3fe5761040277531c5f22367bbbc4ab05e3676d21c3fdf388e6f78d00ac98386
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# precursor-mz-tolerance-grouping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Groups nearly identical MS2 spectra by precursor ion m/z tolerance via MS-Cluster in GNPS classic molecular networking, merging them into consensus spectra (nodes) that feed network construction and spectral library search.

## When to use

Use when raw MS2 peak lists (.mzXML/.mzML/.mgf) from multiple LC-MS files must be collapsed into consensus spectra to build GNPS molecular network nodes, or when choosing a Minimum Cluster Size for a dataset.

## When NOT to use

- Input is already a feature table or feature annotations (use Feature-Based Molecular Networking instead of classic MS-Cluster clustering)
- Only MS1 scans are available — classic molecular networking considers only MS2 scans
- The goal is calibrated quantitation: classic molecular networking provides only qualitative information because only MS2 scans are considered

## Inputs

- Peak-list (.mzXML) files for dataset MSV000083437 from the GNPS-MassIVE public repository
- MS2 spectra filtered per GNPS settings (±17 Da precursor window, top 6 peaks per ±50 Da window, minimum peak intensity)
- User parameters: parent-ion mass tolerance, Minimum Cluster Size, Run MSCluster = yes

## Outputs

- Consensus-cluster table with cluster index, precursor m/z, member-spectrum count, and consensus peak list (e.g. MSV000083437_consensus_clusters.tsv)
- Consensus spectra (nodes) input to network construction and spectral library searching

## How to apply

Parse the MS2 spectra and first apply GNPS spectrum filters: delete all peaks in a ±17 Da range around the precursor ion mass, retain only the top six most intense peaks within each ±50 Da window, and delete fragment ions below the minimum peak intensity threshold. Then run MS-Cluster as implemented in the METABOLOMICS-SNETS-V2 workflow, grouping nearly identical MS2 spectra whose precursor ion m/z values lie within the parent-ion mass tolerance. Set 'Run MSCluster' to yes for classic molecular networking and choose the Minimum Cluster Size by dataset scale — 5+ for hundreds of files, 1–2 for one or two files; the tutorial dataset MSV000083437 (germ-free and specific-pathogen-free C57Bl/6J mice) used a value of 4. Discard clusters below the size threshold and compute a consensus spectrum for each retained cluster by merging the fragment peaks of its member spectra.

## Related tools

- **GNPS** (Hosts the METABOLOMICS-SNETS-V2 workflow whose MS-Cluster step performs the precursor-m/z-tolerance grouping and consensus spectrum computation) — http://github.com/CCMS-UCSD/GNPS_Workflows
- **GNPS-MassIVE repository** (Source of the public LC-MS dataset files (e.g. MSV000083437) that are downloaded as .mzXML before clustering)

## Examples

```
https://gnps.ucsd.edu/ProteoSAFe/index.jsp?params=%7B%22workflow%22:%22METABOLOMICS-SNETS-V2%22,%22library_on_server%22:%22d.speclibs;%22%7D
```

## Evaluation signals

- Consensus-cluster table (cluster count, precursor m/z values, member counts) is comparable to the 'view all clusters with IDs' output of the corresponding GNPS molecular networking job on MSV000083437
- Every emitted consensus cluster contains at least Minimum Cluster Size member spectra (4 for the MSV000083437 tutorial)
- Member spectra within a cluster share precursor m/z within the parent-ion mass tolerance
- Filtered spectra retain at most the top six most intense peaks per ±50 Da window and no peaks within ±17 Da of the precursor mass
- Consensus spectra load as nodes in the downstream molecular network and are queryable against GNPS spectral libraries

## Limitations

- Minimum Cluster Size must be scaled to dataset size: 5+ for hundreds of files, 1–2 for one or two files; a mismatched value can under- or over-collapse clusters
- Only ~5–6% of MS2 spectra in untargeted metabolomics are annotated; most consensus spectra remain unannotated 'dark matter'
- MS2 spectra typically cannot differentiate regio- or stereoisomers, limiting node identity resolution
- Classic molecular networking with MS-Cluster provides only qualitative information since only MS2 scans are considered

## Evidence

- [methods] MS-Cluster groups nearly identical MS2 spectra within parent-ion tolerance: [evidence span withheld — anchor status: not_found]
- [supplementary] Minimum Cluster Size setting for the tutorial dataset: [evidence span withheld — anchor status: not_found]
