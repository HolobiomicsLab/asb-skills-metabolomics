---
name: label-transfer-accuracy-evaluation
description: Use when comparing spot-to-spot label transfer outcomes across aligned
  spatial data objects (e.g., b_predict from haCCA vs STUtility/ICP/direct alignments)
  against ground-truth cluster labels, to score integration accuracy by AC and ARI.
when_to_use_negative:
- No ground-truth or reference cluster labels exist for either modality, so no label
  transfer accuracy or ARI can be computed
- The goal is comparing continuous feature (gene/metabolite) values rather than discrete
  spot-level cluster labels across aligned spots
- The datasets share spots or features directly, making a label-transfer accuracy
  proxy unnecessary for judging integration
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3361
tools:
- name: haCCA
  role: Produces b_predict via manual_gross_alignment, icp_3d_alignment, and direct_alignment;
    its label transfer accuracy and ARI are the values being evaluated against the
    STUtility benchmark
  repo: github.com/LittleLittleCloud/haCCA
- name: STUtility
  role: Morphological-alignment comparator whose outcome supplies the ground-truth
    cluster labels and the benchmark AC/ARI that haCCA must exceed by 6-20%
- name: ICP
  role: Standalone iterative-closest-point alignment comparator run on the same pseudo-paired
    pairs before label transfer scoring
- name: Scanpy
  role: Loads a.h5ad and b_prime.h5ad so cluster Label arrays can be extracted for
    transfer and accuracy computation
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:fe5c7bb063230406906a8a10966088b83a374099811bd0856d61d0f576cc000c
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
  - ICP
  - Python
  - STUtility
  - Scanpy
  - haCCA
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# label-transfer-accuracy-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how accurately cluster labels are transferred across aligned spots between two spatial modalities (e.g., 10X Visium spatial transcriptome and MALDI-MSI), using label transfer accuracy (AC) and label transfer ARI to benchmark integration methods such as haCCA against morphological-alignment baselines.

## When to use

Use when comparing spot-to-spot label transfer outcomes across aligned spatial data objects (e.g., b_predict from haCCA vs STUtility/ICP/direct alignments) against ground-truth cluster labels, to score integration accuracy by AC and ARI.

## When NOT to use

- No ground-truth or reference cluster labels exist for either modality, so no label transfer accuracy or ARI can be computed
- The goal is comparing continuous feature (gene/metabolite) values rather than discrete spot-level cluster labels across aligned spots
- The datasets share spots or features directly, making a label-transfer accuracy proxy unnecessary for judging integration

## Inputs

- b_predict / aligned Data objects (features from a, spots from b_prime) for each alignment method (haCCA, direct, STUtility, ICP)
- Cluster Label arrays attached to haCCA Data objects (from a.h5ad and b_prime.h5ad)
- Ground-truth cluster labels obtained via STUtility morphological alignment (for the mouse-brain Visium + MALDI-MSI evaluation)

## Outputs

- Per-method label transfer accuracy (AC) and label transfer ARI values
- benchmark_alignment_comparison.csv (pseudo multi-module AC/ARI for haCCA, direct, STUtility-manual, ICP)
- label_transfer_accuracy_mouse_brain.csv (per-brain accuracies for the three mouse-brain sections)

## How to apply

After inferring b_predict for each alignment method (haCCA's manual_gross_alignment + icp_3d_alignment + direct_alignment, standalone ICP, or STUtility morphological alignment), transfer cluster labels across the aligned spots of the pseudo-paired or same-section spatial datasets. For each of the four methods, compute label transfer accuracy (AC) and label transfer ARI, and assemble them into benchmark_alignment_comparison.csv for the pseudo multi-module comparison. For the 10X Visium + MALDI-MSI mouse-brain sections, use the cluster labels obtained via STUtility morphological alignment as ground truth, evaluate per brain, and write label_transfer_accuracy_mouse_brain.csv. Correct application is judged by haCCA exceeding the STUtility morphological-alignment benchmark by 6-20% AC, and by the three mouse-brain accuracies matching the reported 87.5%, 79.5%, and 87.6% (Fig 3 A,C,E).

## Related tools

- **haCCA** (Produces b_predict via manual_gross_alignment, icp_3d_alignment, and direct_alignment; its label transfer accuracy and ARI are the values being evaluated against the STUtility benchmark) — github.com/LittleLittleCloud/haCCA
- **STUtility** (Morphological-alignment comparator whose outcome supplies the ground-truth cluster labels and the benchmark AC/ARI that haCCA must exceed by 6-20%)
- **ICP** (Standalone iterative-closest-point alignment comparator run on the same pseudo-paired pairs before label transfer scoring)
- **Scanpy** (Loads a.h5ad and b_prime.h5ad so cluster Label arrays can be extracted for transfer and accuracy computation)

## Evaluation signals

- haCCA's label transfer AC exceeds the STUtility morphological-alignment benchmark by 6-20% on pseudo multi-module data
- Per-brain label transfer accuracies for the three mouse-brain sections match the reported 87.5%, 79.5%, and 87.6% (Fig 3 A,C,E)
- benchmark_alignment_comparison.csv contains AC and ARI rows for all four methods (haCCA, direct, STUtility-manual, ICP), with haCCA achieving the highest AC and ARI
- label_transfer_accuracy_mouse_brain.csv reports one accuracy per mouse brain evaluated against STUtility-derived ground-truth labels
- Both label transfer accuracy and label transfer ARI are reported, not accuracy alone

## Limitations

- Ground truth for the same-section mouse-brain evaluation is itself STUtility's morphological alignment outcome, so accuracy is measured against a comparator-derived reference rather than an independent annotation
- Behavior of haCCA on higher-resolution data has not been evaluated, affecting how transfer accuracies generalize
- No data accession or dataset identifier is given for the pseudo-paired and real-world evaluations, limiting reproducibility of the reported AC/ARI values
- No software version number is stated for the STUtility comparator used as the benchmark

## Evidence
