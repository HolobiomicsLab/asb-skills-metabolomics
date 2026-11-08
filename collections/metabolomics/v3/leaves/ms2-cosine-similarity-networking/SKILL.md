---
name: ms2-cosine-similarity-networking
description: Use when you have GNPS feature-based molecular networking spectral matches
  (mzML/mzXML-derived feature tables plus paired MS/MS spectra) and need a library-match-only
  chemical tree to compare sample chemistries against fingerprint-predicted or tree-agnostic
  metrics.
when_to_use_negative:
- Input features lack MS/MS data (MS1-only features cannot be placed on a cosine-score
  hierarchy and would be filtered out)
- Only a small fraction of features have library matches and you need dense tree coverage
  — fingerprint prediction from SIRIUS/CSI:FingerID is the better backbone
- You need to compare independently processed datasets without re-running feature
  detection and alignment — fingerprint-based representation handles chromatography
  shifts better
edam_operation: http://edamontology.org/operation_3638
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: GNPS
  role: Provides feature-based molecular networking spectral matches used to seed
    the cosine-score hierarchy
- name: QIIME 2
  role: Imports GNPS matches as FeatureData[Molecules] and hosts the weighted UniFrac
    metric used to compare samples on the tree
- name: Qemistree
  role: Companion workflow whose fingerprint-based hierarchy is compared against this
    cosine-score hierarchy in Extended Data Table 4
  repo: github.com/biocore/q2-qemistree
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:1681edcab8ba960a0ebe0da1f7be652cb33eb4850821d946c218b7b79a7c728a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/biocore/q2-qemistree
  tools:
  - CSI:FingerID
  - GNPS
  - Python
  - QIIME 2
  - Qemistree
  - R
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3638
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# ms2-cosine-similarity-networking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds an MS/MS cosine-score-based chemical hierarchy by importing GNPS feature-based molecular networking spectral matches as FeatureData[Molecules] and computing weighted UniFrac-style sample dissimilarities, providing a library-match-only comparator to fingerprint-based Qemistree hierarchies.

## When to use

Use when you have GNPS feature-based molecular networking spectral matches (mzML/mzXML-derived feature tables plus paired MS/MS spectra) and need a library-match-only chemical tree to compare sample chemistries against fingerprint-predicted or tree-agnostic metrics.

## When NOT to use

- Input features lack MS/MS data (MS1-only features cannot be placed on a cosine-score hierarchy and would be filtered out)
- Only a small fraction of features have library matches and you need dense tree coverage — fingerprint prediction from SIRIUS/CSI:FingerID is the better backbone
- You need to compare independently processed datasets without re-running feature detection and alignment — fingerprint-based representation handles chromatography shifts better

## Inputs

- GNPS feature-based molecular networking spectral matches exported as FeatureData[Molecules]
- Feature table (BIOM) of MS1 features with peak areas
- Sample metadata with food ontology (or other) categorical levels
- Paired MS/MS spectra (MGF from feature detection)

## Outputs

- MS/MS cosine-score-based chemical hierarchy (Newick tree over matched features)
- Pairwise weighted-UniFrac sample dissimilarity matrix computed on the cosine-score hierarchy
- Per-food-ontology-level comparison table (extended_data_table4_metric_comparison.csv) with counts of chemically meaningful clusterings per metric

## How to apply

Process LC-MS/MS data through feature detection (e.g., MZmine2) to obtain a feature table and GNPS molecular networking job; import the resulting GNPS matches as FeatureData[Molecules] in QIIME 2, which represent only the subset of features with spectral library annotations (typically 1–20% of all features, depending on how well the sample type has been investigated). Build a chemical hierarchy from these matched molecules using cosine similarity of their MS/MS fragmentation spectra, then compute pairwise sample dissimilarities with weighted UniFrac on this cosine-score hierarchy. In the Global FoodOmics benchmark (N=126 samples), this was run alongside weighted UniFrac on the fingerprint hierarchy, Bray-Curtis on the feature table, and CSCS across food ontology levels 1–4, and the results were compiled into extended_data_table4_metric_comparison.csv to compare how many food categories each metric resolved as chemically coherent clusters. Because spectral matches are sparse and skewed toward well-studied chemical space, treat this hierarchy as a lower-coverage but more accurate comparator rather than a standalone clustering backbone.

## Related tools

- **GNPS** (Provides feature-based molecular networking spectral matches used to seed the cosine-score hierarchy)
- **QIIME 2** (Imports GNPS matches as FeatureData[Molecules] and hosts the weighted UniFrac metric used to compare samples on the tree)
- **Qemistree** (Companion workflow whose fingerprint-based hierarchy is compared against this cosine-score hierarchy in Extended Data Table 4) — github.com/biocore/q2-qemistree

## Examples

```
qiime qemistree make-hierarchy --i-paired-feature-tables feature-table.biom --i-ms2-matches gnps-matches.qza --o-hierarchy ms2-cosine-tree.qza
```

## Evaluation signals

- Confirm the cosine-score tree contains only features with MS/MS library matches — MS1-only features without fingerprints must be filtered out of the feature table before weighted UniFrac
- Cross-check the proportion of library-matched features against the expected 1–20% range for the sample type; very low coverage invalidates the comparator
- Reproduce the Extended Data Table 4 pattern: fingerprint-based weighted UniFrac should support chemically meaningful clustering at least as well as the tree-agnostic Bray-Curtis across food ontology levels 1–4
- Verify the cosine-score hierarchy yields a valid, fully resolved Newick tree over the matched features
- Check that the pairwise dissimilarity matrix covers all N=126 Global FoodOmics samples with no missing pairs

## Limitations

- Typically only 1–20% of features have spectral library matches, so the cosine-score hierarchy covers a small, biased subset of the chemical space
- Spectral match annotations have coverage too low to adequately summarize the chemical content of complex samples
- In negative ionization mode, fewer matches and fingerprints can be confidently predicted, resulting in less-extensive trees
- A broader benchmarking effort would be needed to understand when each tree-building approach (fingerprint vs. cosine-score) should be used

## Evidence

- [methods] Benchmark of four sample-dissimilarity metrics: [evidence span withheld — anchor status: not_found]
