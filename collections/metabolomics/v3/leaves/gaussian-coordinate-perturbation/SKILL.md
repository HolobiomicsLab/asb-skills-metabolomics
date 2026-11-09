---
name: gaussian-coordinate-perturbation
description: Use when generating pseudo-paired spatial transcriptome / MALDI-MSI benchmark
  pairs from a single source tissue — e.g. to stress-test coordinate-alignment methods
  (manual gross, ICP 3D, direct alignment) against realistic inter-section morphology
  drift.
when_to_use_negative:
- You have real paired spatial-transcriptome/MALDI-MSI sections with actual morphological
  disagreement — no synthetic perturbation is needed.
- Your downstream method does not use spatial coordinates (D matrix), since the perturbation
  only alters spot locations, not the feature matrix X.
- You need a changelog/identified dataset provenance — the article provides no accession
  for the spatial transcription data or MSI-MALDI data used in the pseudo-paired evaluations.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3174
tools:
- name: haCCA
  role: The integration workflow whose pseudo-paired benchmark pairs produced by this
    perturbation are used to evaluate alignment methods (manual_gross_alignment, icp_3d_alignment,
    direct_alignment) by label transfer accuracy and ARI
  repo: github.com/LittleLittleCloud/haCCA
- name: Scanpy
  role: Used to load the source spatial transcriptome AnnData objects (sc.read_h5ad)
    and to emit the paired a.h5ad and b_prime.h5ad benchmark files
- name: Python
  role: Implementation language for the pseudo-MALDI-MSI benchmark data generator
    (feature splitting + Gaussian translation)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:91d85cadefd883c428a27c5faa44c5459d45b00acbc8e1322fa99e71a1165622
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# gaussian-coordinate-perturbation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmark-data construction technique for haCCA-style spot-to-spot integration evaluation: the spatial coordinates of one member of a paired dataset (e.g. a pseudo-MALDI-MSI section) are perturbed by Gaussian translation so the pair mimics the morphological disagreement between neighboring tissue sections, creating realistic pseudo-paired inputs without real paired acquisitions.

## When to use

Use when generating pseudo-paired spatial transcriptome / MALDI-MSI benchmark pairs from a single source tissue — e.g. to stress-test coordinate-alignment methods (manual gross, ICP 3D, direct alignment) against realistic inter-section morphology drift.

## When NOT to use

- You have real paired spatial-transcriptome/MALDI-MSI sections with actual morphological disagreement — no synthetic perturbation is needed.
- Your downstream method does not use spatial coordinates (D matrix), since the perturbation only alters spot locations, not the feature matrix X.
- You need a changelog/identified dataset provenance — the article provides no accession for the spatial transcription data or MSI-MALDI data used in the pseudo-paired evaluations.

## Inputs

- Spatial transcriptome matrices for source tissues (3 mouse brain, 1 rat brain, 1 human gastric cancer) as AnnData objects with .X feature matrix and .obsm['spatial'] coordinates
- Feature/gene set of each source tissue to be randomly split into two disjoint subsets

## Outputs

- Paired benchmark files a.h5ad and b_prime.h5ad, one pair per feature split — 15 pairs per source tissue
- Gaussian-translated spatial coordinate matrices (dtrain and dtruth) exhibiting simulated inter-section morphological disagreement

## How to apply

Load the spatial transcriptome matrices for each source tissue (3 mouse brain, 1 rat brain, 1 human gastric cancer) as AnnData objects with Scanpy, then for each tissue randomly split the feature (gene) set into two disjoint subsets — one representing the pseudo-spatial-transcriptome section and the other the pseudo-MALDI-MSI section. Apply Gaussian translation to the spatial coordinates of one section (dtrain and dtruth) so the translated coordinates deviate from the originals by the Gaussian perturbation amount, mimicking the morphological disagreement between neighboring sections. Emit the paired data as a.h5ad and b_prime.h5ad files, one pair per split, yielding 15 pairs per source tissue (n=15 pairs total per tissue source). Each pair must retain the same number of spots as the source; the perturbation deliberately breaks exact spot correspondence so downstream alignment (manual_gross_alignment, icp_3d_alignment, direct_alignment) must recover the mapping for evaluation by label transfer accuracy/ARI.

## Related tools

- **haCCA** (The integration workflow whose pseudo-paired benchmark pairs produced by this perturbation are used to evaluate alignment methods (manual_gross_alignment, icp_3d_alignment, direct_alignment) by label transfer accuracy and ARI) — github.com/LittleLittleCloud/haCCA
- **Scanpy** (Used to load the source spatial transcriptome AnnData objects (sc.read_h5ad) and to emit the paired a.h5ad and b_prime.h5ad benchmark files)
- **Python** (Implementation language for the pseudo-MALDI-MSI benchmark data generator (feature splitting + Gaussian translation))

## Examples

```
a_h5ad = sc.read_h5ad('/path/to/a.h5ad'); b_prime_h5ad = sc.read_h5ad('/path/to/b_prime.h5ad'); a = Data(X=a_h5ad.X.toarray(), D=a_h5ad.obsm['spatial']); _a, _b_prime = hacca.icp_3d_alignment(a, _b_prime)
```

## Evaluation signals

- Each generated pair retains the same number of spots as its source tissue dataset.
- Translated coordinates in the perturbed section deviate from the original coordinates by exactly the applied Gaussian perturbation amount.
- The feature (gene) sets of the two sections in each pair are disjoint, since they come from the random split of the source gene set.
- Exactly 15 pairs (a.h5ad / b_prime.h5ad) are emitted per source tissue, one per feature split.
- Downstream haCCA alignment on the perturbed pairs yields a measurable baseline (label transfer accuracy, label transfer ARI) against which the 6–20% outperformance of haCCA over STUtility was assessed.

## Limitations

- The perturbation is synthetic: no data accession or dataset identifier is given for the spatial transcription data or MSI-MALDI data used in the pseudo-paired evaluations.
- The perturbation only simulates coordinate disagreement; haCCA's behavior on higher-resolution data has not been evaluated.
- The 6–20% benchmark improvement over STUtility lacks a stated STUtility software version.

## Evidence

- [results] Gaussian translation mimics morphological disagreement: [evidence span withheld — anchor status: not_found]
