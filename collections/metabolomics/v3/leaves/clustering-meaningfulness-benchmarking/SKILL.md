---
name: clustering-meaningfulness-benchmarking
description: Use when a quant/feature table plus sample metadata are available and
  the goal is to test whether chemically informed (hierarchy-based) sample clustering
  aligns with known sample-type categories better than agnostic ordination metrics.
when_to_use_negative:
- Chemical features are MS1-only with no MS/MS data, since only features with MS/MS
  spectra can receive fingerprint predictions and enter the hierarchy
- The goal is structural annotation of individual molecules rather than benchmarking
  sample-level clustering against a metadata ontology
- Spectral library match coverage is the only signal available (typically 1-20% of
  features), which is too low to summarize chemical content for a cosine-score-based
  comparison
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: Qemistree
  role: Builds the chemical hierarchy from predicted fingerprints and provides the
    QIIME 2 commands (compute-fragmentation-trees, rerank-molecular-formulas, predict-fingerprints,
    make-hierarchy) used to construct the tree for UniFrac comparisons
  repo: https://github.com/biocore/q2-qemistree
- name: SIRIUS
  role: Computes molecular formulas and fragmentation trees per feature to feed fingerprint
    prediction (--p-ppm-max 15, --p-profile orbitrap)
- name: CSI:FingerID
  role: Predicts the 2936 molecular-property fingerprint vector per feature used for
    hierarchical clustering of chemical features
- name: GNPS
  role: Source of MS/MS spectral library matches used to build the MS/MS cosine-score
    hierarchy (FeatureData[Molecules]) as a tree-based comparator
  repo: https://gnps.ucsd.edu/
- name: QIIME 2
  role: Plugin framework (q2-qemistree) hosting the tree construction and diversity-metric
    commands used in the benchmark
  repo: https://github.com/biocore/q2-qemistree
- name: R
  role: Used for system-wide analysis, e.g., compiling per-metric, per-ontology-level
    counts into the comparison table
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:621a8cbd5df66f6cca156b3e636aaee00674ab344a72a1128828af39ac956fe3
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
  - R
  - SIRIUS
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9047619047619048
n_claims: 21
---

# Clustering-meaningfulness benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark whether tree-informed sample dissimilarity metrics (weighted UniFrac on a Qemistree fingerprint hierarchy, CSCS, UniFrac on an MS/MS cosine-score hierarchy) capture chemically meaningful sample structure better than, or comparable to, the tree-agnostic Bray-Curtis metric, stratified by metadata ontology levels.

## When to use

Use when a quant/feature table plus sample metadata are available and the goal is to test whether chemically informed (hierarchy-based) sample clustering aligns with known sample-type categories better than agnostic ordination metrics.

## When NOT to use

- Chemical features are MS1-only with no MS/MS data, since only features with MS/MS spectra can receive fingerprint predictions and enter the hierarchy
- The goal is structural annotation of individual molecules rather than benchmarking sample-level clustering against a metadata ontology
- Spectral library match coverage is the only signal available (typically 1-20% of features), which is too low to summarize chemical content for a cosine-score-based comparison

## Inputs

- Quant/feature table (e.g., complex foodomics quant table) of LC-MS/MS chemical features with peak areas
- GNPS feature-based molecular networking matches for cosine-score hierarchy construction
- Sample metadata with an ontology of categorical levels (e.g., food ontology levels 1-4)
- Predicted molecular fingerprints (2936 properties per feature) from CSI:FingerID

## Outputs

- Per-metric, per-ontology-level comparison table (e.g., extended_data_table4_metric_comparison.csv)
- Chemical hierarchy tree (fingerprint-based) and MS/MS cosine-score hierarchy
- Pairwise sample dissimilarity matrices under weighted UniFrac, Bray-Curtis, and CSCS

## How to apply

