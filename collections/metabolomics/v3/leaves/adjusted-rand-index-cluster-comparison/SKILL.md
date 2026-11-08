---
name: adjusted-rand-index-cluster-comparison
description: Use when spatial transcriptome and MALDI-MSI (or other spatial omics)
  sections have been aligned by competing methods and cluster labels exist on both
  sections, and you need label transfer AC/ARI to rank the alignments.
when_to_use_negative:
- Sections share actual spots or features between modalities — the article notes the
  integration problem exists precisely because there are no shared spots or features,
  so overlap-based label comparison is not applicable
- Higher-resolution spatial data, image-based sequencing, spatial proteomics, or histological
  images — the article states haCCA's behavior on such data has not been evaluated
  and only spatial transcriptome and spatial metabolome data have been used
- Pseudo-paired data without ground-truth cluster labels on both sections — AC and
  ARI cannot be computed without Label arrays on a and b_prime
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: haCCA
  role: Produces b_predict via manual_gross_alignment → icp_3d_alignment → direct_alignment;
    labels from its Data objects are scored by AC and ARI
  repo: github.com/LittleLittleCloud/haCCA
- name: Scanpy
  role: Reads a.h5ad and b_prime.h5ad and provides the processed spatial data whose
    cluster labels are transferred and compared
- name: STUtility
  role: Morphological-alignment comparator whose label transfer AC/ARI establishes
    the 6-20% baseline gap that haCCA must exceed
- name: ICP
  role: Standalone Iterative Closest Point alignment comparator scored with the same
    AC/ARI metrics
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:5aa6542bc43b1842c410e507bd4fc11d460abf8c7fff03bbaed918b553edadf2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LittleLittleCloud/haCCA
  tools:
  - ICP
  - Python
  - STUtility
  - Scanpy
  - haCCA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# adjusted-rand-index-cluster-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies agreement between cluster labels transferred across spatially aligned spot sets using label transfer accuracy (AC) and Adjusted Rand Index (ARI), as used in haCCA to benchmark spot-to-spot integration of spatial transcriptomes and MALDI-MSI metabolomes against morphological-only alignments.

## When to use

Use when spatial transcriptome and MALDI-MSI (or other spatial omics) sections have been aligned by competing methods and cluster labels exist on both sections, and you need label transfer AC/ARI to rank the alignments.

## When NOT to use

- Sections share actual spots or features between modalities — the article notes the integration problem exists precisely because there are no shared spots or features, so overlap-based label comparison is not applicable
- Higher-resolution spatial data, image-based sequencing, spatial proteomics, or histological images — the article states haCCA's behavior on such data has not been evaluated and only spatial transcriptome and spatial metabolome data have been used
- Pseudo-paired data without ground-truth cluster labels on both sections — AC and ARI cannot be computed without Label arrays on a and b_prime

## Inputs

- Aligned spot-to-spot label assignments from haCCA (b_predict from Data triplets of X feature matrix, D spatial matrix, optional Label cluster array)
- Cluster labels transferred under comparator alignments (STUtility morphological alignment, standalone ICP, direct alignment)
- Label arrays attached to Data objects a and b_prime

## Outputs

- benchmark_alignment_comparison.csv with label transfer accuracy (AC) and label transfer ARI per alignment method
- Ranking of haCCA vs direct, STUtility-manual, and ICP alignments (Figure 2 reproduction)

## How to apply

For each alignment method — the haCCA chain (manual_gross_alignment → icp_3d_alignment → direct_alignment producing b_predict), direct alignment alone, STUtility morphological alignment, and standalone ICP — transfer cluster labels across aligned spots between the paired sections (a, b_prime). Compute two metrics per method: label transfer accuracy (AC) and label transfer ARI. Assemble the per-method AC/ARI values into benchmark_alignment_comparison.csv and compare the combined haCCA alignment against the morphological-only alignment; the expected result is that haCCA outperforms morphological alignment (represented by STUtility) by 6-20% AC, and that the modified morphological alignment component of haCCA alone already outperforms STUtility with further gains from combining alignments.

## Related tools

- **haCCA** (Produces b_predict via manual_gross_alignment → icp_3d_alignment → direct_alignment; labels from its Data objects are scored by AC and ARI) — github.com/LittleLittleCloud/haCCA
- **Scanpy** (Reads a.h5ad and b_prime.h5ad and provides the processed spatial data whose cluster labels are transferred and compared)
- **STUtility** (Morphological-alignment comparator whose label transfer AC/ARI establishes the 6-20% baseline gap that haCCA must exceed)
- **ICP** (Standalone Iterative Closest Point alignment comparator scored with the same AC/ARI metrics)

## Examples

```
_b_prime = hacca.manual_gross_alignment(a, b_prime); _a, _b_prime = hacca.icp_3d_alignment(a, _b_prime); b_predict = hacca.direct_alignment(_a, _b_prime)
```

## Evaluation signals

- benchmark_alignment_comparison.csv contains an AC and ARI value for each of the four alignment methods (haCCA, direct, STUtility-manual, ICP)
- haCCA's label transfer AC exceeds the morphological-only (STUtility) alignment by 6-20%
- haCCA achieves the highest label transfer accuracy (AC) and ARI among all four methods on the pseudo multi-module spatial data
- Combined haCCA alignment (manual gross + ICP 3D + direct) scores at or above the modified morphological alignment of haCCA alone, which itself outperforms STUtility
- Reproduces the qualitative ordering reported for Figure 2

## Limitations

- No data accession or dataset identifier is given for the spatial transcriptomics or MSI-MALDI data used in the pseudo-paired evaluation, complicating direct reproduction
- No software version number is stated for STUtility used as the benchmark comparator
- Only spatial transcriptome and spatial metabolome data have been evaluated; other modalities (image-based sequencing, spatial proteomics, histological images) are not yet supported
- Behavior on higher-resolution data has not been explored

## Evidence

- [discussion] AC/ARI benchmark result: "we shown that haCCA outperformed morphological alignment by 6-20%, represented by STUtility, evaluating by label transfer accuracy, label transfer ARI"
- [readme] Metric definitions via Data triplets: "Data is a triplet of (X: np.ndarray, D: np.ndarray, Label: Optional[np.ndarray]), where X is the feature matrix, D is the spatial matrix"
- [methods] Alignment chain whose labels are scored: "manual_gross_alignment | icp_3d_alignment | direct_alignment"
- [readme] Output semantics of scored alignment: "b_predict contains aligned feature from a and samples from b_prime"
- [intro] Comparator framing: "proving haCCA outperformed morphological-aligned method(for example, STUtility)"
