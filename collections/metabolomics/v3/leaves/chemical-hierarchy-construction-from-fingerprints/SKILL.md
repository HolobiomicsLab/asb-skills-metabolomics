---
name: chemical-hierarchy-construction-from-fingerprints
description: Use when you have a feature table plus MS/MS spectra (MGF) from LC-MS/MS
  feature detection (e.g. MZmine2/GNPS) and want a fingerprint-based chemical tree
  for UniFrac-type sample comparison or cross-dataset merging without re-alignment.
when_to_use_negative:
- Input data lack MS/MS spectra (MS1-only features), since fingerprint prediction
  requires fragmentation trees and the features would be dropped.
- You only need spectral-library-match-based annotations for a small percentage of
  features — in such cases the low annotation coverage may be too sparse to summarize
  the chemical content of complex samples, and a tree-agnostic metric like Bray-Curtis
  may suffice.
- The dataset was already processed into a molecular-network cosine-score hierarchy
  (e.g. GNPS) and you only want topology-level annotation updates rather than fingerprint-based
  reconstruction.
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0605
tools:
- name: Qemistree (q2-qemistree)
  role: QIIME 2 plugin that orchestrates fragmentation-tree computation, formula reranking,
    fingerprint prediction, and hierarchy construction.
  repo: https://github.com/biocore/q2-qemistree
- name: SIRIUS
  role: De-novo molecular formula assignment and fragmentation-tree estimation from
    MS/MS spectra (isotope and fragmentation patterns).
  repo: https://bio.informatik.uni-jena.de/sirius/
- name: CSI:FingerID
  role: Predicts 2936 binary molecular properties from fragmentation trees via kernel
    support vector machines; these fingerprints define the pairwise distances used
    for tree construction.
- name: ClassyFire
  role: Assigns a 5-level chemical taxonomy (chemical kingdom, superclass, class,
    subclass, direct parent) to annotated molecules for tree decoration.
- name: QIIME 2
  role: Host framework for the q2-qemistree plugin; supplies phylogeny-aware distance
    and diversity methods and visualization (e.g. Empress).
  repo: https://github.com/biocore/q2-qemistree
- name: Empress
  role: Phylogenetic tree viewer used to interactively visualize Qemistree chemical
    hierarchies alongside sample metadata.
  repo: https://github.com/biocore/empress
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:b893c52e7f4dfa9daee327c061e3494d6a64100cb1fa5700f320474a8ae9a6d7
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
  - ClassyFire
  - Empress
  - GNPS
  - Python
  - QIIME 2
  - Qemistree
  - Qemistree (q2-qemistree)
  - R
  - SIRIUS
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# chemical-hierarchy-construction-from-fingerprints

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds a chemical hierarchy (Qemistree) over LC-MS/MS features by hierarchically clustering 2936-property molecular fingerprints predicted by SIRIUS/CSI:FingerID, so that tree-aware ecology metrics (e.g. Weighted UniFrac) and visualizations in QIIME 2/Empress can be applied to metabolomics data.

## When to use

Use when you have a feature table plus MS/MS spectra (MGF) from LC-MS/MS feature detection (e.g. MZmine2/GNPS) and want a fingerprint-based chemical tree for UniFrac-type sample comparison or cross-dataset merging without re-alignment.

## When NOT to use

- Input data lack MS/MS spectra (MS1-only features), since fingerprint prediction requires fragmentation trees and the features would be dropped.
- You only need spectral-library-match-based annotations for a small percentage of features — in such cases the low annotation coverage may be too sparse to summarize the chemical content of complex samples, and a tree-agnostic metric like Bray-Curtis may suffice.
- The dataset was already processed into a molecular-network cosine-score hierarchy (e.g. GNPS) and you only want topology-level annotation updates rather than fingerprint-based reconstruction.

## Inputs

- Feature table (BIOM) from feature detection, with observed ion peak areas (e.g. MZmine2/GNPS quant table)
- MGF file of MS/MS fragmentation spectra (e.g. sirius.mgf)
- Feature data mapping feature IDs to MS/MS spectra (FeatureData[Spectra])
- Optional: MS/MS library matches imported as FeatureData[Molecules]
- Optional: PubChem/SIRIUS reference database version consistent with the CSI:FingerID training release

## Outputs

- Newick-format chemical hierarchy (Qemistree) relating MS1 features by predicted fingerprint similarity
- Fingerprint prediction table (2936 molecular properties per feature)
- ClassyFire 5-level chemical taxonomy (kingdom, superclass, class, subclass, direct parent) for annotated molecules
- Filtered feature table restricted to MS1 features with predicted fingerprints
- QIIME 2 artifacts for downstream UniFrac / Faith's PD / Empress visualization

## How to apply

