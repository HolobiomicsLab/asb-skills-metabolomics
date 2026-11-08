---
name: feature-relabeling-md5-hash
description: 'Use when preparing fingerprint-bearing MS1 features for qiime qemistree
  make-hierarchy: raw feature IDs (from MZmine2 feature detection) must become stable
  tip names that persist across independently processed datasets.'
when_to_use_negative:
- Feature identifiers are already stable, unique, and dataset-independent — re-hashing
  would break linkage to prior annotations
- The dataset contains only MS1 features without MS/MS spectra; these are excluded
  by Qemistree ('features with only MS1 are not considered') so no relabeling step
  will produce tips
- The goal is taxonomy assignment or spectral library matching rather than tree construction
  — hashing is a hierarchy-building pre-step, not needed elsewhere
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3292
tools:
- name: Qemistree (q2-qemistree)
  role: 'Implements the relabeling as part of make-hierarchy: MS1 features are relabeled
    with deterministic checksum IDs and identical fingerprint vectors merged before
    hierarchical clustering'
  repo: https://github.com/biocore/q2-qemistree
- name: CSI:FingerID
  role: Produces the fingerprint vectors (2936 molecular properties) whose identity
    determines the merged representative tips
- name: QIIME 2
  role: Artifact framework in which the relabeled feature table and rooted tree are
    emitted and validated
  repo: https://github.com/biocore/q2-qemistree
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:699024d44f5ee09550fa7e1b0292274a2571c49c0368224598a77f3d9d5f1db4
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
  - Python
  - QIIME 2
  - Qemistree
  - Qemistree (q2-qemistree)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# feature-relabeling-md5-hash

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Deterministically relabel MS1 chemical features with MD5-based checksum identifiers before building the Qemistree hierarchy, guaranteeing unique, dataset-independent tip names and enabling identical fingerprint vectors to be merged into a single representative tip.

## When to use

Use when preparing fingerprint-bearing MS1 features for qiime qemistree make-hierarchy: raw feature IDs (from MZmine2 feature detection) must become stable tip names that persist across independently processed datasets.

## When NOT to use

- Feature identifiers are already stable, unique, and dataset-independent — re-hashing would break linkage to prior annotations
- The dataset contains only MS1 features without MS/MS spectra; these are excluded by Qemistree ('features with only MS1 are not considered') so no relabeling step will produce tips
- The goal is taxonomy assignment or spectral library matching rather than tree construction — hashing is a hierarchy-building pre-step, not needed elsewhere

## Inputs

- fingerprints.qza (artifact type CSIFolder; per-feature probabilities over 2936 molecular properties predicted by CSI:FingerID from SIRIUS fragmentation trees)
- feature-table.qza (FeatureTable[Frequency] with raw MS1 feature IDs from MZmine2 preprocessing of .mzXML/.mzML/.mzDATA files)

## Outputs

- Hashed feature table with MD5-identifiers replacing raw feature IDs, MS1 features without fingerprints filtered out
- Rooted chemical tree (qemistree.qza, Phylogeny[Rooted]) whose tips are MD5-hashed feature identifiers, merged where fingerprint vectors are identical
- Combined feature data carrying parent mass, retention time, and the new feature identifiers

## How to apply

Take the CSI:FingerID fingerprint artifact (fingerprints.qza, artifact type CSIFolder holding per-feature probabilities over the 2936 molecular properties) together with the matching feature-table.qza (FeatureTable[Frequency]). Replace each dataset-specific MS1 feature label with a deterministic MD5-based checksum identifier so tip names are unique and reproducible regardless of which dataset or run produced the feature — this is what lets independently processed datasets be merged later without repeating feature detection or retention-time alignment. Merge features that share identical fingerprint vectors into a single representative tip so tips correspond one-to-one with distinct fingerprint-defined molecules. Before tree assembly, filter MS1 features lacking fingerprints out of the feature table. The relabeled table plus hashed identifiers then feed UPGMA hierarchical clustering (Euclidean distances on fingerprint probability vectors, or binarized fingerprints with Jaccard distance) to emit the rooted chemical hierarchy.

## Related tools

- **Qemistree (q2-qemistree)** (Implements the relabeling as part of make-hierarchy: MS1 features are relabeled with deterministic checksum IDs and identical fingerprint vectors merged before hierarchical clustering) — https://github.com/biocore/q2-qemistree
- **CSI:FingerID** (Produces the fingerprint vectors (2936 molecular properties) whose identity determines the merged representative tips)
- **QIIME 2** (Artifact framework in which the relabeled feature table and rooted tree are emitted and validated) — https://github.com/biocore/q2-qemistree

## Examples

```
qiime qemistree make-hierarchy --i-fingerprints fingerprints.qza --i-feature-table feature-table.qza --o-tree qemistree.qza
```

## Evaluation signals

- Re-running the relabeling yields byte-identical tip names (MD5 checksums are deterministic), and merging a second dataset introduces no duplicate tips
- Every fingerprint-bearing feature appears exactly once as a tip: tips correspond one-to-one with molecules
- Features sharing identical fingerprint vectors were collapsed to a single representative tip rather than duplicated
- Hashed feature table contains no MS1-only features lacking fingerprints
- qemistree.qza exists and parses as a rooted phylogeny usable downstream in Empress or Weighted UniFrac

## Limitations

- Identical fingerprint vectors are merged, so structurally distinct molecules with indistinguishable CSI:FingerID predictions share one tip
- Only chemical features with MS/MS data survive; MS1-only features are dropped from the tree and hashed table
- Tree extent is constrained by fingerprint predictability: in negative ionization mode fewer fingerprints can be confidently predicted, resulting in less-extensive trees
- Qemistree inherits the limitations of the underlying fingerprint prediction tool CSI:FingerID, whose quality depends on available MS/MS reference spectra

## Evidence
