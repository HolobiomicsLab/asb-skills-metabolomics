---
name: chemical-tree-construction
description: Use when untargeted LC-MS/MS features have predicted molecular fingerprints
  (e.g., from SIRIUS/CSI:FingerID on MZmine2 MGF output) and you need a chemical hierarchy
  for downstream tree-based analyses like UniFrac or Faith's phylogenetic diversity.
when_to_use_negative:
- Features have only MS1 data and no MS/MS fragmentation spectra — Qemistree only
  includes chemical features with MS/MS data
- A tree already exists but needs trimming — use the prune-hierarchy method to keep
  only molecules with annotations instead of rebuilding
- You only need library-match-based classification (e.g., ClassyFire 5-level taxonomy)
  rather than a fingerprint-derived hierarchy
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Qemistree
  role: Constructs the chemical tree from predicted molecular fingerprints via the
    make-hierarchy method of the q2-qemistree QIIME 2 plugin
  repo: https://github.com/biocore/q2-qemistree
- name: CSI:FingerID
  role: Predicts the 2936 molecular property probabilities (fingerprint vectors) that
    are clustered into the hierarchy
- name: SIRIUS
  role: Determines molecular formulas and fragmentation trees that CSI:FingerID consumes
    to predict fingerprints
- name: QIIME 2
  role: Plugin framework hosting qemistree make-hierarchy and downstream phylogeny-based
    analysis tools
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
  source_content_hash: sha256:63d981362ce16c369c78b5a6e182bf2a8d31a2520365f7239b51594b3806bd5f
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 24
---

# chemical-tree-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Build a rooted chemical hierarchy (Qemistree) by hierarchically clustering CSI:FingerID fingerprint vectors so that mass-spectrometry features occupy tips of a phylogeny-like tree, enabling tree-based ecological analyses of metabolomics data.

## When to use

Use when untargeted LC-MS/MS features have predicted molecular fingerprints (e.g., from SIRIUS/CSI:FingerID on MZmine2 MGF output) and you need a chemical hierarchy for downstream tree-based analyses like UniFrac or Faith's phylogenetic diversity.

## When NOT to use

- Features have only MS1 data and no MS/MS fragmentation spectra — Qemistree only includes chemical features with MS/MS data
- A tree already exists but needs trimming — use the prune-hierarchy method to keep only molecules with annotations instead of rebuilding
- You only need library-match-based classification (e.g., ClassyFire 5-level taxonomy) rather than a fingerprint-derived hierarchy

## Inputs

- fingerprints.qza (QIIME 2 artifact of type CSIFolder, per-feature probabilities over 2936 molecular properties predicted by CSI:FingerID)
- feature-table.qza (QIIME 2 FeatureTable[Frequency])
- MS1 feature identifiers from MZmine2-preprocessed data (MGF of MS/MS spectra, e.g., sirius.mgf)

## Outputs

- qemistree.qza — rooted chemical hierarchy (QIIME 2 Phylogeny[Rooted]) with one tip per unique fingerprint vector
- Hashed feature table with MS1 features lacking fingerprints filtered out
- Combined feature data with parent mass, retention time, and MD5 checksum feature identifiers

## How to apply

Load the CSI:FingerID fingerprint artifact (fingerprints.qza, type CSIFolder, 2936 molecular properties per feature) together with the FeatureTable[Frequency]. Relabel each MS1 feature with a deterministic MD5-based checksum identifier for unique tip names, and merge features sharing identical fingerprint vectors into a single representative tip. Compute the pairwise distance matrix over the fingerprint probability vectors using Euclidean distance (optionally binarize fingerprints and use Jaccard distance with a 0.5 presence threshold), then apply UPGMA (unweighted average-linkage) hierarchical clustering to assemble the chemical hierarchy whose tips correspond one-to-one with molecules. Emit the rooted tree (Phylogeny[Rooted]) alongside the hashed feature table in which MS1 features without fingerprints are filtered out, plus combined feature data carrying parent mass and retention time. Only chemical features with MS/MS data are candidates; features with only MS1 are not considered. If only annotated molecules are needed downstream, apply prune-hierarchy to keep only molecules with annotations.

## Related tools

- **Qemistree** (Constructs the chemical tree from predicted molecular fingerprints via the make-hierarchy method of the q2-qemistree QIIME 2 plugin) — https://github.com/biocore/q2-qemistree
- **CSI:FingerID** (Predicts the 2936 molecular property probabilities (fingerprint vectors) that are clustered into the hierarchy)
- **SIRIUS** (Determines molecular formulas and fragmentation trees that CSI:FingerID consumes to predict fingerprints)
- **QIIME 2** (Plugin framework hosting qemistree make-hierarchy and downstream phylogeny-based analysis tools)
- **Empress** (Interactive visualization of the resulting Qemistree chemical hierarchy) — https://github.com/biocore/empress

## Examples

```
qiime qemistree make-hierarchy
```

## Evaluation signals

- Output qemistree.qza exists and parses as a rooted phylogeny (Phylogeny[Rooted])
- Every fingerprint-bearing feature appears exactly once as a tree tip (one-to-one correspondence between tips and molecules)
- Features sharing identical fingerprint vectors are collapsed into a single representative tip
- MS1 features without fingerprints are absent from the emitted hashed feature table
- Tip labels are MD5-based checksums, unique across datasets

## Limitations

- Fingerprint prediction quality depends on the coverage of MS/MS spectral databases used to train CSI:FingerID; typically only 1–20% of features have spectral library matches
- In negative ionization mode, fewer fingerprints can be confidently predicted due to fewer public reference spectra, yielding less-extensive trees
- Different tree-building approaches (e.g., Euclidean vs. Jaccard distances) produce different topologies capturing complementary chemical information; broader benchmarking is still needed
- The representation inherits the limitations of the underlying fingerprint predictor CSI:FingerID

## Evidence
