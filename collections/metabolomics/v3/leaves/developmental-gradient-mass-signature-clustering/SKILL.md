---
name: developmental-gradient-mass-signature-clustering
description: Use when you have DESI-MSI imaging data of a root or organ with a linear
  developmental axis, a pool of redundant per-replicate mass signature detections
  (e.
when_to_use_negative:
- When the input is not imaging data along a linear/positional axis (e.g. bulk metabolomics
  with no spatial dimension) — DIMPLE's linear-evaluation assumption fails and clustering
  would be meaningless.
- When the data has already been collapsed to a non-spatial feature table with no
  per-pixel or per-position information, since the enrichment pattern along the axis
  is what drives clustering.
- When the goal is metabolite identification/annotation rather than developmental
  pattern discovery — this skill does not annotate m/z values; use METASPACE, Metlin
  Classic, or LOTUS for that.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: DIMPLE
  role: Pipeline that processes mass signatures along linear gradients and generates
    clusters of metabolites with specific developmental enrichment patterns
  repo: github.com/dickinsonlab/DIMPLE-code
- name: DESI-MSI
  role: Imaging modality that produces the ~50 µm spatial-resolution mass-spectrometry
    imaging data whose m/z signatures are pooled and clustered
- name: batchcdfread
  role: MATLAB function developed by Yifan Meng (Zare lab) for batch-reading the CDF
    imaging files into the DIMPLE pipeline
  repo: github.com/dickinsonlab/DIMPLE-code
- name: MSiReader v1.03
  role: Alternative/complementary MSI data analysis software mentioned for viewing
    and optimizing mass signatures prior to or in parallel with DIMPLE clustering
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:1e2df272a5f3ed4982edb850f0aaa4a17806a48278d362abe2769708d7532ab8
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
  - MSiReader v1.03
  - batchcdfread
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# developmental-gradient-mass-signature-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A clustering skill in which mass signatures detected along a linear developmental axis (e.g. the maize root tip-to-mature gradient imaged by DESI-MSI at ~50–100 µm) are pooled across replicates, reduced to unique signatures, and grouped by their longitudinal enrichment pattern so that developmental transitions can be compared across varieties or conditions.

## When to use

Use when you have DESI-MSI imaging data of a root or organ with a linear developmental axis, a pool of redundant per-replicate mass signature detections (e.g. ~1,000+ unique m/z after collapsing), and you need to group signatures into clusters of shared enrichment pattern to compare conditions such as salt-resilient vs salt-susceptible varieties.

## When NOT to use

- When the input is not imaging data along a linear/positional axis (e.g. bulk metabolomics with no spatial dimension) — DIMPLE's linear-evaluation assumption fails and clustering would be meaningless.
- When the data has already been collapsed to a non-spatial feature table with no per-pixel or per-position information, since the enrichment pattern along the axis is what drives clustering.
- When the goal is metabolite identification/annotation rather than developmental pattern discovery — this skill does not annotate m/z values; use METASPACE, Metlin Classic, or LOTUS for that.

## Inputs

- NetCDF/CDF mass spectrometry imaging files from DESI-MSI of root sections (~50 µm spatial resolution), one per replicate root (e.g. B73-root11, B73-root2-5, B73-root-3-3, OG1, OG5, OG2-P11-0)
- Pre-computed MATLAB workspace .mat files pairing each CDF image with its acquisition metadata (e.g. OG1-workspace-2.mat, OG5-workspace.mat)
- List of per-replicate mass signature detections along the root longitudinal axis
- Optional: unique mass-signature count expectation (e.g. 1,082 for pooled OG) as a validation checkpoint

## Outputs

- A pooled table of unique mass signatures (e.g. 1,082 for Oaxacan Green) with per-signature developmental enrichment profiles
- A cluster assignment for each mass signature grouping signatures by shared longitudinal enrichment pattern (e.g. five clusters for OG, four for B73)
- A per-cluster enrichment-pattern description suitable for cross-variety comparison
- Differential mass signatures and enrichment patterns between conditions (e.g. Oaxacan Green vs B73)

## How to apply

Load the raw DESI-MSI data (NetCDF/CDF files, e.g. OG1, OG5, OG2-P11-0 for Oaxacan Green replicates at ~50 µm spatial resolution) into MATLAB using the deposited workspaces (e.g. OG1-workspace-2.mat, OG5-workspace.mat), which pair each CDF image with its imaging geometry. Run DIMPLE (github.com/dickinsonlab/DIMPLE-code), the version described in Sama et al. 2025, to extract and process mass signatures along the linear developmental gradient of each root section, using the batchcdfread function (credited to Yifan Meng, Zare lab) for reading CDF files. Pool the per-replicate mass signature detections across datasets and collapse redundant detections to build a unique signature list — for the Oaxacan Green roots this yields 1,082 unique mass signatures. Cluster the signatures by their developmental enrichment pattern along the root axis, choosing the number of clusters that matches the described enrichment-pattern groups in the source article's text (e.g. five clusters for OG linescans vs four for B73) rather than an arbitrary k. Validate by checking that the pooled table contains the expected number of unique signatures and that each cluster's enrichment pattern (e.g. tip-enriched, maturation-enriched, absent-in-tip) is consistent with the descriptive cluster language in the paper, then compare cluster composition across varieties (B73 vs Oaxacan Green) to identify condition-specific differences.

## Related tools

- **DIMPLE** (Pipeline that processes mass signatures along linear gradients and generates clusters of metabolites with specific developmental enrichment patterns) — github.com/dickinsonlab/DIMPLE-code
- **DESI-MSI** (Imaging modality that produces the ~50 µm spatial-resolution mass-spectrometry imaging data whose m/z signatures are pooled and clustered)
- **batchcdfread** (MATLAB function developed by Yifan Meng (Zare lab) for batch-reading the CDF imaging files into the DIMPLE pipeline) — github.com/dickinsonlab/DIMPLE-code
- **MSiReader v1.03** (Alternative/complementary MSI data analysis software mentioned for viewing and optimizing mass signatures prior to or in parallel with DIMPLE clustering)

## Examples

```
run DIMPLE_pipeline.m on OG1/OG5/OG2-P11-0 CDFs loaded via batchcdfread, pool detections, collapse to unique m/z, then cluster by developmental enrichment pattern
```

## Evaluation signals

- The pooled unique mass-signature count matches the expected value (e.g. exactly 1,082 for the three Oaxacan Green replicates OG1, OG5, OG2-P11-0).
- The number of enrichment-pattern clusters matches the described groupings (five clusters for OG linescans; four clusters for B73 linescans).
- Each resulting cluster's enrichment pattern (e.g. tip-enriched vs mature-zone-enriched) is consistent with the corresponding cluster description in the paper's text for that variety.
- Cluster membership of key marker metabolites (e.g. D-erythrose) is consistent with the paper's localization claims (different localization patterns between Oaxacan Green and B73).
- Cross-variety comparison yields distinct differential mass signatures and enrichment patterns between Oaxacan Green and B73, not a single merged cluster set.

## Limitations

- The pipeline assumes a linear developmental axis; organs without a clear linear gradient are outside its scope.
- Untargeted MSI produces hundreds to thousands of peaks per pixel, so clustering operates on unannotated mass signatures — most remain unannotated and require downstream database lookups (METASPACE, Metlin Classic, LOTUS).
- Results depend on the deposited workspaces/parameter choices from the specific Sama et al. 2025 version of DIMPLE; other imaging geometries or resolutions (e.g. 100 µm instead of ~50 µm) may change the number and composition of clusters.
- Bulk-scale metabolomics is too coarse to capture the rare-cell-type signatures this skill is designed to expose; using bulk data defeats the purpose of the linear-axis clustering.

## Evidence

- [intro] Streamline untargeted MSI analysis by developing a tool specifically for interpretation of plant root MSI data.: [evidence span withheld — anchor status: not_found]
