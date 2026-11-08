---
name: mass-spectrometry-imaging-data-processing
description: Use when you have DESI-MSI imzML/NetCDF (cdf) pixel spectra from elongating
  root sections arranged along a linear developmental gradient and need to identify
  metabolite mass signatures and group them by developmental enrichment pattern.
when_to_use_negative:
- Input is bulk metabolomics extract data or MS/MS spectra rather than spatially resolved
  MSI pixel spectra along a linear tissue axis
- Tissue geometry is not approximately linear (e.g. 2D organs or non-elongating sections),
  since DIMPLE's enrichment assignment assumes a tip-to-base gradient
- The goal is targeted quantification of known metabolites rather than untargeted
  detection of developmental mass signatures
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: DESI-MSI
  role: Source modality producing ~50–100 µm spatial-resolution mass-spectrometry
    imaging pixel spectra of root sections that feed the pipeline
- name: DIMPLE
  role: Computational pipeline (MATLAB code) that processes pixel spectra along the
    linear developmental axis, collapses redundant mass-signature detections, and
    clusters signatures by enrichment pattern
  repo: github.com/dickinsonlab/DIMPLE-code
- name: batchcdfread
  role: MATLAB helper function (developed by Yifan Meng in Dr. Richard N. Zare's lab
    at Stanford) that reads the .cdf pixel-spectra files into the pipeline
  repo: github.com/dickinsonlab/DIMPLE-code
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:fe77433af5f732b2dd3827abfd2d6b4d65beb1de8766f8f93afcdbd26c86fb82
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
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# mass-spectrometry-imaging-data-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Processes DESI-MSI datasets of developing maize roots through the DIMPLE pipeline to extract mass signatures, collapse them into a unique-signature table, and cluster their enrichment patterns along the linear developmental axis of the root.

## When to use

Use when you have DESI-MSI imzML/NetCDF (cdf) pixel spectra from elongating root sections arranged along a linear developmental gradient and need to identify metabolite mass signatures and group them by developmental enrichment pattern.

## When NOT to use

- Input is bulk metabolomics extract data or MS/MS spectra rather than spatially resolved MSI pixel spectra along a linear tissue axis
- Tissue geometry is not approximately linear (e.g. 2D organs or non-elongating sections), since DIMPLE's enrichment assignment assumes a tip-to-base gradient
- The goal is targeted quantification of known metabolites rather than untargeted detection of developmental mass signatures

## Inputs

- NetCDF (.cdf) files of DESI-MSI pixel spectra per root section (e.g. B73-root11, OG1, OG5, OG2-P11-0)
- MATLAB workspace files (.mat, e.g. OG1-workspace-2.mat) carrying per-pixel spatial coordinates and metadata
- Root sections imaged at ~50–100 µm spatial resolution along the linear developmental gradient

## Outputs

- Per-root linescans of mass-signature abundance along the developmental axis
- Pooled table of unique mass signatures (1,082 for pooled Oaxacan Green roots) with developmental enrichment profiles
- Clusters of mass signatures grouped by shared developmental enrichment pattern (4 clusters for B73, 5 for Oaxacan Green)

## How to apply

Load each root's imaged dataset with the batchcdfread function on the deposited .cdf files, using the accompanying MATLAB workspace (.mat) files to keep per-pixel spatial coordinates aligned with the root's tip-to-base axis. For each replicate root (e.g. B73-root11, B73-root2-5, B73-root-3-3, OG1, OG5, OG2-P11-0) run DIMPLE to extract pixel-level peaks and map mass signatures along the linear axis, then pool signatures across replicates and collapse redundant detections into a unique mass-signature table (1,082 unique signatures for the pooled Oaxacan Green roots). Finally, cluster signatures by their developmental enrichment profile along the root axis — B73 linescans resolve into four clusters and Oaxacan Green into five, with the first four clusters consistent between varieties — and judge correctness by reproducing these signature counts and cluster counts.

## Related tools

- **DESI-MSI** (Source modality producing ~50–100 µm spatial-resolution mass-spectrometry imaging pixel spectra of root sections that feed the pipeline)
- **DIMPLE** (Computational pipeline (MATLAB code) that processes pixel spectra along the linear developmental axis, collapses redundant mass-signature detections, and clusters signatures by enrichment pattern) — github.com/dickinsonlab/DIMPLE-code
- **batchcdfread** (MATLAB helper function (developed by Yifan Meng in Dr. Richard N. Zare's lab at Stanford) that reads the .cdf pixel-spectra files into the pipeline) — github.com/dickinsonlab/DIMPLE-code

## Examples

```
matlab> workspace = load('OG1-workspace-2.mat'); raw = batchcdfread('OG1.cdf'); dimple_run(raw, workspace); % process OG root 1 linescan and extract mass signatures
```

## Evaluation signals

- Pooled mass-signature table for the three Oaxacan Green replicates (OG1, OG5, OG2-P11-0) contains exactly 1,082 unique mass signatures
- Clustering step resolves the Oaxacan Green linescans into exactly five cluster groups matching the enrichment patterns described in the paper text
- B73 root replicates (B73-root11, B73-root2-5, B73-root-3-3) resolve into four enrichment clusters, and the first four OG clusters are consistent with these B73 clusters
- Per-pixel mass spectra and spatial coordinates remain correctly paired when workspace files and cdf files are loaded together
- Cluster membership in the workspace .mat files matches the cluster descriptions reported in the article text

## Limitations

- Majority of detected mass signatures remain unannotated; downstream annotation depends on external resources such as METASPACE, Metlin Classic, or LOTUS
- MSI datasets are large — each pixel contains a full mass spectrum with hundreds to thousands of peaks — which complicates untargeted analysis
- Analysis requires datasets where the developmental axis is linear; the pipeline was designed specifically to leverage the linear developmental axis of the root
- Unannotated signatures cannot be assumed to have one-to-one metabolite identity without database searching or follow-up validation

## Evidence

- [results] 1,082 unique mass signatures: [evidence span withheld — anchor status: not_found]
