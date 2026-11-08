---
name: biological-fingerprint-pearson-similarity
description: Use when scoring co-occurring MS features by consistency of their bioactivity
  across assays — e.g., to compute NP Analyst Cluster Scores from a basketed feature
  table paired with a normalized bioassay matrix.
when_to_use_negative:
- Bioassay columns are not yet normalized to a common scale — fingerprints and correlations
  will be invalid until each column uses the same scale
- The goal is grouping samples by their activity profiles — NP Analyst communities
  are created based on bioactive MS feature distribution, not sample activity profiles,
  so this feature-level similarity is not a sample clustering method
- The input is raw, unbasketed mass peak lists — replicate noise peaks should first
  be removed (e.g., replicate comparison requiring presence in n − 1 of n replicates)
  before fingerprinting
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0092
tools:
- name: NP Analyst
  role: 'Platform whose feature-scoring algorithm this skill reimplements: computes
    Activity and Cluster Scores from integrated MS feature and bioassay data'
  repo: https://github.com/liningtonlab/npanalyst
- name: MZmine 2
  role: Source of processed MS feature (peak list) tables that provide the co-occurrence
    structure over which fingerprints are correlated
- name: GNPS
  role: Source of molecular network files whose features feed the fingerprint computation
    after import
- name: Tableau
  role: Third-party visualization of the scored feature table (scatter, network, and
    community views) after Cluster Scores are computed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acscentsci.1c01108
    title: 'NP Analyst: An Open Online Platform for Compound Activity Mapping'
  source_content_hash: sha256:5ce74c10b8c05833df818f0972bf151777c35797c805e72a5a2cc277c0396b7d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liningtonlab/npanalyst
  tools:
  - GNPS
  - MZmine 2
  - NP Analyst
  - Tableau
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# biological-fingerprint-pearson-similarity

## Summary

Scores the consistency of each MS feature's predicted biological profile by computing Pearson correlations between binary bioactivity fingerprints of co-occurring MS features, as used in NP Analyst's Cluster Score.

## When to use

Use when scoring co-occurring MS features by consistency of their bioactivity across assays — e.g., to compute NP Analyst Cluster Scores from a basketed feature table paired with a normalized bioassay matrix.

## When NOT to use

- Bioassay columns are not yet normalized to a common scale — fingerprints and correlations will be invalid until each column uses the same scale
- The goal is grouping samples by their activity profiles — NP Analyst communities are created based on bioactive MS feature distribution, not sample activity profiles, so this feature-level similarity is not a sample clustering method
- The input is raw, unbasketed mass peak lists — replicate noise peaks should first be removed (e.g., replicate comparison requiring presence in n − 1 of n replicates) before fingerprinting

## Inputs

- Basketed/aligned MS feature table (features × samples CSV with retention time and m/z columns, e.g., from MZmine 2 or a GNPS network file import)
- Normalized bioassay matrix (samples × bioassay columns, all columns on the same scale)

## Outputs

- Cluster Score per MS feature (mean Pearson similarity to co-active feature partners)
- Scored feature table (e.g., scored_features.csv) with Activity Score and Cluster Score columns appended

## How to apply

First assemble the biological profiles: normalize the bioassay CSV so that each assay column uses the same scale, then binarize each MS feature's activities to produce its biological fingerprint (the set of assays in which that feature is active). For each pair of MS features that co-occur in at least one sample, compute the Pearson correlation of their binary biological fingerprints. The Cluster Score for each feature is the mean Pearson similarity across its co-active feature partners; append it (alongside the Activity Score, the sum over assays of the mean squared normalized bioactivity values for samples containing the feature) to the feature table. Because NP Analyst scores each MS feature independently, all features belonging to the same molecule (adduct/fragment stripes at the same retention time) will carry similar Activity and Cluster Scores but different m/z values, so interpret stripes together rather than expecting a single winner feature.

## Related tools

- **NP Analyst** (Platform whose feature-scoring algorithm this skill reimplements: computes Activity and Cluster Scores from integrated MS feature and bioassay data) — https://github.com/liningtonlab/npanalyst
- **MZmine 2** (Source of processed MS feature (peak list) tables that provide the co-occurrence structure over which fingerprints are correlated)
- **GNPS** (Source of molecular network files whose features feed the fingerprint computation after import)
- **Tableau** (Third-party visualization of the scored feature table (scatter, network, and community views) after Cluster Scores are computed)

## Examples

```
import pandas as pd; sim = fingerprints.T.corr(method='pearson'); cluster_score = sim.mean(axis=1)
```

## Evaluation signals

- Cluster Score must lie within the Pearson r bounds [-1, 1] for every feature; features with no co-occurring partners have no defined cluster score
- All MS features from the same bioactive molecule (same retention-time stripe, different m/z) should receive similar Activity and Cluster Scores
- A feature's fingerprint must only span assays for samples in which that feature is present — co-occurrence in at least one sample is the pairing criterion
- Downstream, the scored table should support NP Analyst's complementary scatter, network, and community views without contradictions (e.g., micromonolactam-containing features cluster together)

## Limitations

- NP Analyst scores each MS feature independently, requiring manual interpretation of adduct/fragment/multiply-charged feature stripes
- Communities lack clear individual bioactive MS features in some cases, requiring retention-time stripe strategies instead of single-feature scores
- Low signal-intensity cutoffs can dramatically increase network complexity without adding biologically informative MS features

## Evidence

- [discussion] Normalization prerequisite: "these data are suitable for assembly into biological profiles for use in NP Analyst, provided that the data are normalized so that each column uses the same scale"
- [discussion] Independent feature scoring: "Because NP Analyst scores each MS feature independently, all MS features from a given bioactive molecule will have similar Activity and Cluster Scores but different m/z ratios"
- [discussion] Community basis: "communities are created based on bioactive MS feature distribution, rather than sample activity profiles"
- [readme] Basketing of co-occurring features: "Using an r-tree to build connected component graphs of m/z features with overlapping error ranges (in all dimensions). These are then combined (averaged) and represent a replicated or basketed"