Process the MZmine2/GNPS feature output with the q2-qemistree plugin: first run 'compute-fragmentation-trees' on the MGF through SIRIUS (e.g. profile orbitrap, ppm-max 15), then 'rerank-molecular-formulas' (ZODIAC scoring, --p-zodiac-threshold 0.95), and 'predict-fingerprints' to obtain 2936 binary molecular-property predictions per feature via CSI:FingerID. Only features with MS/MS spectra are considered; MS1-only features without fingerprints are filtered out before merging the feature table with the fingerprint hierarchy. Build the hierarchy with 'make-hierarchy' by hierarchically clustering the fingerprint vectors into a Newick tree, optionally retaining MS/MS library matches with --i-ms2-matches; optionally decorate the tree with a ClassyFire 5-level taxonomy ('get-classyfire-taxonomy') or prune it to annotated molecules ('prune-hierarchy'). The resulting tree plus feature table supports Weighted UniFrac/Faith's PD comparisons, and visual exploration in Empress; because fingerprints are independent of retention time alignment, datasets processed separately can be merged into one hierarchy without re-running feature detection.

## Related tools

- **Qemistree (q2-qemistree)** (QIIME 2 plugin that orchestrates fragmentation-tree computation, formula reranking, fingerprint prediction, and hierarchy construction.) — https://github.com/biocore/q2-qemistree
- **SIRIUS** (De-novo molecular formula assignment and fragmentation-tree estimation from MS/MS spectra (isotope and fragmentation patterns).) — https://bio.informatik.uni-jena.de/sirius/
- **CSI:FingerID** (Predicts 2936 binary molecular properties from fragmentation trees via kernel support vector machines; these fingerprints define the pairwise distances used for tree construction.)
- **ClassyFire** (Assigns a 5-level chemical taxonomy (chemical kingdom, superclass, class, subclass, direct parent) to annotated molecules for tree decoration.)
- **QIIME 2** (Host framework for the q2-qemistree plugin; supplies phylogeny-aware distance and diversity methods and visualization (e.g. Empress).) — https://github.com/biocore/q2-qemistree
- **Empress** (Phylogenetic tree viewer used to interactively visualize Qemistree chemical hierarchies alongside sample metadata.) — https://github.com/biocore/empress

## Examples

```
qiime qemistree compute-fragmentation-trees --i-spectra sirius.mgf --p-profile orbitrap --p-ppm-max 15 --o-fragments-tree fragments-tree.qza ; qiime qemistree make-hierarchy --p-ppm-max 15 --o-tree hierarchy.qza
```

## Evaluation signals

- The output Newick tree tips correspond one-to-one with the MS1 features retained after filtering (features with predicted fingerprints), and the tree is rooted for UniFrac computation.
- The fingerprint table contains 2936 molecular-property columns per feature; --p-qc-properties should restrict these to 489 PubChem fingerprint positions when building contingency tables.
- MS1 features without predicted fingerprints are excluded from the merged feature table used for downstream UniFrac.
- Running compute-fragmentation-trees → rerank-molecular-formulas → predict-fingerprints in sequence yields progressively refined formulas (ZODIAC reranking at threshold 0.95) before tree construction.
- Downstream ordination on Weighted UniFrac over the chemical tree should reproduce chemically meaningful clustering comparable to or better than tree-agnostic Bray-Curtis on the same feature table (per the Extended Data Table 4 comparison).

## Limitations

- Only 1–20% of features typically receive spectral-library matches; annotation coverage may be too low to summarize complex samples, which motivates but also constrains fingerprint-based trees.
- Fingerprint prediction depends on the quality and coverage of MS/MS spectral databases available for training the predictive models.
- In negative ionization mode fewer fingerprints can be confidently predicted because fewer reference spectra are publicly available, producing less-extensive trees.
- Qemistree inherits limitations of the underlying CSI:FingerID fingerprint predictions (e.g. false positives/negatives in substructure assignment).
- Different tree-building approaches (fingerprint-based vs MS/MS cosine-based) yield different topologies capturing complementary chemical information; a broader benchmarking effort is needed to decide when each should be used.

## Evidence

- [intro] Qemistree software that constructs a chemical tree based on predicted molecular fingerprints from MS/MS fragmentation spectra: "Here, we introduce Qemistree (pronounced chemis-tree) software that constructs a chemical tree based on predicted molecular fingerprints from MS/MS fragmentation spectra"
- [intro] CSI:FingerID operates on the fragmentation trees using kernel support vector machines to predict molecular properties: "CSI:FingerID operates on the fragmentation trees using kernel support vector machines to predict molecular properties (2936 properties; Supplementary Dataset 1)"
- [intro] We use these molecular fingerprints to calculate pairwise distances between chemical features and hierarchically cluster the fingerprint vectors to generate a tree: "We use these molecular fingerprints to calculate pairwise distances between chemical features and hierarchically cluster the fingerprint vectors to generate a tree"
- [intro] SIRIUS then determines the molecular formula of each feature using the isotope and fragmentation patterns and estimates the best fragmentation tree explaining the fragmentation spectrum.: "SIRIUS then determines the molecular formula of each feature using the isotope and fragmentation patterns and estimates the best fragmentation tree explaining the fragmentation spectrum."
- [intro] Only chemical features with MS/MS data are included; features with only MS1 are not considered.: "Only chemical features with MS/MS data are included; features with only MS1 are not considered."
- [methods] MS1 features without fingerprints are filtered out of this feature table.: "MS1 features without fingerprints are filtered out of this feature table."
