---
name: weighted-unifrac-distance-estimation-on-chemical-tree
description: Use when a metabolomics chemical-feature table (FeatureTable[Frequency])
  is paired with a Qemistree hierarchy and you need tree-informed sample distances,
  PCoA ordination, or PERMANOVA that reduces chromatography-driven batch signal.
when_to_use_negative:
- Only MS1 features are available (no MS/MS fragmentation spectra), so no fingerprints
  can be predicted and no Qemistree hierarchy can be built to support UniFrac.
- The feature table has not been matched to the hierarchy (MS1 features without fingerprints
  still present) — filter those features first or the phylogenetic beta computation
  cannot run.
- The goal is per-molecule chemical taxonomy assignment (ClassyFire kingdom through
  direct parent) rather than between-sample beta diversity — use get-classyfire-taxonomy/prune-hierarchy
  instead.
edam_topics:
- http://edamontology.org/topic_3293
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3697
tools:
- name: QIIME 2
  role: 'Hosts the diversity plugin commands used here: beta-phylogenetic (weighted
    UniFrac on the chemical tree), beta (Bray-Curtis baseline), pcoa, and beta-group-significance
    (PERMANOVA)'
  repo: https://docs.qiime2.org/2019.7/install/
- name: q2-qemistree
  role: Builds the chemical hierarchy (compute-fragmentation-trees, predict-fingerprints,
    make-hierarchy, prune-hierarchy) whose tips serve as the tree for the UniFrac
    calculation
  repo: https://github.com/biocore/q2-qemistree
- name: Empress
  role: Interactive visualization of the Qemistree chemical tree alongside PCoA ordination
    plots exported as .qzv
  repo: https://github.com/biocore/empress
- name: Emperor
  role: Interactive PCoA ordination plots of the weighted UniFrac distance matrix
    exported from QIIME 2 as .qzv
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:c84a73278ee81e892a906d70977d1ea7b1e349f3f84f4a8f1c02a27902bcd1c2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/biocore/q2-qemistree
  tools:
  - Emperor
  - Empress
  - Python
  - QIIME 2
  - Qemistree
  - q2-qemistree
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3293
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8947368421052632
n_claims: 19
---

# weighted-unifrac-distance-estimation-on-chemical-tree

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes weighted UniFrac between-sample distances for metabolomics data using a Qemistree chemical hierarchy — built from SIRIUS/CSI:FingerID-predicted fingerprints — as the tree, so beta diversity and PCoA reflect chemical relatedness instead of raw feature co-occurrence. This mitigates technical variation such as chromatographic batch shifts: on the 162-sample evaluation dataset the batch pseudo-F drops from 120.75 (Bray-Curtis) to 18.2239 (weighted UniFrac), both p=0.001.

## When to use

Use when a metabolomics chemical-feature table (FeatureTable[Frequency]) is paired with a Qemistree hierarchy and you need tree-informed sample distances, PCoA ordination, or PERMANOVA that reduces chromatography-driven batch signal.

## When NOT to use

- Only MS1 features are available (no MS/MS fragmentation spectra), so no fingerprints can be predicted and no Qemistree hierarchy can be built to support UniFrac.
- The feature table has not been matched to the hierarchy (MS1 features without fingerprints still present) — filter those features first or the phylogenetic beta computation cannot run.
- The goal is per-molecule chemical taxonomy assignment (ClassyFire kingdom through direct parent) rather than between-sample beta diversity — use get-classyfire-taxonomy/prune-hierarchy instead.

## Inputs

- FeatureTable[Frequency] artifact of chemical features (e.g., an MZmine2/GNPS quant table such as the evaluation dataset quant table from MassIVE MSV000083306)
- Qemistree chemical hierarchy as a rooted phylogeny artifact (produced via qiime qemistree predict-fingerprints and make-hierarchy)
- Sample metadata TSV with a batch column defining the chromatographic batches (two batches in the evaluation dataset)

## Outputs