Process the quant tables (e.g., Global FoodOmics complex foodomics quant table) through the Qemistree/QIIME 2 pipeline: run SIRius compute-fragmentation-trees (--p-ppm-max 15, --p-profile orbitrap), rerank-molecular-formulas (--p-zodiac-threshold 0.95), and predict-fingerprints with CSI:FingerID to obtain the 2936-property fingerprint vector per feature; filter out MS1 features without predicted fingerprints. Build the fingerprint hierarchy with qiime qemistree make-hierarchy and a parallel MS/MS cosine-score hierarchy from GNPS feature-based molecular networking matches imported as FeatureData[Molecules]. For all N samples (e.g., 126 food samples), compute pairwise dissimilarities under four metrics: weighted UniFrac on the fingerprint hierarchy, Bray-Curtis on the feature table, CSCS, and weighted UniFrac on the cosine-score hierarchy. For each level of the sample ontology (e.g., food ontology levels 1-4), test whether within-category sample pairs are more chemically similar than between-category pairs, and count chemically-meaningful clusterings per metric per level. Compile the per-metric, per-level counts into a comparison table (e.g., extended_data_table4_metric_comparison.csv) and check it reproduces the qualitative pattern reported in the source study, where fingerprint-based UniFrac supported meaningful clustering at least as well as Bray-Curtis.

## Related tools

- **Qemistree** (Builds the chemical hierarchy from predicted fingerprints and provides the QIIME 2 commands (compute-fragmentation-trees, rerank-molecular-formulas, predict-fingerprints, make-hierarchy) used to construct the tree for UniFrac comparisons) — https://github.com/biocore/q2-qemistree
- **SIRIUS** (Computes molecular formulas and fragmentation trees per feature to feed fingerprint prediction (--p-ppm-max 15, --p-profile orbitrap))
- **CSI:FingerID** (Predicts the 2936 molecular-property fingerprint vector per feature used for hierarchical clustering of chemical features)
- **GNPS** (Source of MS/MS spectral library matches used to build the MS/MS cosine-score hierarchy (FeatureData[Molecules]) as a tree-based comparator) — https://gnps.ucsd.edu/
- **QIIME 2** (Plugin framework (q2-qemistree) hosting the tree construction and diversity-metric commands used in the benchmark) — https://github.com/biocore/q2-qemistree
- **R** (Used for system-wide analysis, e.g., compiling per-metric, per-ontology-level counts into the comparison table)

## Examples

```
qiime qemistree compute-fragmentation-trees --p-ppm-max 15 --p-profile orbitrap && qiime qemistree predict-fingerprints && qiime qemistree make-hierarchy
```

## Evaluation signals

- Recomputed per-metric, per-ontology-level counts reproduce the qualitative pattern of the source comparison table (Extended Data Table 4 layout), with fingerprint-based weighted UniFrac supporting chemically meaningful clustering at least as well as tree-agnostic Bray-Curtis across ontology levels 1-4
- All four dissimilarity matrices are computed for the full sample set (N=126 in the Global FoodOmics case) with no missing pairwise comparisons
- Hierarchy contains only features with predicted fingerprints; MS1 features without fingerprints are filtered out of the feature table before UniFrac computation
- Number of chemically-meaningful clusterings per metric per ontology level is finite and directly comparable across metrics (same sample pairs, same ontology categories)
- CSI:FingerID output contains 2936 molecular properties per feature, matching Supplementary Dataset 1

## Limitations

- Different tree-building approaches (fingerprint-based vs MS/MS cosine-score) yield different tree topologies capturing complementary chemical information; a broader benchmarking effort is needed to know when each should be used
- Fingerprint prediction depends on the quality and coverage of MS/MS spectral databases available for training the predictive models
- In negative ionization mode, fewer molecular fingerprints can be confidently predicted, resulting in less-extensive trees
- Only 1-20% of features typically have spectral library matches, limiting the coverage of cosine-score-based hierarchies

## Evidence
