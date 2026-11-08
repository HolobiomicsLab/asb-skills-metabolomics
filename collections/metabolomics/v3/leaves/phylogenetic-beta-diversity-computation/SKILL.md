---
name: phylogenetic-beta-diversity-computation
description: Use when a chemical hierarchy (Qemistree Newick/Phylogeny[Rooted]) exists
  for the MS1 features and the goal is to quantify sample dissimilarity in a way that
  accounts for chemical relatedness of features, e.g., batch-effect mitigation or
  food-ontology clustering.
when_to_use_negative:
- The feature set contains only MS1 features without predicted fingerprints — MS1-only
  features are filtered out and cannot be placed on the hierarchy
- No chemical hierarchy tree has been built (use qiime qemistree make-hierarchy from
  fingerprint predictions first)
- The goal is spectral-library annotation coverage rather than sample-level dissimilarity
  — annotations from spectral matches alone cover typically 1-20% of features, too
  low to summarize sample chemistry
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: QIIME 2
  role: Runs beta-phylogenetic (weighted UniFrac), beta (Bray-Curtis), beta-group-significance
    (PERMANOVA), and pcoa commands
  repo: https://github.com/biocore/q2-qemistree
- name: Qemistree
  role: Builds the chemical hierarchy (make-hierarchy from fingerprint predictions)
    that serves as the phylogeny input for UniFrac
  repo: https://github.com/biocore/q2-qemistree
- name: Empress
  role: Visualizes the Qemistree tree alongside PCoA ordinations as an integrated
    community-plot
  repo: https://github.com/biocore/empress
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:00023b7797f7c1ebdee483aaae48b4df61541c9dd8f86f1f7e6683f8a0a63fdf
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
  - GNPS
  - Python
  - QIIME 2
  - Qemistree
  - R
  - SIRIUS
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# phylogenetic-beta-diversity-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute tree-informed beta diversity (weighted UniFrac on the Qemistree chemical hierarchy) between metabolomics samples using QIIME 2 phylogenetic diversity tools, in contrast to tree-agnostic metrics like Bray-Curtis, then test group separation with PERMANOVA and visualize via PCoA.

## When to use

Use when a chemical hierarchy (Qemistree Newick/Phylogeny[Rooted]) exists for the MS1 features and the goal is to quantify sample dissimilarity in a way that accounts for chemical relatedness of features, e.g., batch-effect mitigation or food-ontology clustering.

## When NOT to use

- The feature set contains only MS1 features without predicted fingerprints — MS1-only features are filtered out and cannot be placed on the hierarchy
- No chemical hierarchy tree has been built (use qiime qemistree make-hierarchy from fingerprint predictions first)
- The goal is spectral-library annotation coverage rather than sample-level dissimilarity — annotations from spectral matches alone cover typically 1-20% of features, too low to summarize sample chemistry

## Inputs

- FeatureTable[Frequency] (quant table, e.g., MZmine2-processed .biom from MassIVE accession MSV000083306 or Global FoodOmics)
- Qemistree chemical hierarchy as phylogeny artifact (Phylogeny[Rooted], Newick)
- Sample metadata TSV with grouping column (e.g., batch, food ontology level 1-4)

## Outputs

- Weighted UniFrac distance matrix (.qza)
- Bray-Curtis distance matrix (.qza) for tree-agnostic comparison
- PERMANOVA pseudo-F and p-values (999 permutations) per metric
- PCoA ordination with interactive Emperor/EMPress plot (.qzv)

## How to apply

Import the quant feature table as FeatureTable[Frequency] and the Qemistree hierarchy as a phylogeny artifact, then compute the weighted UniFrac distance matrix with qiime diversity beta-phylogenetic using the chemical tree as the phylogeny input. In parallel compute the tree-agnostic Bray-Curtis matrix with qiime diversity beta so the two metrics can be compared. Assign samples to groups via the metadata column of interest (e.g., chromatographic batch or food sample type) and run one-sided PERMANOVA (999 permutations) with qiime diversity beta-group-significance on both matrices. Generate PCoA ordinations with qiime diversity pcoa and export interactive ordination plots as .qzv (Emperor/EMPress). The rationale is that the fingerprint-based representation is largely independent of technical variation such as chromatography shifts, so UniFrac on the chemical tree mitigates batch signal that Bray-Curtis retains.

## Related tools

- **QIIME 2** (Runs beta-phylogenetic (weighted UniFrac), beta (Bray-Curtis), beta-group-significance (PERMANOVA), and pcoa commands) — https://github.com/biocore/q2-qemistree
- **Qemistree** (Builds the chemical hierarchy (make-hierarchy from fingerprint predictions) that serves as the phylogeny input for UniFrac) — https://github.com/biocore/q2-qemistree
- **Empress** (Visualizes the Qemistree tree alongside PCoA ordinations as an integrated community-plot) — https://github.com/biocore/empress

## Examples

```
qiime diversity beta-phylogenetic --i-table table.qza --i-phylogeny qemistree.qza --p-metric weighted_unifrac --o-distance-matrix wunifrac.qza
```

## Evaluation signals

- On the 162-sample evaluation dataset, one-sided PERMANOVA should recover pseudo-F=120.75 (p=0.001) for Bray-Curtis vs pseudo-F=18.2239 (p=0.001) for weighted UniFrac, demonstrating the tree-informed metric mitigates the chromatographic batch signal
- Each distance matrix is square and symmetric over the N samples (e.g., N=126 for Global FoodOmics) with zero diagonal
- For food-ontology levels 1-4, within-food sample pairs should be more chemically similar than between-food pairs, matching the qualitative pattern of Extended Data Table 4 where fingerprint-based UniFrac supports chemically meaningful clustering at least as well as Bray-Curtis
- The PCoA .qzv renders in Emperor/EMPress without missing-sample errors, confirming table/tree/metadata sample IDs intersect correctly
- All tips in the chemical hierarchy map to feature IDs present in the (MS1-filtered) feature table

## Limitations

- Fingerprint prediction depends on the quality and coverage of MS/MS spectral databases; in negative ionization mode fewer fingerprints can be confidently predicted, resulting in less-extensive trees
- Different tree-building approaches (fingerprint-based vs MS/MS cosine-score) produce different topologies capturing complementary chemical information; broader benchmarking is needed to know when each should be used
- Only chemical features with MS/MS data are included; features with only MS1 are not considered

## Evidence

- [discussion] Using Qemistree we can begin to explore the applicability of other methods, such as Faith's Phylogenetic Diversity: [evidence span withheld — anchor status: not_found]
