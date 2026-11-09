---
name: upgma-hierarchical-clustering
description: Use when you have predicted molecular fingerprints (e.g., CSI:FingerID
  probability vectors over 2936 molecular properties) for LC-MS/MS chemical features
  and need a chemical tree for downstream tree-informed metrics like Weighted UniFrac
  or Faith's PD.
when_to_use_negative:
- The input features lack MS/MS data (features with only MS1 are not considered, since
  no fragmentation spectra means no fingerprints).
- You already have spectral library annotations for enough features that a ClassyFire-taxonomy-based
  hierarchy would suffice; the paper notes only 1–20% of features typically have library
  matches, so fingerprint clustering is for summarizing the unannotated majority.
- You need a phylogenetic tree of organisms/DNA sequences — this skill clusters molecular
  fingerprints, not sequence data.
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: Qemistree
  role: Implements make-hierarchy, which computes pairwise fingerprint distances and
    applies UPGMA clustering to build the rooted chemical tree
  repo: https://github.com/biocore/q2-qemistree
- name: CSI:FingerID
  role: Predicts the 2936-property molecular fingerprints from SIRIUS fragmentation
    trees whose vectors are clustered
- name: SIRIUS
  role: Determines molecular formulas and fragmentation trees that CSI:FingerID consumes;
    molecular substructures from SIRIUS feed the hierarchy
- name: QIIME 2
  role: Plugin host providing the artifact types (CSIFolder, FeatureTable[Frequency],
    Phylogeny[Rooted]) and the qiime qemistree make-hierarchy method
  repo: https://github.com/biocore/q2-qemistree
- name: Empress
  role: Interactive visualization of the resulting Qemistree chemical hierarchy
  repo: https://github.com/biocore/empress
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:f7c190a71a1dedf1fb73ed0d51538c9f8419dd123a00ef45facd1e0d6b7e189e
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
  - SIRIUS
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# upgma-hierarchical-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds a rooted chemical hierarchy ('Qemistree') by hierarchically clustering CSI:FingerID molecular fingerprint vectors with UPGMA, so that MS1 features occupy a one-to-one correspondence with tree tips and tree-based ecological tools can be applied to metabolomics data.

## When to use

Use when you have predicted molecular fingerprints (e.g., CSI:FingerID probability vectors over 2936 molecular properties) for LC-MS/MS chemical features and need a chemical tree for downstream tree-informed metrics like Weighted UniFrac or Faith's PD.

## When NOT to use

- The input features lack MS/MS data (features with only MS1 are not considered, since no fragmentation spectra means no fingerprints).
- You already have spectral library annotations for enough features that a ClassyFire-taxonomy-based hierarchy would suffice; the paper notes only 1–20% of features typically have library matches, so fingerprint clustering is for summarizing the unannotated majority.
- You need a phylogenetic tree of organisms/DNA sequences — this skill clusters molecular fingerprints, not sequence data.

## Inputs

- fingerprints.qza (QIIME 2 artifact of type CSIFolder: per-feature probabilities over the 2936 molecular properties predicted by CSI:FingerID)
- feature-table.qza (QIIME 2 artifact of type FeatureTable[Frequency])
- MS1 feature identifiers from MZmine2 pre-processed MGF output

## Outputs

- qemistree.qza (QIIME 2 artifact of type Phylogeny[Rooted], tips one-to-one with fingerprint-bearing molecules)
- hashed feature table with fingerprint-less MS1 features filtered out
- combined feature data with parent mass, retention time, and checksum feature identifiers

## How to apply

Load the CSI:FingerID fingerprint artifact (fingerprints.qza, type CSIFolder) together with the feature-table.qza (FeatureTable[Frequency]). Relabel each MS1 feature with a deterministic MD5-based checksum identifier for unique tip names, and merge features sharing identical fingerprint vectors into a single representative tip. Compute the pairwise distance matrix over fingerprint vectors using Euclidean distance on the probability vectors (optionally binarize fingerprints and apply Jaccard distance with a 0.5 presence threshold). Apply UPGMA (unweighted average-linkage) hierarchical clustering to assemble the chemical hierarchy whose tips correspond one-to-one with molecules, then emit the rooted tree (Phylogeny[Rooted]) alongside the hashed feature table with MS1 features lacking fingerprints filtered out.

## Related tools

- **Qemistree** (Implements make-hierarchy, which computes pairwise fingerprint distances and applies UPGMA clustering to build the rooted chemical tree) — https://github.com/biocore/q2-qemistree
- **CSI:FingerID** (Predicts the 2936-property molecular fingerprints from SIRIUS fragmentation trees whose vectors are clustered)
- **SIRIUS** (Determines molecular formulas and fragmentation trees that CSI:FingerID consumes; molecular substructures from SIRIUS feed the hierarchy)
- **QIIME 2** (Plugin host providing the artifact types (CSIFolder, FeatureTable[Frequency], Phylogeny[Rooted]) and the qiime qemistree make-hierarchy method) — https://github.com/biocore/q2-qemistree
- **Empress** (Interactive visualization of the resulting Qemistree chemical hierarchy) — https://github.com/biocore/empress

## Examples

```
qiime qemistree make-hierarchy
```

## Evaluation signals

- The output qemistree.qza exists and parses as a rooted phylogeny (Phylogeny[Rooted]).
- Every fingerprint-bearing feature appears exactly once as a tip — tips correspond one-to-one with molecules.
- Features lacking fingerprints (MS1-only features) are absent from both the tree tips and the emitted hashed feature table.
- Features sharing identical fingerprint vectors are collapsed into a single representative tip rather than duplicated.
- Tree topology is reproducible across runs given the same fingerprints, since MD5-based checksum identifiers are deterministic.

## Limitations

- Fingerprint prediction depends on the quality and coverage of MS/MS spectral databases used to train the CSI:FingerID models.
- In negative ionization mode, fewer fingerprints can be confidently predicted due to fewer publicly available reference spectra, producing less-extensive trees.
- Different tree-building approaches produce different tree topologies capturing complementary chemical information; broader benchmarking is needed to know when to use which approach.
- Qemistree inherits the limitations of the underlying fingerprint prediction tool CSI:FingerID, since it depends on representing chemical features as molecular fingerprints.

## Evidence
