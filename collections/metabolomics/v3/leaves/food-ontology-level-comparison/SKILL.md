---
name: food-ontology-level-comparison
description: Use when a metabolomics feature table has rich sample metadata organized
  as a multi-level ontology (e.g. food sample types in Global FoodOmics) and you need
  to quantify, per ontology level, whether fingerprint-tree-informed sample dissimilarities
  reveal structure that tree-agnostic metrics miss.
when_to_use_negative:
- The dataset lacks multi-level sample-type ontology metadata, so 'within-food vs
  between-food' comparison per level cannot be defined
- Features carry only MS1 spectra with no predicted fingerprints and no MS/MS library
  matches — the chemical hierarchy and hence fingerprint-based UniFrac cannot be built
  (the article notes features with only MS1 are not considered)
- The analysis goal is simply a single global clustering of samples rather than a
  per-ontology-level comparison across multiple metrics
edam_operation: http://edamontology.org/operation_3845
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Qemistree (q2-qemistree)
  role: Builds the chemical hierarchy from CSI:FingerID-predicted fingerprints and
    the cosine-score hierarchy from GNPS matches; provides compute-fragmentation-trees,
    rerank-molecular-formulas, predict-fingerprints, and make-hierarchy commands
  repo: https://github.com/biocore/q2-qemistree
- name: SIRIUS
  role: Computes fragmentation trees and molecular formulas for each feature (with
    --p-ppm-max 15, --p-profile orbitrap) before fingerprint prediction
  repo: https://bio.informatik.uni-jena.de/sirius/
- name: CSI:FingerID
  role: Predicts the 2936 molecular properties per feature that define the fingerprint
    vectors used in the hierarchy
  repo: https://bio.informatik.uni-jena.de/sirius/
- name: GNPS
  role: Provides feature-based molecular networking matches used to construct the
    MS/MS cosine-score hierarchy
  repo: https://gnps.ucsd.edu/
- name: QIIME 2
  role: Hosts the qemistree plugin and phylogeny-based metrics (weighted UniFrac)
    used for sample dissimilarity computations
  repo: https://qiime2.org
- name: q2-empress
  role: Visualizes the Qemistree tree; the article notes Qemistree trees can be visualized
    using q2-empress
  repo: https://github.com/biocore/empress
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:0f62a1f226ccb1cb4fbd790b3bde7118e31e02fecdca097d65cfce124ea9fb9e
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
  - GNPS
  - Python
  - QIIME 2
  - Qemistree
  - Qemistree (q2-qemistree)
  - R
  - SIRIUS
  - q2-empress
  edam_operation: http://edamontology.org/operation_3845
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# food-ontology-level-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assess whether a Qemistree fingerprint-based chemical hierarchy captures chemically meaningful sample clustering at each level of a food ontology (levels 1–4) by comparing weighted UniFrac, Bray-Curtis, CSCS, and MS/MS cosine-score hierarchy dissimilarities across within-food vs between-food sample pairs, and compile the per-metric per-level counts into an Extended Data Table 4-style comparison.

## When to use

Use when a metabolomics feature table has rich sample metadata organized as a multi-level ontology (e.g. food sample types in Global FoodOmics) and you need to quantify, per ontology level, whether fingerprint-tree-informed sample dissimilarities reveal structure that tree-agnostic metrics miss.

## When NOT to use

- The dataset lacks multi-level sample-type ontology metadata, so 'within-food vs between-food' comparison per level cannot be defined
- Features carry only MS1 spectra with no predicted fingerprints and no MS/MS library matches — the chemical hierarchy and hence fingerprint-based UniFrac cannot be built (the article notes features with only MS1 are not considered)
- The analysis goal is simply a single global clustering of samples rather than a per-ontology-level comparison across multiple metrics

## Inputs

- Global FoodOmics quant tables (feature tables of chemical features with peak areas per sample, N=126 food samples)
- Sample metadata mapping each sample to food ontology categories (levels 1–4)
- GNPS feature-based molecular networking matches (imported as FeatureData[Molecules] to build the cosine-score hierarchy)
- Qemistree outputs: predicted fingerprints (2936 properties), fragmentation trees, merged hierarchy qza

## Outputs

- extended_data_table4_metric_comparison.csv: per-metric (fingerprint w UniFrac, Bray-Curtis, CSCS, cosine-hierarchy w UniFrac) × per-ontology-level (1–4) counts of food types showing chemically meaningful clustering
- Pairwise sample dissimilarity matrices under the four metrics
- Optional: Qemistree hierarchy visualizations (e.g. via q2-empress or iTOL)

