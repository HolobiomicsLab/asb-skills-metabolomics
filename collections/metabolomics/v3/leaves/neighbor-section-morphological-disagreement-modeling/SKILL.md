---
name: neighbor-section-morphological-disagreement-modeling
description: Use when creating pseudo-paired spatial transcriptome / MALDI-MSI benchmark
  datasets from spatial transcriptome matrices, so that spot-to-spot integration methods
  can be evaluated under realistic neighbor-section morphological disagreement.
when_to_use_negative:
- Input is real-world paired spatial transcriptome and MALDI-MSI data with its own
  tissue sections rather than pseudo-pairs needing synthetic morphological disagreement
- The spatial coordinates are already aligned between the two sections, so no Gaussian
  translation perturbation should be injected
- Data is non-spatial (bulk or single-cell without coordinates), since the perturbation
  acts on the spatial matrix D rather than only the feature matrix
tools:
- name: Scanpy
  role: Loads and standard-processes the source spatial transcriptome AnnData matrices
    and reads the emitted a.h5ad / b_prime.h5ad pairs
- name: haCCA
  role: Downstream workflow whose high-correlated feature pairs and modified spatial
    morphological alignment (manual_gross_alignment, icp_3d_alignment, direct_alignment)
    are benchmarked on the Gaussian-translated pseudo-pairs
  repo: github.com/LittleLittleCloud/haCCA
- name: Python
  role: Language in which the pseudo-pair generator and the published haCCA package
    are implemented
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:a57d6d478cb3721a445d1e8f33b3d2f9111d18fc9448865977fd815757028482
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# neighbor-section-morphological-disagreement-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A pseudo-MALDI-MSI benchmark data generator that mimics the morphological disagreement between neighboring tissue sections by randomly splitting a spatial transcriptome's gene set into two disjoint pseudo-sections and applying Gaussian translation to the spatial coordinates of one of them.

## When to use

Use when creating pseudo-paired spatial transcriptome / MALDI-MSI benchmark datasets from spatial transcriptome matrices, so that spot-to-spot integration methods can be evaluated under realistic neighbor-section morphological disagreement.

## When NOT to use

- Input is real-world paired spatial transcriptome and MALDI-MSI data with its own tissue sections rather than pseudo-pairs needing synthetic morphological disagreement
- The spatial coordinates are already aligned between the two sections, so no Gaussian translation perturbation should be injected
- Data is non-spatial (bulk or single-cell without coordinates), since the perturbation acts on the spatial matrix D rather than only the feature matrix

## Inputs

- AnnData (.h5ad) spatial transcriptome matrices for the 5 source tissues (3 mouse brain, 1 rat brain, 1 human gastric cancer), with spatial coordinates in obsm['spatial']
- Feature (gene) expression matrix X per tissue, standard Scanpy-processed

## Outputs

- Paired a.h5ad and b_prime.h5ad files per split (pseudo-spatial-transcriptome section and Gaussian-translated pseudo-MALDI-MSI section)
- 15 pseudo-pairs per source tissue (n=15 pairs total per tissue source)
- Benchmark inputs for label transfer accuracy / label transfer ARI comparison of haCCA vs STUtility

## How to apply

Load the spatial transcriptome matrices for the 5 source tissues (3 mouse brain, 1 rat brain, 1 human gastric cancer) as AnnData objects with Scanpy. For each source tissue, randomly split the feature (gene) set into two disjoint subsets: one representing the pseudo-spatial-transcriptome section and the other the pseudo-MALDI-MSI section. Apply Gaussian translation to the spatial coordinates of one section so the pair mimics the morphological disagreement observed between neighboring tissue sections. Emit the paired data as a.h5ad and b_prime.h5ad files, one pair per split, yielding 15 pairs per source tissue. These pseudo-pairs then feed haCCA's Data objects (X feature matrix, D spatial matrix) and its manual_gross_alignment, icp_3d_alignment, and direct_alignment pipeline to benchmark against STUtility via label transfer accuracy and label transfer ARI.

## Related tools

- **Scanpy** (Loads and standard-processes the source spatial transcriptome AnnData matrices and reads the emitted a.h5ad / b_prime.h5ad pairs)
- **haCCA** (Downstream workflow whose high-correlated feature pairs and modified spatial morphological alignment (manual_gross_alignment, icp_3d_alignment, direct_alignment) are benchmarked on the Gaussian-translated pseudo-pairs) — github.com/LittleLittleCloud/haCCA
- **Python** (Language in which the pseudo-pair generator and the published haCCA package are implemented)

## Examples

```
a_h5ad = sc.read_h5ad(os.path.join('/path/to/a.h5ad')); b_prime_h5ad = sc.read_h5ad(os.path.join('/path/to/b_prime.h5ad')); a = Data(X=a_h5ad.X.toarray(), D=a_h5ad.obsm['spatial'])
```

## Evaluation signals

- Each generated pair retains the same number of spots between a.h5ad and b_prime.h5ad
- Translated coordinates in b_prime deviate from the original coordinates by the Gaussian perturbation amount
- The two feature sets within a pair are disjoint subsets of the original gene set
- 15 pairs are emitted per source tissue (n=15 pairs total per tissue source)
- Running haCCA on the pairs yields 6-20% higher label transfer accuracy and label transfer ARI than STUtility morphological alignment

## Limitations

- No data accession or dataset identifier is given for the spatial transcriptome data used in the pseudo-paired evaluation
- haCCA's behavior on higher resolution data has not been evaluated, and integration is limited to spatial transcriptome and spatial metabolome data (image-based sequencing, spatial proteomics, histological images not yet supported)
- No software version number is stated for STUtility, the benchmark comparator, and no changelog exists for the haCCA repository

## Evidence

- [methods] Gaussian translation of pseudo-pairs: [evidence span withheld — anchor status: not_found]
- [methods] Feature split into pseudo-sections: [evidence span withheld — anchor status: not_found]
