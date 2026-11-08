---
name: molecular-formula-reranking-zodiac
description: Use when features from an untargeted LC-MS/MS run have candidate formulas
  ranked by SIRIUS fragmentation trees but top-ranked candidates may be wrong due
  to interference, and before fingerprint prediction via CSI:FingerID.
when_to_use_negative:
- When the dataset has very few features or no correlated MS/MS patterns — ZODIAC's
  joint re-scoring relies on cross-feature evidence and may not improve on SIRIUS-only
  rankings.
- When you only need a first-pass formula candidate and plan no downstream fingerprint
  prediction or tree-based analysis.
- When you want per-feature structural class only from spectral library matches (ClassyFire)
  without molecular formula assignment.
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: ZODIAC
  role: Re-ranks candidate molecular formulas produced by SIRIUS using correlation
    of fragmentation patterns across features in the dataset
  repo: https://github.com/biocore/q2-qemistree
- name: SIRIUS
  role: Generates fragmentation trees and initial molecular formula candidates that
    ZODIAC re-ranks
- name: q2-qemistree
  role: QIIME 2 plugin wrapping SIRIUS/ZODIAC/CSI:FingerID commands (rerank-molecular-formulas,
    compute-fragmentation-trees, predict-fingerprints)
  repo: https://github.com/biocore/q2-qemistree
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:a3dc7a70cfe682d1565b0fa5f3c43b1cf96e1bc80026f6fa5e210a2aa05775b1
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
  - QIIME 2
  - Qemistree
  - SIRIUS
  - ZODIAC
  - q2-qemistree
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# molecular-formula-reranking-zodiac

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reranks candidate molecular formulas for MS/MS features using ZODIAC (SIRIUS 4), correcting SIRIUS per-feature fragmentation-tree scores by exploiting correlation of fragmentation patterns across features in the same dataset to select the most probable formula.

## When to use

Use when features from an untargeted LC-MS/MS run have candidate formulas ranked by SIRIUS fragmentation trees but top-ranked candidates may be wrong due to interference, and before fingerprint prediction via CSI:FingerID.

## When NOT to use

- When the dataset has very few features or no correlated MS/MS patterns — ZODIAC's joint re-scoring relies on cross-feature evidence and may not improve on SIRIUS-only rankings.
- When you only need a first-pass formula candidate and plan no downstream fingerprint prediction or tree-based analysis.
- When you want per-feature structural class only from spectral library matches (ClassyFire) without molecular formula assignment.

## Inputs

- fragmentation_trees.qza (SiriusFolder artifact from q2-qemistree compute-fragmentation-trees)
- MGF file with MS1 and MS2 peak lists per feature (e.g. sirius.mgf)
- SIRIUS configuration used to build the trees (ionization mode, profile, ppm tolerance)

## Outputs

- molecular_formulas.qza (ZodiacFolder artifact with ZODIAC-reranked candidate formulas and scores)
- Intermediary artifact consumed downstream by qemistree predict-fingerprints (CSI:FingerID)

## How to apply

Run `qiime qemistree rerank-molecular-formulas` on a fragmentation_trees.qza artifact produced by `qiime qemistree compute-fragmentation-trees` (SIRIUS, e.g. --p-ppm-max 15, --p-profile orbitrap, --p-ions-considered '[M+H]+'). ZODIAC jointly re-scores all candidate formulas of all features using dataset-wide information (inter-feature correlations of MS/MS patterns) rather than per-feature evidence alone, and outputs a ZodiacFolder artifact. In the Qemistree evaluation-dataset workflow the intermediate molecular_formulas.qza is then retained for features with ZODIAC score > 0.9841 and precursor m/z < 600 before being passed to `qiime qemistree predict-fingerprints` (--p-ppm-max 20). Treat the ZODIAC-ranked formula artifact as a quality gate: features whose top candidate remains low-confidence should not silently propagate into fingerprint predictions.

## Related tools

- **ZODIAC** (Re-ranks candidate molecular formulas produced by SIRIUS using correlation of fragmentation patterns across features in the dataset) — https://github.com/biocore/q2-qemistree
- **SIRIUS** (Generates fragmentation trees and initial molecular formula candidates that ZODIAC re-ranks)
- **q2-qemistree** (QIIME 2 plugin wrapping SIRIUS/ZODIAC/CSI:FingerID commands (rerank-molecular-formulas, compute-fragmentation-trees, predict-fingerprints)) — https://github.com/biocore/q2-qemistree

## Examples

```
qiime qemistree rerank-molecular-formulas --i-csi-results fragmentation_trees.qza --o-formula-features molecular_formulas.qza
```

## Evaluation signals

- Output artifact is a ZodiacFolder type and is accepted by `qiime qemistree predict-fingerprints` without type error.
- ZODIAC scores are attached per candidate formula; in the evaluation-dataset workflow only candidates with score > 0.9841 and precursor m/z < 600 are retained for fingerprint prediction.
- Number of features retained after reranking does not exceed number of features with fragmentation trees.
- Downstream fingerprints.qza (CSIFolder) is produced for a substantial subset of features — the article notes typically 70–90% coverage of MS1 features.
- Features whose top-ranked formula changed after ZODIAC should show improved CSI:FingerID substructure predictions rather than propagating SIRIUS-only errors.

## Limitations

- Qemistree was initially developed under SIRIUS 4.0.1, which reached end of life; the plugin was later adapted for SIRIUS > 4.4.29 — version mismatches can alter behavior.
- Formula reranking quality depends on the SIRIUS fragmentation trees being generated with correct parameters (e.g. --p-ppm-max 15, correct ionization mode such as [M+H]+, profile such as orbitrap).
- Only features with MS/MS data are considered; MS1-only features are not processed.
- In negative ionization mode, fewer molecular fingerprints can be confidently predicted due to fewer publicly available reference spectra, resulting in less-extensive trees.

## Evidence
