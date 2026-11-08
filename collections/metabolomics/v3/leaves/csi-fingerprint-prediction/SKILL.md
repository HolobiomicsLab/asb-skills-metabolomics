---
name: csi-fingerprint-prediction
description: Use when you have MS/MS fragmentation spectra (MGF) for chemical features
  and need predicted molecular fingerprints to compute pairwise chemical distances
  or build a Qemistree hierarchy.
when_to_use_negative:
- Features lack MS/MS data (MS1-only features are not considered and are filtered
  out of the feature table)
- The goal is spectral library match annotation only, and its coverage (typically
  1-20% of features) is deemed sufficient to summarize the sample's chemical content
- Working in negative ionization mode where too few fingerprints can be confidently
  predicted to build an informative tree
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SIRIUS
  role: De-novo determination of molecular formulas from isotope and fragmentation
    patterns and estimation of the best fragmentation trees per feature, which are
    the direct input for fingerprint prediction
  repo: https://github.com/biocore/q2-qemistree
- name: CSI:FingerID
  role: Predicts 2936 molecular properties (fingerprints) from the fragmentation trees
    using kernel support vector machines
- name: q2-qemistree
  role: QIIME 2 plugin wrapping SIRIUS/CSI:FingerID; provides compute-fragmentation-trees,
    rerank-molecular-formulas, and predict-fingerprints commands
  repo: https://github.com/biocore/q2-qemistree
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:14dd3eef5f2d4414b075f942c943788810ba79e5da1a499a98c84f1a9c452697
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
  - MZmine2
  - QIIME 2
  - Qemistree
  - SIRIUS
  - q2-qemistree
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# csi-fingerprint-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predict substructure-based molecular fingerprints for LC-MS/MS chemical features using SIRIUS fragmentation trees and CSI:FingerID kernel support vector machines, via the q2-qemistree QIIME 2 plugin. These fingerprints (2936 molecular properties per feature) underpin chemical hierarchy construction in Qemistree.

## When to use

Use when you have MS/MS fragmentation spectra (MGF) for chemical features and need predicted molecular fingerprints to compute pairwise chemical distances or build a Qemistree hierarchy.

## When NOT to use

- Features lack MS/MS data (MS1-only features are not considered and are filtered out of the feature table)
- The goal is spectral library match annotation only, and its coverage (typically 1-20% of features) is deemed sufficient to summarize the sample's chemical content
- Working in negative ionization mode where too few fingerprints can be confidently predicted to build an informative tree

## Inputs

- MGF file of MS/MS fragmentation spectra (only features with MS/MS data; MS1-only features are not considered)
- Feature table (e.g., MassIVE MSV000082969 Cheetah Quant table) as FeatureTable[Frequency] or feature-table.biom
- SIRIUS fragmentation trees (q2-qemistree compute-fragmentation-trees output)
- Reranked molecular formulas (qiime qemistree rerank-molecular-formulas output, --p-zodiac-threshold 0.95)

## Outputs

- Molecular fingerprints: probabilities of 2936 molecular properties per chemical feature (Supplementary Dataset 1)
- Fingerprint artifacts suitable for input to qiime qemistree make-hierarchy
- Downstream: rooted chemical hierarchy (Phylogeny[Rooted]) and hashed feature table after make-hierarchy

## How to apply

First run feature detection (e.g., MZmine2 on .mzXML/.mzML files) to produce an MGF of MS/MS spectra and a quant (feature) table. Use qiime qemistree compute-fragmentation-trees (SIRIUS; e.g., --p-ppm-max 15, --p-profile orbitrap, --p-ions-considered '[M+H]+') so SIRIUS determines each feature's molecular formula from isotope and fragmentation patterns and estimates the best fragmentation tree. Optionally rerank molecular formulas with qiime qemistree rerank-molecular-formulas (--p-zodiac-threshold 0.95) to select the top-scoring formula per feature. Then run qiime qemistree predict-fingerprints (e.g., --p-ppm-max 20) so CSI:FingerID operates on the fragmentation trees with kernel support vector machines to predict 2936 molecular properties per feature (Supplementary Dataset 1). The rationale is that fingerprint-based representations are largely independent of technical variation such as chromatography shifts, enabling tree-informed and cross-dataset comparisons without repeating feature detection or alignment.

## Related tools

- **SIRIUS** (De-novo determination of molecular formulas from isotope and fragmentation patterns and estimation of the best fragmentation trees per feature, which are the direct input for fingerprint prediction) — https://github.com/biocore/q2-qemistree
- **CSI:FingerID** (Predicts 2936 molecular properties (fingerprints) from the fragmentation trees using kernel support vector machines)
- **q2-qemistree** (QIIME 2 plugin wrapping SIRIUS/CSI:FingerID; provides compute-fragmentation-trees, rerank-molecular-formulas, and predict-fingerprints commands) — https://github.com/biocore/q2-qemistree

## Examples

```
qiime qemistree predict-fingerprints --p-ppm-max 20
```

## Evaluation signals

- The predict-fingerprints step yields probabilities for all 2936 molecular properties (Supplementary Dataset 1) per feature with a fragmentation tree
- Downstream make-hierarchy output tree is of type Phylogeny[Rooted]
- MS1 features without fingerprints are filtered out of the merged/hashed feature table
- Feature data after make-hierarchy contains columns featureID, parent_mass, retention_time, csi_smiles, ms2_smiles, and table_number
- Mass tolerance parameters match the instrument profile (e.g., --p-ppm-max 15/20 for orbitrap data)

## Limitations

- Fingerprint prediction depends on the quality and coverage of MS/MS spectral databases available for training the predictive models
- In negative ionization mode, fewer molecular fingerprints can be confidently predicted due to fewer publicly available reference spectra, resulting in less-extensive trees
- Qemistree shares limitations with the underlying fingerprint prediction tool CSI:FingerID
- Qemistree was developed under SIRIUS 4.0.1; since that version reached end of life, q2-qemistree was adapted to work with SIRIUS versions >4.4.29

## Evidence

- [methods] Predict molecular fingerprints using qiime qemistree predict-fingerprints with --p-ppm-max 20 to obtain probabilities of 2936 molecular properties per feature.: [evidence span withheld — anchor status: not_found]
