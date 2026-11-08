---
name: msi-transcriptome-integrated-data-interpretation
description: Use when an integrated spatial transcriptome/MALDI-MSI object (haCCA
  b_predict with X feature matrix and D spatial matrix) is available and the goal
  is to test whether a gene's spatial expression distribution overlaps a specific
  metabolite's intensity distribution.
when_to_use_negative:
- The transcriptome and MALDI-MSI sections have not yet been aligned/integrated by
  haCCA (no b_predict) — interpretability of spot-to-spot overlap is not meaningful
  before integration
- The metabolite of interest has no intensity features represented in the aligned
  MALDI-MSI matrix (no shared features to map onto spots)
- The analysis target is cross-modality integration/alignment itself rather than downstream
  biological interpretation of an already-integrated object
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: haCCA
  role: Produces the integrated b_predict object (MALDI-MSI features aligned onto
    10X Visium spot coordinates) that this interpretation step consumes; via manual_gross_alignment,
    icp_3d_alignment, and direct_alignment
  repo: https://github.com/LittleLittleCloud/haCCA
- name: Scanpy
  role: Spatial plotting of dopamine/3-MT intensity maps over the transcriptome spot
    background and extraction of the Th gene expression vector
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:1cc78fcd1991565be3ab3de34aba2325c9005796d787abeceef30b3bf6a977f3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LittleLittleCloud/haCCA
  repo_url: https://github.com/LittleLittleCloud/haCCA
  tools:
  - Python
  - Scanpy
  - haCCA
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# msi-transcriptome-integrated-data-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interpret haCCA-integrated mouse-brain data (b_predict), in which MALDI-MSI metabolite intensities (e.g. dopamine, 3-MT) are mapped onto 10X Visium transcriptome spot coordinates, by rendering spatial metabolite maps and quantifying co-localization with candidate genes (e.g. Th) to nominate gene–metabolite associations.

## When to use

Use when an integrated spatial transcriptome/MALDI-MSI object (haCCA b_predict with X feature matrix and D spatial matrix) is available and the goal is to test whether a gene's spatial expression distribution overlaps a specific metabolite's intensity distribution.

## When NOT to use

- The transcriptome and MALDI-MSI sections have not yet been aligned/integrated by haCCA (no b_predict) — interpretability of spot-to-spot overlap is not meaningful before integration
- The metabolite of interest has no intensity features represented in the aligned MALDI-MSI matrix (no shared features to map onto spots)
- The analysis target is cross-modality integration/alignment itself rather than downstream biological interpretation of an already-integrated object

## Inputs

- haCCA b_predict Data object (X feature matrix, D spatial coordinate matrix) from integrated 10X Visium + MALDI-MSI mouse-brain data
- a.h5ad and b_prime.h5ad files with .X feature matrices and .obsm['spatial'] coordinates
- Gene expression vector for the candidate gene (e.g. Th) across Visium spots

## Outputs

- Dopamine intensity spatial plot over transcriptome spots (PNG, matching integrating spatial transcriptome into MSI-MALDI_15_0.png)
- 3-MT intensity spatial plot over transcriptome spots (PNG, matching integrating spatial transcriptome into MSI-MALDI_16_0.png)
- Quantified spatial overlap (correlation/co-localization) between Th expression and dopamine/3-MT metabolite distributions

## How to apply

Load the haCCA-integrated b_predict Data object, built per the README as a triplet (X: np.ndarray, D: np.ndarray, Label: Optional[np.ndarray]) with metabolite features aligned onto transcriptome spot coordinates from .h5ad files (a_h5ad.X.toarray(), a_h5ad.obsm['spatial']). Render the dopamine and 3-MT (3-methoxytyramine) intensity maps over the 10X Visium spot background using Scanpy spatial plotting, matching the notebook outputs 'integrating spatial transcriptome into MSI-MALDI_15_0.png' and '_16_0.png'. Extract the Th (tyrosine hydroxylase) expression vector across the same spots from the transcriptome layer, then quantify spatial overlap between Th expression and dopamine/3-MT distributions across spots (e.g. spatial correlation or co-localization of high-intensity regions). Report both PNG figures and the overlap statistic; the expected result is that Th coincides with the dopamine metabolite signal, consistent with Figure 3G of the paper.

## Related tools

- **haCCA** (Produces the integrated b_predict object (MALDI-MSI features aligned onto 10X Visium spot coordinates) that this interpretation step consumes; via manual_gross_alignment, icp_3d_alignment, and direct_alignment) — https://github.com/LittleLittleCloud/haCCA
- **Scanpy** (Spatial plotting of dopamine/3-MT intensity maps over the transcriptome spot background and extraction of the Th gene expression vector)

## Examples

```
a = Data(X=a_h5ad.X.toarray(), D=a_h5ad.obsm['spatial']); b_prime = Data(X=b_prime_h5ad.X.toarray(), D=b_prime_h5ad.obsm['spatial']); b_predict = hacca.direct_alignment(_a, _b_prime)
```

## Evaluation signals

- Regenerated dopamine and 3-MT spatial plots visually match the in-package notebook outputs integrating spatial transcriptome into MSI-MALDI_15_0.png and _16_0.png
- The Th-gene signal spatially coincides with the dopamine metabolite signal, consistent with Figure 3G-H of the paper
- The b_predict object retains the Data triplet structure (X feature matrix, D spatial matrix) with metabolite features on the Visium spot coordinate grid
- Overlap quantification is reported as a per-spot spatial statistic computed over the same spot set used for plotting
- Both spatial plots are saved as PNG figures alongside the numeric overlap result

## Limitations

- Behavior of haCCA on higher-resolution data has not been evaluated, limiting spot-level interpretive claims
- Integration is limited to spatial transcriptome and spatial metabolome modalities; image-based sequencing, spatial proteomics, and histological images are not yet supported
- No data accession or dataset identifier is given for the spatial transcriptome/MSI-MALDI data used in the evaluations, making independent regeneration dependent on the package's example data

## Evidence
