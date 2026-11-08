---
name: phylogeny-format-serialization
description: Use when fingerprint vectors predicted from MS/MS fragmentation spectra
  must be emitted as a rooted chemical tree (Phylogeny[Rooted]) with one tip per molecule
  for downstream tree-based analyses.
when_to_use_negative:
- Features lack MS/MS data — only chemical features with MS/MS spectra are considered;
  MS1-only features cannot obtain fingerprints and cannot become tips.
- The tree already exists as a species/ASV phylogeny (e.g. 16S rRNA data); this skill
  is for fingerprint-derived chemical hierarchies, not biological phylogenies.
- You need a different tree topology (e.g. ClassyFire taxonomy-based); the article
  notes different tree-building approaches produce different, complementary topologies.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0194
tools:
- name: Qemistree (q2-qemistree)
  role: QIIME 2 plugin whose make-hierarchy method builds and serializes the rooted
    chemical tree from fingerprint artifacts
  repo: https://github.com/biocore/q2-qemistree
- name: QIIME 2
  role: artifact framework providing the Phylogeny[Rooted] semantic type and .qza
    serialization for the output tree
- name: iTOL
  role: interactive visualization of the serialized Qemistree tree
- name: Empress
  role: phylogenetic tree viewer used to visualize Qemistree trees as q2-empress visualizations
  repo: https://github.com/biocore/empress
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:5dbbdfcd996e0022bb1e4740efd7259266ed84bfb4f3382f11f962c07a3e7c68
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
  - Qemistree (q2-qemistree)
  - iTOL
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# phylogeny-format-serialization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Serializes the Qemistree chemical hierarchy — assembled by UPGMA clustering of CSI:FingerID fingerprint vectors — as a rooted phylogeny artifact (qemistree.qza, Newick inside a .qza) so that LC-MS/MS metabolomics features can be consumed by phylogeny-based tools built for DNA data.

## When to use

Use when fingerprint vectors predicted from MS/MS fragmentation spectra must be emitted as a rooted chemical tree (Phylogeny[Rooted]) with one tip per molecule for downstream tree-based analyses.

## When NOT to use

- Features lack MS/MS data — only chemical features with MS/MS spectra are considered; MS1-only features cannot obtain fingerprints and cannot become tips.
- The tree already exists as a species/ASV phylogeny (e.g. 16S rRNA data); this skill is for fingerprint-derived chemical hierarchies, not biological phylogenies.
- You need a different tree topology (e.g. ClassyFire taxonomy-based); the article notes different tree-building approaches produce different, complementary topologies.

## Inputs

- fingerprints.qza (QIIME 2 artifact of type CSIFolder: per-feature CSI:FingerID probabilities over 2936 molecular properties)
- feature-table.qza (QIIME 2 artifact of type FeatureTable[Frequency])

## Outputs

- qemistree.qza (QIIME 2 artifact of type Phylogeny[Rooted], Newick-format chemical tree with molecules as tips)
- hashed feature table (features without fingerprints filtered out)
- combined feature data with parent mass, retention time, and feature identifiers

## How to apply

After CSI:FingerID predicts per-feature probabilities over the 2936 molecular properties (fingerprints.qza, type CSIFolder), run the q2-qemistree make-hierarchy method together with the FeatureTable[Frequency] feature-table.qza. Relabel each MS1 feature with a deterministic MD5-based checksum identifier so tip names are unique and merge features sharing identical fingerprint vectors into one representative tip; the UPGMA (unweighted average-linkage) hierarchy over Euclidean distances between fingerprint probability vectors is then written out as a rooted tree. Emit the tree as qemistree.qza (Phylogeny[Rooted]) alongside the hashed feature table — from which MS1 features without fingerprints are filtered out — and combined feature data carrying parent mass, retention time, and feature identifiers. If only annotated molecules are needed, apply prune-hierarchy to keep just the annotated tips. Because the artifact is Newick-format and QIIME 2-compatible, it can be visualized with iTOL or Empress and feeds tree-informed metrics such as Weighted UniFrac.

## Related tools

- **Qemistree (q2-qemistree)** (QIIME 2 plugin whose make-hierarchy method builds and serializes the rooted chemical tree from fingerprint artifacts) — https://github.com/biocore/q2-qemistree
- **QIIME 2** (artifact framework providing the Phylogeny[Rooted] semantic type and .qza serialization for the output tree)
- **iTOL** (interactive visualization of the serialized Qemistree tree)
- **Empress** (phylogenetic tree viewer used to visualize Qemistree trees as q2-empress visualizations) — https://github.com/biocore/empress

## Examples

```
qiime qemistree make-hierarchy
```

## Evaluation signals

- The output qemistree.qza exists and parses as a rooted phylogeny (Phylogeny[Rooted]).
- Every fingerprint-bearing feature appears exactly once as a tip — tips correspond one-to-one with molecules.
- Tip names are deterministic MD5-based checksum identifiers, unique across datasets, and features with identical fingerprint vectors share a single representative tip.
- The accompanying feature table contains no MS1 features lacking fingerprints.
- The Newick tree loads successfully in a tree viewer (iTOL or q2-empress) and supports downstream tree-informed metrics such as Weighted UniFrac and Faith's Phylogenetic Diversity.

## Limitations

- Only chemical features with MS/MS data are included; features with only MS1 are not considered and are filtered out of the feature table.
- Fingerprint prediction depends on the quality and coverage of MS/MS spectral databases used for training, which caps tree completeness.
- In negative ionization mode, fewer molecular fingerprints can be confidently predicted due to fewer publicly available reference spectra, resulting in less-extensive trees.

## Evidence

- [readme] Empress supports Qemistree trees: [evidence span withheld — anchor status: not_found]
