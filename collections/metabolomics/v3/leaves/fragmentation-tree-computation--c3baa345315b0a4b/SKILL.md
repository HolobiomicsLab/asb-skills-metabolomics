---
name: fragmentation-tree-computation
description: Use when a chemical-feature list (MGF with MS1 and MS2 peaks from MZmine2)
  must be converted into molecular formulas, fragmentation trees, and predicted fingerprints
  before building a chemical tree with q2-qemistree make-hierarchy.
when_to_use_negative:
- Chemical features have only MS1 data and no MS/MS spectra — the paper states 'Only
  chemical features with MS/MS data are included; features with only MS1 are not considered.'
- The goal is library-based annotation rather than in silico prediction — spectral
  matches alone cover too little (typically 1–20% of features) and cannot be substituted
  for fingerprint prediction when building the hierarchy.
- The downstream tree/hierarchy already exists and the task is only pruning or visualization
  (use prune-hierarchy or Empress/iTOL instead of re-running SIRIUS).
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_3520
tools:
- name: QIIME 2 / q2-qemistree
  role: Wraps SIRIUS, ZODIAC and CSI:FingerID as the compute-fragmentation-trees,
    rerank-molecular-formulas and predict-fingerprints actions
  repo: https://github.com/biocore/q2-qemistree
- name: SIRIUS
  role: Determines each feature's molecular formula from isotope and fragmentation
    patterns and estimates the best fragmentation tree explaining the fragmentation
    spectrum
  repo: https://bio.informatik.uni-jena.de/sirius/
- name: ZODIAC
  role: Re-ranks candidate molecular formulas from the fragmentation trees (e.g. ZODIAC
    score > 0.9841 in the evaluation dataset) before fingerprint prediction
- name: CSI:FingerID
  role: Operates on the fragmentation trees with kernel support vector machines to
    predict 2936 molecular properties per chemical feature
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:fa8db0979c8545c9585e566c77cc96a6fc41d7521c6a70cb634265e4abcc1b54
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
  - QIIME 2 / q2-qemistree
  - Qemistree
  - SIRIUS
  - ZODIAC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# fragmentation-tree-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-feature molecular formulas and fragmentation trees from untargeted LC-MS/MS spectra (MGF) using SIRIUS, optionally re-ranks candidate formulas with ZODIAC, and feeds the trees to CSI:FingerID to predict 2936-property molecular fingerprints that underpin the Qemistree chemical hierarchy.

## When to use

Use when a chemical-feature list (MGF with MS1 and MS2 peaks from MZmine2) must be converted into molecular formulas, fragmentation trees, and predicted fingerprints before building a chemical tree with q2-qemistree make-hierarchy.

## When NOT to use

- Chemical features have only MS1 data and no MS/MS spectra — the paper states 'Only chemical features with MS/MS data are included; features with only MS1 are not considered.'
- The goal is library-based annotation rather than in silico prediction — spectral matches alone cover too little (typically 1–20% of features) and cannot be substituted for fingerprint prediction when building the hierarchy.
- The downstream tree/hierarchy already exists and the task is only pruning or visualization (use prune-hierarchy or Empress/iTOL instead of re-running SIRIUS).

## Inputs

- MGF file with MS1 + MS2 peaks (e.g. sirius.mgf from MZmine2 preprocessing of .mzXML/.mzML/.mzDATA)
- feature-table.biom (FeatureTable[Frequency]) of MS1 chemical features
- QIIME 2 artifact of type MassSpectrometryFeatures

## Outputs

- fragmentation_trees.qza (SiriusFolder artifact)
- molecular_formulas.qza (ZodiacFolder artifact, after ZODIAC reranking)
- fingerprints.qza (CSIFolder artifact with substructure probabilities for 2936 molecular properties per feature)

## How to apply

Import the MGF into QIIME 2 as a MassSpectrometryFeatures artifact, verifying every MS1 entry has a corresponding MS2 entry. Run 'qiime qemistree compute-fragmentation-trees' with SIRIUS (originally 4.0.1; q2-qemistree now also supports SIRIUS >4.4.29) using --p-ppm-max 15, --p-profile orbitrap, and the ion mode, e.g. --p-ions-considered '[M+H]+', with JVM flags -Xms16G -Xmx64G and a writable temp directory to produce fragmentation_trees.qza (SiriusFolder). Optionally run 'qiime qemistree rerank-molecular-formulas' (ZODIAC, e.g. --p-zodiac-threshold 0.95) to select the top-scoring molecular formula per feature. Then run 'qiime qemistree predict-fingerprints' with --p-ppm-max 20 so CSI:FingerID applies kernel support vector machines to the fragmentation trees, predicting the 2936 molecular properties (Supplementary Dataset 1) into fingerprints.qza (CSIFolder). The rationale is that fingerprint vectors — not low-coverage spectral matches — let Qemistree calculate pairwise chemical relationships and hierarchically cluster features, while being largely independent of chromatography shifts.

## Related tools

- **QIIME 2 / q2-qemistree** (Wraps SIRIUS, ZODIAC and CSI:FingerID as the compute-fragmentation-trees, rerank-molecular-formulas and predict-fingerprints actions) — https://github.com/biocore/q2-qemistree
- **SIRIUS** (Determines each feature's molecular formula from isotope and fragmentation patterns and estimates the best fragmentation tree explaining the fragmentation spectrum) — https://bio.informatik.uni-jena.de/sirius/
- **ZODIAC** (Re-ranks candidate molecular formulas from the fragmentation trees (e.g. ZODIAC score > 0.9841 in the evaluation dataset) before fingerprint prediction)
- **CSI:FingerID** (Operates on the fragmentation trees with kernel support vector machines to predict 2936 molecular properties per chemical feature)

## Examples

```
qiime qemistree compute-fragmentation-trees --i-spectra sirius.mgf --p-ppm-max 15 --p-profile orbitrap --p-ions-considered '[M+H]+' --o-fragmentation-trees fragmentation_trees.qza
```

## Evaluation signals

- fragmentation_trees.qza exists as a SiriusFolder artifact and molecular_formulas.qza as a ZodiacFolder artifact for every processed feature
- Every MS1 entry in the imported MGF has a corresponding MS2 entry before tree computation
- Reranked formulas satisfy the dataset filter (evaluation dataset: ZODIAC score > 0.9841 and precursor m/z < 600; cheetah workflow: --p-zodiac-threshold 0.95)
- fingerprints.qza is a CSIFolder artifact containing substructure probabilities spanning all 2936 molecular properties per predicted feature
- Fingerprint coverage falls in the expected ~70–90% of MS1 features; features without fingerprints are filtered out of the merged feature table in make-hierarchy

## Limitations

- Only chemical features with MS/MS data are eligible; features with only MS1 are excluded, so coverage of the full feature table is inherently partial (~70–90%).
- In negative ionization mode, fewer molecular fingerprints can be confidently predicted due to fewer publicly available reference spectra, resulting in less-extensive trees.
- Fingerprint prediction depends on the quality and coverage of MS/MS spectral databases used to train the CSI:FingerID models; SIRIUS results also depend on the PubChem version bundled (13 August 2017 release).
- Qemistree was initially developed under SIRIUS 4.0.1, which reached end of life; q2-qemistree has since been adapted to SIRIUS versions >4.4.29, so parameter behavior may differ across versions.

## Evidence

- [methods] SIRIUS PubChem version dependency: [evidence span withheld — anchor status: not_found]
- [readme] q2-qemistree README on SIRIUS dependency: [evidence span withheld — anchor status: not_found]
