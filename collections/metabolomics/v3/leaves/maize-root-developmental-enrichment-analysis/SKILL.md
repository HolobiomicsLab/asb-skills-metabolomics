---
name: maize-root-developmental-enrichment-analysis
description: Use when DESI-MSI data from maize root sections sampled along a linear
  developmental axis needs to be grouped into enrichment clusters and compared across
  varieties.
when_to_use_negative:
- Data is not from a tissue with a linear developmental axis (DIMPLE is designed specifically
  for linear gradients such as the maize root)
- Input is already a collapsed feature table or enrichment cluster result — this skill
  operates on raw .cdf/.mat MSI workspaces
- Goal is metabolite annotation or database searching (use METASPACE, Metlin Classic,
  or LOTUS instead of the clustering step)
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: DIMPLE
  role: Pipeline that processes mass signatures along the linear developmental gradient
    and clusters them by enrichment pattern
  repo: github.com/dickinsonlab/DIMPLE-code
- name: DESI-MSI
  role: Imaging modality generating the mass-spectrometry-imaging data at ~50–100
    µm spatial resolution that DIMPLE consumes
- name: batchcdfread
  role: Function used to read NetCDF .cdf imaging files into the DIMPLE workspace,
    developed by Yifan Meng (Zare lab)
  repo: github.com/dickinsonlab/DIMPLE-code
- name: METASPACE
  role: Optional plug-in for downstream annotation of unknown mass signatures after
    clustering
- name: Metlin Classic
  role: Alternative annotation resource for mass signatures after enrichment clustering
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:abc38355c0c9c0d76e0ae2d2fd95371e52caf7e36318ab1dba3984b7bc8e4ad8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/dickinsonlab/DIMPLE-code
  tools:
  - DESI-MSI
  - DIMPLE
  - METASPACE
  - Metlin Classic
  - batchcdfread
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# maize-root-developmental-enrichment-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Clusters DESI-MSI mass signatures by developmental enrichment pattern along the linear maize root axis using the DIMPLE pipeline, comparing signature sets across maize varieties (e.g., Oaxacan Green vs. B73).

## When to use

Use when DESI-MSI data from maize root sections sampled along a linear developmental axis needs to be grouped into enrichment clusters and compared across varieties.

## When NOT to use

- Data is not from a tissue with a linear developmental axis (DIMPLE is designed specifically for linear gradients such as the maize root)
- Input is already a collapsed feature table or enrichment cluster result — this skill operates on raw .cdf/.mat MSI workspaces
- Goal is metabolite annotation or database searching (use METASPACE, Metlin Classic, or LOTUS instead of the clustering step)

## Inputs

- DESI-MSI root section data in NetCDF (.cdf) format
- Matlab DIMPLE workspace files (.mat) provided per root replicate
- List of maize root replicates per variety (e.g., B73 root11, B73-root2-5, B73-root-3-3; OG1, OG5, OG2-P11-0)

## Outputs

- Unique pooled mass-signature table with a verified count (e.g., 1,082 for OG)
- Developmental enrichment clusters per variety (e.g., 4 clusters for B73, 5 for Oaxacan Green)
- Cross-variety comparison of cluster enrichment patterns along the root axis

## How to apply

Load the deposited DIMPLE workspace files (e.g., OG1, OG5, and OG2-P11-0 for Oaxacan Green roots at ~50 µm DESI-MSI resolution) from the dickinsonlab/DIMPLE-code repository and run the pipeline on the NetCDF (.cdf) mass-spectrometry-imaging data. Pool mass signatures across replicates and collapse redundant detections to build a unique signature list (e.g., 1,082 unique mass signatures for the OG datasets). Cluster signatures by their enrichment pattern along the root's linear developmental gradient, then compare cluster counts and enrichment patterns across varieties — B73 resolves four clusters while Oaxacan Green typically resolves five, with the first four OG clusters generally consistent with the B73 clusters. Validation requires the pooled signature table to hit the expected unique-signature count and the clustering step to yield the exact cluster count reported in the text.

## Related tools

- **DIMPLE** (Pipeline that processes mass signatures along the linear developmental gradient and clusters them by enrichment pattern) — github.com/dickinsonlab/DIMPLE-code
- **DESI-MSI** (Imaging modality generating the mass-spectrometry-imaging data at ~50–100 µm spatial resolution that DIMPLE consumes)
- **batchcdfread** (Function used to read NetCDF .cdf imaging files into the DIMPLE workspace, developed by Yifan Meng (Zare lab)) — github.com/dickinsonlab/DIMPLE-code
- **METASPACE** (Optional plug-in for downstream annotation of unknown mass signatures after clustering)
- **Metlin Classic** (Alternative annotation resource for mass signatures after enrichment clustering)

## Examples

```
DIMPLE.run(OG1_workspace.mat, OG5_workspace.mat, OG2-P11-0_workspace.mat)  # pool OG replicates and cluster by developmental enrichment
```

## Evaluation signals

- Pooled signature table contains the expected number of unique mass signatures (e.g., 1,082 for Oaxacan Green)
- Clustering yields the exact cluster count reported in the text (5 for OG, 4 for B73)
- OG cluster enrichment patterns are consistent with cluster descriptions in the paper text
- First four OG clusters are generally consistent with the B73 clusters
- Spatial resolution of the input DESI-MSI data is within the ~50–100 µm range

## Limitations

- DESI-MSI spatial resolution (~50–100 µm) is too coarse to resolve metabolite signatures of rare cell types such as stem cells
- A major remaining challenge is that the majority of metabolites remain unannotated, limiting functional interpretation of clusters
- The pipeline is designed specifically for linear developmental axes; non-linear tissue architectures are out of scope

## Evidence