- Weighted UniFrac DistanceMatrix artifact (.qza), plus a parallel Bray-Curtis DistanceMatrix for comparison
- PCoA ordination artifacts (.qza) and Emperor/EMPress interactive ordination visualizations (.qzv)
- PERMANOVA results (pseudo-F and p-value, 999 permutations) from qiime diversity beta-group-significance (.qzv)

## How to apply

Import the chemical feature table (e.g., the 162-sample evaluation dataset quant table from MassIVE accession MSV000083306, produced by MZmine2 feature detection) as FeatureTable[Frequency] and the Qemistree chemical hierarchy as a rooted phylogeny artifact, then filter MS1 features without fingerprints from the table so every table feature is a tip in the tree. Compute the tree-informed distances with 'qiime diversity beta-phylogenetic --p-metric weighted_unifrac' using the Qemistree hierarchy as the phylogeny input, and in parallel compute the tree-agnostic baseline with 'qiime diversity beta --p-metric braycurtis' for comparison. Ordinate both matrices with 'qiime diversity pcoa' and export interactive Emperor/EMPress ordination plots (.qzv). To quantify batch signal, run a one-sided PERMANOVA (999 permutations) on both matrices with 'qiime diversity beta-group-significance', grouping samples on the chromatographic batch column of the sample metadata. The rationale is that the fingerprint-based representation is largely independent of chromatography shifts across MS experiments, so the tree-informed metric should show substantially lower batch pseudo-F — reported 18.2239 (p=0.001) for weighted UniFrac versus 120.75 (p=0.001) for Bray-Curtis on the evaluation dataset.

## Related tools

- **QIIME 2** (Hosts the diversity plugin commands used here: beta-phylogenetic (weighted UniFrac on the chemical tree), beta (Bray-Curtis baseline), pcoa, and beta-group-significance (PERMANOVA)) — https://docs.qiime2.org/2019.7/install/
- **q2-qemistree** (Builds the chemical hierarchy (compute-fragmentation-trees, predict-fingerprints, make-hierarchy, prune-hierarchy) whose tips serve as the tree for the UniFrac calculation) — https://github.com/biocore/q2-qemistree
- **Empress** (Interactive visualization of the Qemistree chemical tree alongside PCoA ordination plots exported as .qzv) — https://github.com/biocore/empress
- **Emperor** (Interactive PCoA ordination plots of the weighted UniFrac distance matrix exported from QIIME 2 as .qzv)

## Examples

```
qiime diversity beta-phylogenetic --i-table feature-table.qza --i-phylogeny qemistree-hierarchy.qza --p-metric weighted_unifrac --o-distance-matrix weighted-unifrac-dm.qza
```

## Evaluation signals

- PERMANOVA on the 162-sample evaluation dataset recovers pseudo-F ≈ 18.2239 (p=0.001) for weighted UniFrac versus pseudo-F ≈ 120.75 (p=0.001) for Bray-Curtis — the tree-informed pseudo-F must be far lower.
- The beta-phylogenetic run succeeds only after MS1 features without fingerprints are filtered from the feature table, so every table feature corresponds to a tip in the Qemistree hierarchy.
- Each distance matrix is square and symmetric (162×162 for the evaluation dataset) with a zero diagonal and one row/column per sample id in the metadata.
- At 999 permutations the minimum attainable p-value is 0.001; reported PERMANOVA p=0.001 for both metrics is consistent with this permutation count.
- PCoA Emperor/EMPress .qzv files open at view.qiime2.org with batch metadata coloring and show visibly weaker batch separation under weighted UniFrac than under Bray-Curtis.

## Limitations

- Only chemical features with MS/MS data are considered; features with only MS1 are not included in the hierarchy and are filtered from the feature table.
- Qemistree depends on representing features as molecular fingerprints and shares limitations with CSI:FingerID; fingerprint prediction depends on the quality and coverage of MS/MS spectral databases, and spectral library matches alone (typically 1–20% of features) are too low to summarize complex samples.
- In negative ionization mode, fewer molecular fingerprints can be confidently predicted due to fewer publicly available reference spectra, resulting in less-extensive trees.
- Different tree-building approaches will produce different tree topologies capturing complementary chemical information; a broader benchmarking effort is needed to know when each approach should be used.

## Evidence
