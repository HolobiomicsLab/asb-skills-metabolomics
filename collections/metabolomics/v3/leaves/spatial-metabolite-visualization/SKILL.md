---
name: spatial-metabolite-visualization
description: Use when haCCA-integrated mouse-brain or ICC data (b_predict, MALDI-MSI
  features aligned onto Visium spot coordinates) is available and you need spatial
  intensity maps and gene–metabolite co-localization quantification.
when_to_use_negative:
- Input data has not yet been integrated by haCCA (no shared spots or features between
  raw spatial transcriptome and MALDI-MSI data), so metabolite intensities cannot
  be placed on transcriptome spot coordinates
- Modalities other than spatial transcriptome and spatial metabolome data (e.g. image-based
  sequencing, spatial proteomics, histological images), which haCCA does not yet support
- Higher-resolution spatial data, whose behavior with haCCA integration has not been
  evaluated in the paper
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3308
tools:
- name: haCCA
  role: Produces b_predict, the integrated Data object in which MALDI-MSI metabolite
    features are aligned onto 10X Visium spot coordinates, via manual_gross_alignment,
    icp_3d_alignment and direct_alignment
  repo: github.com/LittleLittleCloud/haCCA
- name: Scanpy
  role: Standard processing and spatial plotting to render metabolite intensity maps
    over the transcriptome spot background
- name: Python
  role: Language of the easy-to-use haCCA package used to run the integration and
    plotting workflow
  repo: github.com/LittleLittleCloud/haCCA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:5f3b413577ff62f8d288039216625f7ac81959f89b55b754e71f37fc02dfd7eb
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LittleLittleCloud/haCCA
  tools:
  - Python
  - Scanpy
  - haCCA
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# spatial-metabolite-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Render MALDI-MSI metabolite intensity maps (e.g. dopamine, 3-MT) onto 10X Visium transcriptome spot coordinates from haCCA-integrated b_predict data, and quantify spatial co-localization with candidate genes such as Th to link metabolites to genes.

## When to use

Use when haCCA-integrated mouse-brain or ICC data (b_predict, MALDI-MSI features aligned onto Visium spot coordinates) is available and you need spatial intensity maps and gene–metabolite co-localization quantification.

## When NOT to use

- Input data has not yet been integrated by haCCA (no shared spots or features between raw spatial transcriptome and MALDI-MSI data), so metabolite intensities cannot be placed on transcriptome spot coordinates
- Modalities other than spatial transcriptome and spatial metabolome data (e.g. image-based sequencing, spatial proteomics, histological images), which haCCA does not yet support
- Higher-resolution spatial data, whose behavior with haCCA integration has not been evaluated in the paper

## Inputs

- a.h5ad — spatial transcriptome data (10X Visium, BMKMANU S1000) with X feature matrix and obsm['spatial'] coordinates
- b_prime.h5ad — MALDI-MSI metabolome data with peak/intensity feature matrix and spatial coordinates
- b_predict — haCCA-integrated Data object with MALDI-MSI features aligned onto transcriptome spot coordinates
- Target metabolite identities (e.g. dopamine, 3-MT) and target gene expression vector (e.g. Th)

## Outputs

- Spatial intensity PNG plot of the metabolite (e.g. dopamine) over the transcriptome spot background, matching 'integrating spatial transcriptome into MSI-MALDI_15_0.png'
- Spatial intensity PNG plot of the second metabolite (e.g. 3-MT), matching '_16_0.png'
- Quantified spatial overlap (correlation / co-localization of high-intensity regions) between gene expression (Th) and metabolite distributions

## How to apply

First construct Data objects from the .h5ad inputs (X feature matrix, D spatial coordinate matrix from obsm['spatial']) and run the haCCA alignment chain (manual_gross_alignment, icp_3d_alignment, direct_alignment) to obtain b_predict, in which MALDI-MSI metabolite features are aligned onto transcriptome spot coordinates. Then extract the metabolite intensity vectors (e.g. dopamine, 3-MT) and the gene expression vector (e.g. Th, tyrosine hydroxylase) across the same spots and render each as a spatial plot with Scanpy's spatial plotting on the transcriptome spot background, matching the notebook outputs 'integrating spatial transcriptome into MSI-MALDI_15_0.png' and '_16_0.png'. Finally quantify the spatial overlap between the Th expression distribution and the dopamine/3-MT distribution across spots (spatial correlation / co-localization of high-intensity regions). The rationale is that haCCA integration guarantees spot-to-spot correspondence, so overlapping spatial distributions indicate gene–metabolite associations such as Th with dopamine metabolism (paper Fig. 3G).

## Related tools

- **haCCA** (Produces b_predict, the integrated Data object in which MALDI-MSI metabolite features are aligned onto 10X Visium spot coordinates, via manual_gross_alignment, icp_3d_alignment and direct_alignment) — github.com/LittleLittleCloud/haCCA
- **Scanpy** (Standard processing and spatial plotting to render metabolite intensity maps over the transcriptome spot background)
- **Python** (Language of the easy-to-use haCCA package used to run the integration and plotting workflow) — github.com/LittleLittleCloud/haCCA

## Examples

```
from hacca import *; _b_prime = hacca.manual_gross_alignment(a, b_prime); _a, _b_prime = hacca.icp_3d_alignment(a, _b_prime); b_predict = hacca.direct_alignment(_a, _b_prime)
```

## Evaluation signals

- Regenerated metabolite spatial plots visually match the in-package notebook outputs 'integrating spatial transcriptome into MSI-MALDI_15_0.png' and '_16_0.png'
- The Th-gene expression signal coincides with the dopamine metabolite signal across spots, consistent with Figure 3G-H of the paper
- b_predict exists only after the full alignment chain (manual_gross_alignment -> icp_3d_alignment -> direct_alignment) has been applied to (a, b_prime)
- The rendered plots use the transcriptome spot coordinates (D matrix from obsm['spatial']) as the shared spatial background for both metabolite and gene signals
- Overlap quantification (spatial correlation / co-localization of high-intensity regions) is reported numerically, not just visually

## Limitations

- haCCA and this downstream visualization assume neighbor tissue sections; integration quality depends on the alignment methods used
- Behavior of haCCA on higher resolution data has not been evaluated
- Only spatial transcriptome and spatial metabolome data have been used; other modalities are not supported
- No data accession or dataset identifier is given for the mouse-brain spatial transcriptome / MSI-MALDI cohort in the discussion

## Evidence
