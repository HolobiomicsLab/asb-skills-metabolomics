---
name: desi-msi-spatial-resolution-handling
description: Use when DESI-MSI imaging data (e.g., .cdf files or Matlab workspace
  files) of tissue sections must be interpreted at ~50 µm pixel resolution along a
  linear developmental gradient, such as maize root longitudinal sections.
when_to_use_negative:
- The dataset was acquired at substantially coarser spatial resolution (>100 µm pixel
  size), so narrow developmental zones along the root axis cannot be resolved
- The tissue geometry is not a linear developmental axis (e.g., 2D organ-wide imaging),
  since DIMPLE is designed to leverage the linear root gradient
- The input is already a processed feature/signature table rather than raw DESI-MSI
  imaging data (cdf files or workspaces)
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3272
tools:
- name: DESI-MSI
  role: Desorption electrospray ionization mass spectrometry imaging modality that
    acquires metabolite maps of maize root sections at ~50–100 µm spatial resolution
- name: DIMPLE
  role: Computational pipeline that processes the ~50 µm DESI-MSI pixel data along
    the linear root axis to generate developmental enrichment clusters
  repo: github.com/dickinsonlab/DIMPLE-code
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:75eef810a69d9b74d4ce5d9d5471ca02a1bc34b25f119d47acb5f0157792ad17
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
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# desi-msi-spatial-resolution-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Handles DESI-MSI datasets acquired at approximately 50–100 µm spatial resolution for developmental mapping in maize root tissue, ensuring that pixel-level mass spectra resolve biologically meaningful gradients along the linear root axis.

## When to use

Use when DESI-MSI imaging data (e.g., .cdf files or Matlab workspace files) of tissue sections must be interpreted at ~50 µm pixel resolution along a linear developmental gradient, such as maize root longitudinal sections.

## When NOT to use

- The dataset was acquired at substantially coarser spatial resolution (>100 µm pixel size), so narrow developmental zones along the root axis cannot be resolved
- The tissue geometry is not a linear developmental axis (e.g., 2D organ-wide imaging), since DIMPLE is designed to leverage the linear root gradient
- The input is already a processed feature/signature table rather than raw DESI-MSI imaging data (cdf files or workspaces)

## Inputs

- DESI-MSI imaging data at ~50 µm spatial resolution (cdf files) for maize root longitudinal sections
- Matlab workspace files accompanying DIMPLE (e.g., OG1-workspace-2.mat, OG5-workspace.mat, B73-root11-workspace.mat)

## Outputs

- Linescans of mass signature intensity along the linear root developmental axis per replicate
- Pooled unique mass signature table (e.g., 1,082 unique mass signatures for Oaxacan Green replicates)
- Developmental enrichment clusters of mass signatures (five OG clusters; four B73 clusters for comparison)

## How to apply

Acquire or load DESI-MSI imaging data at approximately 50–100 µm spatial resolution, which is the resolution at which in situ chemical composition of tissue sections can be resolved for the developing maize root. For the Oaxacan Green replicates, load the deposited DIMPLE workspace files (OG1, OG5, and OG2-P11-0) at ~50 µm resolution, and for B73 replicates load B73-root11, B73-root2-5, and B73-root-3-3. Feed the imaging data into the DIMPLE pipeline, which exploits the linear developmental axis of the root so that each pixel's mass spectrum is positioned along the longitudinal gradient. At ~50 µm pixel size, mass signatures enriched in narrow developmental zones (e.g., meristematic vs. elongation zones) remain separable, enabling downstream clustering of signatures by developmental enrichment pattern. Verify that the pooled signature table from three OG replicates contains 1,082 unique mass signatures after collapsing redundant detections, and that linescans resolve into the expected cluster count (five for OG, four for B73).

## Related tools

- **DESI-MSI** (Desorption electrospray ionization mass spectrometry imaging modality that acquires metabolite maps of maize root sections at ~50–100 µm spatial resolution)
- **DIMPLE** (Computational pipeline that processes the ~50 µm DESI-MSI pixel data along the linear root axis to generate developmental enrichment clusters) — github.com/dickinsonlab/DIMPLE-code

## Evaluation signals

- Spatial resolution of the acquired imaging data is within the ~50–100 µm range stated for DESI-MSI in situ tissue analysis
- Pooled unique mass signature table across the three OG replicates contains exactly 1,082 unique mass signatures after collapsing redundant detections
- Clustering of OG linescans yields exactly five developmental enrichment clusters, consistent with the cluster descriptions reported in the text
- B73 comparison linescans yield four clusters, with the first four OG clusters generally consistent with the B73 clusters
- Each replicate (OG1, OG5, OG2-P11-0; B73-root11, B73-root2-5, B73-root-3-3) is loaded from its corresponding cdf and workspace file pair

## Limitations

- DESI-MSI spatial resolution (~50–100 µm) may still mask metabolic signatures of rare cell types such as stem cells, a limitation noted for bulk and spatial metabolomics approaches
- Most detected mass signatures remain unannotated, since the majority of metabolites in plants and animals are unannotated, complicating functional interpretation at any resolution
- The workspace/cdf file pairing is version-specific to Sama et al. 2025; e.g., OG2-P11-0 is listed without an explicit workspace .mat file in the README

## Evidence
