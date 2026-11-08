---
name: spatial-transcriptome-data-splitting
description: Use when you need to benchmark a spatial transcriptome-to-MALDI-MSI integration
  method (e.g. haCCA) against a known ground truth but lack true paired sections,
  by constructing pseudo-paired data from a single spatial transcriptome.
when_to_use_negative:
- When real paired spatial transcriptome and MALDI-MSI sections with shared tissue
  morphology are already available — the ground truth is then directly measured rather
  than simulated.
- When the analysis goal is single-modality spatial transcriptomics analysis (clustering,
  DE) rather than benchmarking cross-modality spot-to-spot integration, as the split+translation
  yields artificial pairs not suited to native tissue interpretation.
- When the data are not spot-resolution spatial transcriptomes (e.g. image-based sequencing,
  spatial proteomics, or histological images) — haCCA's integration and this benchmarking
  scheme are limited to spatial transcriptome and spatial metabolome data.
edam_operation: http://edamontology.org/operation_3804
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0092
tools:
- name: Scanpy
  role: Reads and processes the spatial transcriptome AnnData (.h5ad) that is split
    and coordinate-translated
- name: haCCA
  role: Benchmark target whose performance is evaluated on the pseudo-paired a.h5ad
    / b_prime.h5ad produced by this skill
  repo: github.com/LittleLittleCloud/haCCA
- name: 10X Visium
  role: Source platform providing the spatial transcriptome matrices that are split
    into pseudo-paired sections
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:3b4e3621e14deb9292f48fc8680e68946d9f421def67daebea006e4dc03deaf1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LittleLittleCloud/haCCA
  tools:
  - 10X Visium
  - Python
  - Scanpy
  - haCCA
  edam_operation: http://edamontology.org/operation_3804
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# spatial-transcriptome-data-splitting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Splits the gene/feature set of a spatial transcriptome (AnnData/.h5ad) into two disjoint pseudo-sections and applies Gaussian translation to the spatial coordinates of one, producing pseudo-paired a.h5ad / b_prime.h5ad benchmark inputs for haCCA's spot-to-spot integration evaluation.

## When to use

Use when you need to benchmark a spatial transcriptome-to-MALDI-MSI integration method (e.g. haCCA) against a known ground truth but lack true paired sections, by constructing pseudo-paired data from a single spatial transcriptome.

## When NOT to use

- When real paired spatial transcriptome and MALDI-MSI sections with shared tissue morphology are already available — the ground truth is then directly measured rather than simulated.
- When the analysis goal is single-modality spatial transcriptomics analysis (clustering, DE) rather than benchmarking cross-modality spot-to-spot integration, as the split+translation yields artificial pairs not suited to native tissue interpretation.
- When the data are not spot-resolution spatial transcriptomes (e.g. image-based sequencing, spatial proteomics, or histological images) — haCCA's integration and this benchmarking scheme are limited to spatial transcriptome and spatial metabolome data.

## Inputs

- Spatial transcriptome data as AnnData/.h5ad (e.g. from 10X Visium or BMKMANU S1000) with a feature matrix (X) and spatial coordinates in obsm['spatial']
- Source tissues: mouse brain, rat brain, and human gastric cancer spatial transcriptome matrices

## Outputs

- Paired pseudo-section files a.h5ad and b_prime.h5ad per split (15 pairs per source tissue)
- Translated spatial coordinate matrices suitable for haCCA Data objects (X, D, optional Label)

## How to apply

Load the spatial transcriptome matrix as an AnnData object with Scanpy (sc.read_h5ad), then randomly partition its gene (feature) set into two disjoint subsets — one representing the pseudo-spatial-transcriptome section (a) and the other the pseudo-MALDI-MSI section (b_prime). Apply a Gaussian translation to the spatial coordinates of one section to mimic the morphological disagreement observed between neighboring tissue sections. Write the resulting pair as a.h5ad and b_prime.h5ad; the haCCA README reads these with sc.read_h5ad and builds Data objects via a = Data(X=a_h5ad.X.toarray(), D=a_h5ad.obsm['spatial']). Each pair must retain the same number of spots across both halves. haCCA paper demonstrates 15 pairs per source tissue across 3 mouse brain, 1 rat brain, and 1 human gastric cancer samples.

## Related tools

- **Scanpy** (Reads and processes the spatial transcriptome AnnData (.h5ad) that is split and coordinate-translated)
- **haCCA** (Benchmark target whose performance is evaluated on the pseudo-paired a.h5ad / b_prime.h5ad produced by this skill) — github.com/LittleLittleCloud/haCCA
- **10X Visium** (Source platform providing the spatial transcriptome matrices that are split into pseudo-paired sections)

## Examples

```
a_h5ad = sc.read_h5ad('/path/to/a.h5ad'); b_prime_h5ad = sc.read_h5ad('/path/to/b_prime.h5ad'); a = Data(X=a_h5ad.X.toarray(), D=a_h5ad.obsm['spatial']); b_prime = Data(X=b_prime_h5ad.X.toarray(), D=b_prime_h5ad.obsm['spatial'])
```

## Evaluation signals

- Each generated pair (a.h5ad, b_prime.h5ad) must contain the same number of spots/pixels — validate spot count invariance after feature splitting.
- The two halves must come from disjoint gene subsets: no feature (gene) may appear in both the pseudo-spatial-transcriptome and the pseudo-MALDI-MSI section.
- Spatial coordinates of the translated section must deviate from the original coordinates by an amount consistent with the applied Gaussian perturbation (non-zero, bounded translation).
- Running haCCA on the pseudo-pairs and comparing label transfer accuracy and ARI against the morphological alignment baseline (STUtility) should yield a measurable performance delta (haCCA outperformed STUtility by 6–20% on such data).

## Limitations

- The Gaussian translation is a simplified mimicry of real inter-section morphological disagreement, so pseudo-pair benchmarks may not capture all real-world misalignment sources.
- haCCA's integration — and hence this benchmarking scheme — has only been exercised on spatial transcriptome and spatial metabolome data; other modalities (image-based sequencing, spatial proteomics, histological images) are not yet supported.
- Behavior of haCCA on higher-resolution data has not been evaluated, so pseudo-pairs derived from high-resolution spatial data may not reflect downstream integration quality.

## Evidence

- [methods] For each source tissue, randomly split the feature (gene) set into two disjoint subsets, one representing the pseudo-spatial-transcriptome section and the other the pseudo-MALDI-MSI section.: [evidence span withheld — anchor status: not_found]
- [methods] Apply Gaussian translation to the spatial coordinates of one section to simulate the morphological disagreement observed between neighboring tissue sections.: [evidence span withheld — anchor status: not_found]
