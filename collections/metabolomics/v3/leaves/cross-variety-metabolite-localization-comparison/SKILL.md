---
name: cross-variety-metabolite-localization-comparison
description: Use when two DESI-MSI developmental-gradient datasets (e.g. B73 vs Oaxacan
  Green roots) already have DIMPLE cluster assignments and the goal is to determine
  which mass signatures from a source-variety cluster (e.
when_to_use_negative:
- Input is bulk metabolomics data lacking spatial coordinates — the comparison depends
  on developmental-zone localization from DESI-MSI
- The datasets have not been processed by DIMPLE, so no developmental enrichment clusters
  exist to map between varieties
- The biological axis is not linear (e.g. 2D tissues without a developmental gradient),
  which DIMPLE's linear-evaluation design does not support
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: DIMPLE
  role: Generates the per-variety cluster assignments and clustered linescans along
    the linear root developmental gradient that are cross-referenced for conserved
    vs divergent localization
  repo: github.com/dickinsonlab/DIMPLE-code
- name: DESI-MSI
  role: Provides the spatial metabolite imaging data (≈50–100 µm resolution) of maize
    root sections from which mass signatures and their positional enrichment are derived
- name: batchcdfread
  role: MATLAB function (developed by Yifan Meng, Zare lab) used to read the .cdf
    imaging files into the workspaces used for cross-variety cluster extraction
  repo: github.com/dickinsonlab/DIMPLE-code
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:c5433c99ae40f6cfc3f45b0cd6488a85fa5221fbaa2093774f1c8e57acb77385
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
  - batchcdfread
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Cross-variety metabolite localization comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compares DIMPLE cluster assignments for mass signatures between two maize root DESI-MSI datasets (e.g. B73 and Oaxacan Green) to classify each signature as developmentally conserved or divergent, producing a conservation/divergence table of m/z, cluster membership per variety, and localization status.

## When to use

Use when two DESI-MSI developmental-gradient datasets (e.g. B73 vs Oaxacan Green roots) already have DIMPLE cluster assignments and the goal is to determine which mass signatures from a source-variety cluster (e.g. Cluster 1, meristem-localized) are conserved, and which conserved ones shift localization.

## When NOT to use

- Input is bulk metabolomics data lacking spatial coordinates — the comparison depends on developmental-zone localization from DESI-MSI
- The datasets have not been processed by DIMPLE, so no developmental enrichment clusters exist to map between varieties
- The biological axis is not linear (e.g. 2D tissues without a developmental gradient), which DIMPLE's linear-evaluation design does not support

## Inputs

- DIMPLE cluster assignments for each variety (B73 and Oaxacan Green root developmental gradients)
- Clustered linescans of mass signatures along the root longitudinal axis
- DESI-MSI imaging data in .cdf format (e.g. B73-root11, B73-root2-5, OG1, OG5) with paired MATLAB workspaces (.mat)

## Outputs

- Conservation/divergence table listing each signature, its B73 cluster, its Oaxacan Green cluster, and its conservation status
- Explicit record of divergent metabolites with variety-specific localization patterns (e.g. m/z 149.0118, m/z 280.2367)
- Count of conserved signatures (e.g. 20 of 48 B73 Cluster 1 signatures conserved in Oaxacan Green)

## How to apply

Load the DIMPLE cluster assignments and clustered linescans previously generated for both varieties' root developmental gradients (NetCDF/cdf imaging files with their MATLAB workspaces, e.g. B73-root11-workspace.mat and OG1-workspace-2.mat). Extract the mass signatures of the source-variety cluster of interest — here the 48 signatures in B73 Cluster 1, enriched at the meristem. For each m/z, locate the corresponding signature in the Oaxacan Green clusters and classify its enrichment pattern as conserved (same developmental zone enrichment) or divergent (e.g. m/z 149.0118 root-tip specific in B73 but broader Cluster 5 localization in Oaxacan Green; m/z 280.2367 root-tip enriched in B73 but strongly EZ-cortex localized in Oaxacan Green). Compile a table with columns for signature m/z, source-variety cluster, target-variety cluster, and conservation status. The rationale is that DIMPLE leverages the linear developmental axis of the root, so cluster identity encodes positional enrichment and cross-variety cluster mapping reveals developmental rewiring of metabolism.

## Related tools

- **DIMPLE** (Generates the per-variety cluster assignments and clustered linescans along the linear root developmental gradient that are cross-referenced for conserved vs divergent localization) — github.com/dickinsonlab/DIMPLE-code
- **DESI-MSI** (Provides the spatial metabolite imaging data (≈50–100 µm resolution) of maize root sections from which mass signatures and their positional enrichment are derived)
- **batchcdfread** (MATLAB function (developed by Yifan Meng, Zare lab) used to read the .cdf imaging files into the workspaces used for cross-variety cluster extraction) — github.com/dickinsonlab/DIMPLE-code

## Examples

```
load('B73-root11-workspace.mat'); % and OG1-workspace-2.mat; then extract B73 Cluster 1 m/z list and map each m/z to Oaxacan Green cluster IDs
```

## Evaluation signals

- The conservation table reports 20 of the 48 B73 Cluster 1 signatures as conserved in Oaxacan Green
- The table contains rows for m/z 149.0118 and m/z 280.2367 with their variety-specific localization patterns (root-tip B73 vs broader/EZ-cortex Oaxacan Green)
- Every B73 Cluster 1 signature has a classified corresponding m/z in an Oaxacan Green cluster (conserved or divergent), with no unassigned rows
- Cluster labels referenced (e.g. Cluster 1 meristem-localized, Cluster 5 broader localization) match the DIMPLE assignments from the source workspaces

## Limitations

- Most mass signatures remain unannotated, so divergence is detected at the m/z level without metabolite identity for many signatures
- DESI-MSI spatial resolution (≈50–100 µm) limits how precisely a shifted localization (e.g. root tip vs EZ cortex) can be resolved
- Conservation of an m/z does not guarantee conservation of localization — about half of the shared signatures lose meristem-specific enrichment in Oaxacan Green, so m/z-level matching alone would overstate conservation

## Evidence
