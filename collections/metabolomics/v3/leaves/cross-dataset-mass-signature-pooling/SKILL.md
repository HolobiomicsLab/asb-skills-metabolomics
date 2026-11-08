---
name: cross-dataset-mass-signature-pooling
description: Use when multiple replicate DESI-MSI datasets of the same tissue type
  (e.g., root sections) have each been processed along a linear developmental gradient
  and you need a pooled, de-replicated mass signature list for cross-sample clustering
  and variety comparison.
when_to_use_negative:
- The datasets are not organized along a linear developmental axis — DIMPLE was designed
  specifically to leverage the linear developmental axis of the maize root and does
  not apply to arbitrary 2D tissue geometry.
- You need metabolite annotation/identification rather than pattern discovery — the
  article notes the majority of mass signatures remain unannotated and recommends
  dedicated database searching (METASPACE, Metlin Classic, LOTUS).
- You have only a single replicate and no cross-dataset de-replication or variety
  comparison is needed.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: DIMPLE
  role: Computational pipeline (Developmental Imaging Mass Spectrometry Pipeline for
    Linear Evaluation) that processes mass signatures along the linear root developmental
    gradient and generates enrichment clusters; pooled signature counts and cluster
    numbers are validated against DIMPLE outputs
  repo: github.com/dickinsonlab/DIMPLE-code
- name: DESI-MSI
  role: Imaging modality that produces the metabolite maps of the developing maize
    root at ~50-100 µm spatial resolution; supplies the per-pixel mass spectra that
    DIMPLE consumes
- name: batchcdfread
  role: Matlab function (developed by Yifan Meng, Zare lab, Stanford) for batch-reading
    the cdf files of multiple root datasets into the pipeline
  repo: github.com/dickinsonlab/DIMPLE-code
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:e18c3a3d697eb3a81025b55137380ed8bca0b2848fe77344b954057aff59acbf
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# cross-dataset-mass-signature-pooling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pool mass signatures detected by DIMPLE across replicate DESI-MSI root datasets (e.g., Oaxacan Green OG1, OG5, OG2-P11-0 at ~50 µm spatial resolution), collapse redundant detections into a unique signature list, and cluster signatures by developmental enrichment pattern along the linear root axis.

## When to use

Use when multiple replicate DESI-MSI datasets of the same tissue type (e.g., root sections) have each been processed along a linear developmental gradient and you need a pooled, de-replicated mass signature list for cross-sample clustering and variety comparison.

## When NOT to use

- The datasets are not organized along a linear developmental axis — DIMPLE was designed specifically to leverage the linear developmental axis of the maize root and does not apply to arbitrary 2D tissue geometry.
- You need metabolite annotation/identification rather than pattern discovery — the article notes the majority of mass signatures remain unannotated and recommends dedicated database searching (METASPACE, Metlin Classic, LOTUS).
- You have only a single replicate and no cross-dataset de-replication or variety comparison is needed.

## Inputs

- DIMPLE cdf files and Matlab workspace files for each replicate root (e.g., OG1, OG5, OG2-P11-0 for Oaxacan Green; B73-root11, B73-root2-5, B73-root-3-3 for B73)
- DESI-MSI imaging data of maize root sections at ~50 µm spatial resolution

## Outputs

- Pooled table of unique mass signatures collapsed across replicates (e.g., 1,082 unique signatures for the three Oaxacan Green roots)
- Developmental enrichment clusters of mass signatures along the root axis (e.g., five OG clusters; four B73 clusters)
- Cross-variety comparison of cluster enrichment patterns (Oaxacan Green vs. B73)

## How to apply

Load the deposited DIMPLE workspace files for each replicate (e.g., OG1/OG1-workspace-2.mat, OG5/OG5-workspace.mat, OG2-P11-0 for the Oaxacan Green roots), run the DIMPLE pipeline (github.com/dickinsonlab/DIMPLE-code) to process mass signatures along each root's linear developmental gradient, then pool mass signatures across all replicate datasets and collapse redundant detections into a single unique signature list. For the Sama et al. 2025 Oaxacan Green data, the pooled table must contain 1,082 unique mass signatures. Cluster the pooled signatures by their developmental enrichment pattern along the root axis, then compare each cluster's pattern against the enrichment descriptions reported in the paper text (five OG clusters versus the four observed in B73). The rationale is that individual replicate linescans are noisy, so pooling across replicates reveals variety-level enrichment patterns that distinguish salt-resilient (Oaxacan Green) from salt-susceptible (B73) maize.

## Related tools

- **DIMPLE** (Computational pipeline (Developmental Imaging Mass Spectrometry Pipeline for Linear Evaluation) that processes mass signatures along the linear root developmental gradient and generates enrichment clusters; pooled signature counts and cluster numbers are validated against DIMPLE outputs) — github.com/dickinsonlab/DIMPLE-code
- **DESI-MSI** (Imaging modality that produces the metabolite maps of the developing maize root at ~50-100 µm spatial resolution; supplies the per-pixel mass spectra that DIMPLE consumes)
- **batchcdfread** (Matlab function (developed by Yifan Meng, Zare lab, Stanford) for batch-reading the cdf files of multiple root datasets into the pipeline) — github.com/dickinsonlab/DIMPLE-code

## Examples

```
batchcdfread({'OG1','OG5','OG2-P11-0'}); % read replicate DIMPLE cdf datasets, then pool and cluster signatures per DIMPLE-code
```

## Evaluation signals

- Pooled signature table contains exactly 1,082 unique mass signatures after collapsing redundant detections across OG1, OG5, and OG2-P11-0.
- Clustering step yields exactly five Oaxacan Green cluster groups (not the four observed in B73).
- The first four OG clusters show enrichment patterns generally consistent with the corresponding B73 clusters.
- Each cluster's enrichment pattern matches the cluster descriptions given in the paper text for Oaxacan Green.
- Each replicate's workspace file (e.g., OG5-workspace.mat) pairs correctly with its cdf data file per the README mapping.

## Limitations

- The majority of mass signatures remain unannotated, so clusters are pattern-level (m/z enrichment profiles), not metabolite-level, conclusions.
- Cluster groupings are variety-dependent: Oaxacan Green resolves into five clusters while B73 resolves into four, so cluster count should not be assumed transferable across varieties.
- The pipeline leverages the linear developmental axis of the root; it was designed for and validated on these maize root datasets (B73 and Oaxacan Green) as described in Sama et al. 2025.

## Evidence
