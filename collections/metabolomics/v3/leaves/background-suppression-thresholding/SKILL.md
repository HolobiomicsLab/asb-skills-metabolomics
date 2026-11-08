---
name: background-suppression-thresholding
description: Use when filtering raw untargeted DESI-MSI pixel-level mass spectra (e.g.,
  maize root imaging workspaces) down to a defensible list of biological m/z signatures
  before downstream annotation or gradient clustering.
when_to_use_negative:
- The m/z list is already curated/annotated (e.g., validated Zhang et al. 2023 peaks)
  — thresholding would discard genuine low-abundance signatures
- The analysis goal is developmental clustering of mass signatures along the root
  axis (DIMPLE clustering) rather than denoising the peak list
- The dataset has no background/reference intensity measurement, making the 1.5-fold
  ratio undefined
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: DIMPLE
  role: Pipeline in which the 1.5-fold background suppression and sparsity filtering
    are applied to consensus peaks from pseudobulk spectra
  repo: github.com/dickinsonlab/DIMPLE-code
- name: DESI-MSI
  role: Imaging modality generating the pixel-level mass spectra that the thresholding
    filters down to biological signatures
- name: batchcdfread
  role: Matlab function (Yifan Meng, Zare lab, Stanford) used to read the deposited
    cdf files into workspaces prior to filtering
  repo: github.com/dickinsonlab/DIMPLE-code
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:d54240ff1dc9f91538dfe5be370324c203a8b5160ebff4f8fc2eebdebe076362
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
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# Background-suppression thresholding for MSI peak filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A filtering step in the DIMPLE pipeline that suppresses non-biological signal in DESI-MSI data by retaining only m/z consensus peaks whose pseudobulk intensity exceeds 1.5-fold the background level, paired with sparsity filtering to drop signatures present in only a small fraction of pixels.

## When to use

Use when filtering raw untargeted DESI-MSI pixel-level mass spectra (e.g., maize root imaging workspaces) down to a defensible list of biological m/z signatures before downstream annotation or gradient clustering.

## When NOT to use

- The m/z list is already curated/annotated (e.g., validated Zhang et al. 2023 peaks) — thresholding would discard genuine low-abundance signatures
- The analysis goal is developmental clustering of mass signatures along the root axis (DIMPLE clustering) rather than denoising the peak list
- The dataset has no background/reference intensity measurement, making the 1.5-fold ratio undefined

## Inputs

- Matlab workspace files from DESI-MSI imaging runs (e.g., B73-root11-workspace.mat, B73-root2-5-workspace.mat, B73-root2-3-worskpace.mat)
- Raw NetCDF/cdf mass spectrometry files (e.g., B73-root11, B73-root2-5, B73-root-3-3)
- Pseudobulk mass spectra (per-m/z-bin intensity sums across all pixels)
- Consensus peak list extracted with 2.5 ppm mass tolerance
- Background intensity measurement per m/z bin

## Outputs

- Per-root filtered m/z signature list (CSV)
- Pooled unique m/z signature list across replicates (e.g., 301 unique B73 signatures)
- Set of m/z peaks surviving the 1.5-fold background threshold and sparsity filter

## How to apply

First build a pseudobulk mass spectrum per root by summing each m/z bin's intensity across all pixels, then extract consensus peaks with a 2.5 ppm mass tolerance. Apply background suppression by retaining only peaks whose intensity exceeds 1.5-fold the background level — this removes chemically meaningless signal that dominates the hundreds to thousands of peaks per pixel in untargeted MSI. Follow with sparsity filtering to remove m/z signatures present in only a small fraction of pixels, since sparse detections are unlikely to represent genuine tissue-localized metabolites. The surviving m/z values constitute the per-root mass signature list, which can be pooled across the three B73 replicates (B73-root11, B73-root2-5, B73-root-3-3) to count unique signatures. The rationale is that untargeted DESI-MSI spectra are too large for direct annotation, so intensity-ratio and sparsity cutoffs are the primary noise controls before tools like METASPACE, Metlin Classic, or LOTUS are consulted.

## Related tools

- **DIMPLE** (Pipeline in which the 1.5-fold background suppression and sparsity filtering are applied to consensus peaks from pseudobulk spectra) — github.com/dickinsonlab/DIMPLE-code
- **DESI-MSI** (Imaging modality generating the pixel-level mass spectra that the thresholding filters down to biological signatures)
- **batchcdfread** (Matlab function (Yifan Meng, Zare lab, Stanford) used to read the deposited cdf files into workspaces prior to filtering) — github.com/dickinsonlab/DIMPLE-code

## Examples

```
load('B73-root11-workspace.mat'); % pseudobulk sum over pixels, consensus peaks at 2.5 ppm, keep peaks with intensity > 1.5x background and passing sparsity filter
```

## Evaluation signals

- Recovery of previously published peaks: filtering should retain ≥97% of the 63 manually reported Zhang et al. 2023 peaks
- Per-root yield in the expected range: ~196 m/z signatures per B73 root after suppression and sparsity filtering
- Pooled replicate output matches ~301 unique B73 signatures across the three roots
- Every retained peak's intensity exceeds 1.5-fold its background level (spot-check a sample of peaks against the background spectrum)
- No retained signature is present in only a small fraction of pixels (sparsity check passes for all surviving m/z values)

## Limitations

- The two missed Zhang et al. 2023 signatures were at very low abundance or sparse localization, so the 1.5-fold background and sparsity cutoffs can discard genuine but weak or localized metabolites
- Untargeted MSI spectra contain hundreds to thousands of peaks per pixel, so threshold choices strongly determine the final signature count
- Cutoffs (1.5-fold background, 2.5 ppm tolerance, sparsity fraction) were tuned for maize root DESI-MSI at ~50-100 µm spatial resolution and may need re-optimization for other tissues or modalities (MALDI, SIMS)

## Evidence

- [other] retaining only peaks whose intensity exceeds 1.5-fold the background level, and then apply sparsity filtering to remove m/z signatures present in only a small fraction of pixels: [evidence span withheld — anchor status: not_found]
- [other] Extract consensus peaks from each pseudobulk spectrum with a 2.5 ppm mass tolerance.: [evidence span withheld — anchor status: not_found]
- [intro] MSI datasets are large; each pixel contains a full mass spectrum with hundreds to thousands of peaks.: "MSI datasets are large; each pixel contains a full mass spectrum with hundreds to thousands of peaks. This sheer volume of data presents a challenge, especially when conducting an untargeted analysis"
- [readme] batchcdfread function developed by Yifan Meng in Dr. Richard N. Zare's lab at Stanford University: "batchcdfread function developed by Yifan Meng in Dr. Richard N. Zare's lab at Stanford University"