## How to apply

Process the quant tables through the Qemistree/QIIME 2 pipeline (compute-fragmentation-trees with SIRIUS at --p-ppm-max 15 and --p-profile orbitrap, rerank-molecular-formulas at --p-zodiac-threshold 0.95, predict-fingerprints via CSI:FingerID yielding 2936 predicted properties, then make-hierarchy merging feature tables and filtering out MS1 features without predicted fingerprints). Separately build the MS/MS cosine-score hierarchy from GNPS feature-based molecular networking matches imported as FeatureData[Molecules]. Compute pairwise sample dissimilarities across all N=126 samples under four metrics: weighted UniFrac on the fingerprint hierarchy, Bray-Curtis on the feature table, CSCS, and weighted UniFrac on the MS/MS cosine-score hierarchy. For each food ontology level 1–4, count the number of food sample types for which within-food pairs are more chemically similar than between-food pairs, and compile these per-metric per-level counts into extended_data_table4_metric_comparison.csv. Validate that the recomputed table reproduces the qualitative pattern of Extended Data Table 4 — the fingerprint-based weighted UniFrac supports chemically meaningful clustering at least as well as the tree-agnostic Bray-Curtis at every ontology level.

## Related tools

- **Qemistree (q2-qemistree)** (Builds the chemical hierarchy from CSI:FingerID-predicted fingerprints and the cosine-score hierarchy from GNPS matches; provides compute-fragmentation-trees, rerank-molecular-formulas, predict-fingerprints, and make-hierarchy commands) — https://github.com/biocore/q2-qemistree
- **SIRIUS** (Computes fragmentation trees and molecular formulas for each feature (with --p-ppm-max 15, --p-profile orbitrap) before fingerprint prediction) — https://bio.informatik.uni-jena.de/sirius/
- **CSI:FingerID** (Predicts the 2936 molecular properties per feature that define the fingerprint vectors used in the hierarchy) — https://bio.informatik.uni-jena.de/sirius/
- **GNPS** (Provides feature-based molecular networking matches used to construct the MS/MS cosine-score hierarchy) — https://gnps.ucsd.edu/
- **QIIME 2** (Hosts the qemistree plugin and phylogeny-based metrics (weighted UniFrac) used for sample dissimilarity computations) — https://qiime2.org
- **q2-empress** (Visualizes the Qemistree tree; the article notes Qemistree trees can be visualized using q2-empress) — https://github.com/biocore/empress

## Examples

```
qiime qemistree make-hierarchy --i-fingerprints fingerprints.qza --i-feature-tables merged-table.qza --p-filter-ms1 --o-hierarchy hierarchy.qza && qiime diversity beta --i-table filtered-table.qza --p-metric weighted_unifrac --i-phylogeny hierarchy.qza --o-distance-matrix unifrac-dm.qza
```

## Evaluation signals

- The recomputed extended_data_table4_metric_comparison.csv reproduces the qualitative pattern of Extended Data Table 4: fingerprint-based weighted UniFrac supports chemically meaningful clustering at least as well as tree-agnostic Bray-Curtis across food ontology levels 1–4
- All four dissimilarity matrices are square, symmetric, N=126 × 126, with zero diagonals
- Number of food sample types counted per ontology level matches the ontology structure (e.g. more types at finer levels 3–4 than at coarse levels 1–2)
- Only features with predicted fingerprints contribute to the fingerprint-hierarchy UniFrac; MS1-only features are excluded
- Cosine-score hierarchy UniFrac counts are ≤ fingerprint-hierarchy UniFrac counts when MS/MS library-match coverage is low (typical 1–20% of features)

## Limitations

- Typically only 1–20% of features have spectral library matches, depending on how well the sample type has been investigated, so the cosine-score hierarchy covers a much smaller feature subset than the fingerprint hierarchy
- Fingerprint prediction depends on the quality and coverage of MS/MS spectral databases available for training the predictive models; in negative ionization mode fewer fingerprints can be confidently predicted, resulting in less-extensive trees
- Different tree-building approaches produce different topologies capturing complementary chemical information, and a broader benchmarking effort is still needed to know when each approach should be used
- Spectral match annotations alone have coverage too low to adequately summarize the chemical content of complex samples

## Evidence

- [results] The Qemistree tree coupled to sample metadata revealed distinct chemical classes expected for each food sample type: [evidence span withheld — anchor status: not_found]
- [other] Source card for the four-metric comparison workflow: [evidence span withheld — anchor status: not_found]
- [other] Source card description of the per-ontology-level assessment: [evidence span withheld — anchor status: not_found]
