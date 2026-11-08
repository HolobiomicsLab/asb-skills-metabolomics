---
name: molecular-fingerprint-distance-computation
description: Use when you have per-feature fingerprint vectors (e.g., a CSI:FingerID
  artifact with 2936 property probabilities) and need a chemical hierarchy to enable
  tree-informed comparisons of untargeted metabolomics data.
when_to_use_negative:
- Features have only MS1 data and no MS/MS fragmentation spectra — no molecular fingerprint
  can be predicted, so features with only MS1 are not considered
- You need accurate structural identifications rather than a similarity hierarchy
  — spectral library matches are more accurate but typically cover only 1–20% of features;
  use them for annotation instead
- Negative ionization mode data with few confidently predictable fingerprints — the
  resulting trees may be too sparse (less-extensive) for meaningful comparison
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: q2-qemistree
  role: QIIME 2 plugin that performs fingerprint-based hierarchy construction (make-hierarchy),
    including checksum relabeling, distance computation, and UPGMA clustering
  repo: https://github.com/biocore/q2-qemistree
- name: CSI:FingerID
  role: Produces the predicted molecular fingerprints (2936 properties via kernel
    support vector machines on SIRIUS fragmentation trees) that serve as the input
    vectors for distance computation
- name: QIIME 2
  role: Framework hosting the plugin; outputs (rooted tree plus filtered feature table)
    plug into QIIME 2 tree-based workflows such as Weighted UniFrac and Faith's Phylogenetic
    Diversity
- name: Empress
  role: Interactive phylogenetic tree viewer used to visualize Qemistree chemical
    hierarchies
  repo: https://github.com/biocore/empress
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:34099914d9063467705e283b3448a9ab3720dc7759c30d83078ce7c998bae61f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/biocore/q2-qemistree
  repo_url: https://github.com/biocore/q2-qemistree
  tools:
  - CSI:FingerID
  - Empress
  - Python
  - QIIME 2
  - Qemistree
  - q2-qemistree
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# molecular-fingerprint-distance-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes pairwise chemical distances between MS1 features from their predicted CSI:FingerID fingerprint vectors (2936 molecular properties) and assembles them via hierarchical clustering into a rooted chemical tree whose tips correspond one-to-one with molecules.

## When to use

Use when you have per-feature fingerprint vectors (e.g., a CSI:FingerID artifact with 2936 property probabilities) and need a chemical hierarchy to enable tree-informed comparisons of untargeted metabolomics data.

## When NOT to use

- Features have only MS1 data and no MS/MS fragmentation spectra — no molecular fingerprint can be predicted, so features with only MS1 are not considered
- You need accurate structural identifications rather than a similarity hierarchy — spectral library matches are more accurate but typically cover only 1–20% of features; use them for annotation instead
- Negative ionization mode data with few confidently predictable fingerprints — the resulting trees may be too sparse (less-extensive) for meaningful comparison

## Inputs

- CSI:FingerID fingerprint artifact (fingerprints.qza, semantic type CSIFolder) with per-feature probabilities over the 2936 molecular properties
- QIIME 2 feature table (feature-table.qza, FeatureTable[Frequency])
- Optionally: feature annotations/parent mass and retention time for merging into output feature data

## Outputs

- Rooted chemical hierarchy (qemistree.qza, Phylogeny[Rooted]) with molecules one-to-one at tips
- Hashed feature table with MS1 features lacking fingerprints filtered out
- Combined feature data with parent mass, retention time, and MD5 checksum feature identifiers

## How to apply

Load the CSI:FingerID fingerprint artifact (type CSIFolder) alongside the feature-table.qza (FeatureTable[Frequency]). Relabel each MS1 feature with a deterministic MD5-based checksum identifier so tip names are unique across datasets, and merge features with identical fingerprint vectors into one representative tip. Compute the pairwise distance matrix over fingerprint probability vectors using Euclidean distance, or optionally binarize the fingerprints and use Jaccard distance with a 0.5 presence threshold. Apply UPGMA (unweighted average-linkage) hierarchical clustering to the distance matrix, then emit the rooted tree (Phylogeny[Rooted]) together with the hashed feature table in which MS1 features without fingerprints are filtered out. The rationale is that fingerprint-based representation is largely independent of technical variation such as chromatography shifts, enabling cross-dataset comparison without repeating feature detection or retention time alignment.

## Related tools

- **q2-qemistree** (QIIME 2 plugin that performs fingerprint-based hierarchy construction (make-hierarchy), including checksum relabeling, distance computation, and UPGMA clustering) — https://github.com/biocore/q2-qemistree
- **CSI:FingerID** (Produces the predicted molecular fingerprints (2936 properties via kernel support vector machines on SIRIUS fragmentation trees) that serve as the input vectors for distance computation)
- **QIIME 2** (Framework hosting the plugin; outputs (rooted tree plus filtered feature table) plug into QIIME 2 tree-based workflows such as Weighted UniFrac and Faith's Phylogenetic Diversity)
- **Empress** (Interactive phylogenetic tree viewer used to visualize Qemistree chemical hierarchies) — https://github.com/biocore/empress

## Examples

```
qiime qemistree make-hierarchy --i-fingerprints fingerprints.qza --i-feature-table feature-table.qza --o-tree qemistree.qza --o-feature-table hashed-table.qza
```

## Evaluation signals

- Output qemistree.qza exists and parses as a valid rooted phylogeny (Phylogeny[Rooted])
- Every fingerprint-bearing feature appears exactly once as a tip — tips correspond one-to-one with molecules
- Features sharing identical fingerprint vectors are merged into a single representative tip rather than duplicated
- The emitted feature table contains no MS1 features that lack fingerprints (they are filtered out)
- Tree tips carry the MD5-based checksum identifiers, matching the hashed feature table

## Limitations

- Qemistree depends on representing chemical features as molecular fingerprints and shares limitations with the underlying fingerprint prediction tool CSI:FingerID
- Fingerprint prediction depends on the quality and coverage of MS/MS spectral databases available for training the predictive models
- In negative ionization mode, fewer molecular fingerprints can be confidently predicted due to fewer publicly available reference spectra, resulting in less-extensive trees
- Different tree-building approaches will produce different tree topologies capturing complementary chemical information, and broader benchmarking of when to use each approach is still needed

## Evidence
