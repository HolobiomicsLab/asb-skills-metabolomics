---
name: substructure-fingerprint-prediction
description: Use when untargeted LC-MS/MS chemical features (MGF with MS1+MS2 peaks
  imported as MassSpectrometryFeatures) need predicted molecular fingerprints to build
  a Qemistree chemical hierarchy, rather than relying on sparse spectral library matches.
when_to_use_negative:
- Features lacking MS/MS data (MS1-only features are not considered — they cannot
  receive fingerprints).
- Analysis goals relying on spectral library matches for structural annotation, when
  library-match coverage (typically only 1–20% of features) is instead the limiting
  factor and fingerprints are needed.
- Datasets requiring ClassyFire taxonomy assignment or hierarchy construction only
  — those are downstream steps (make-hierarchy, get-classyfire-taxonomy) that consume
  fingerprints.qza, not part of this prediction stage.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SIRIUS
  role: Determines the molecular formula of each feature from isotope and fragmentation
    patterns and estimates the best fragmentation tree explaining the fragmentation
    spectrum
  repo: https://github.com/biocore/q2-qemistree
- name: ZODIAC
  role: Re-ranks candidate molecular formulas per feature (qiime qemistree rerank-molecular-formulas)
    before fingerprint prediction
  repo: https://github.com/biocore/q2-qemistree
- name: CSI:FingerID
  role: Operates on the fragmentation trees using kernel support vector machines to
    predict the 2936 molecular properties constituting the fingerprint vector
  repo: https://github.com/biocore/q2-qemistree
- name: QIIME 2
  role: Hosts the q2-qemistree plugin commands (compute-fragmentation-trees, rerank-molecular-formulas,
    predict-fingerprints) and artifact types
  repo: https://github.com/biocore/q2-qemistree
- name: Qemistree
  role: Wraps SIRIUS/CSI:FingerID to obtain predicted molecular fingerprints used
    to build the chemical hierarchy
  repo: https://github.com/biocore/q2-qemistree
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:331ed6d768b2d4c6c3c97bb03249431dc8254995f5d3ad7f804409ac1ba94791
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
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# substructure-fingerprint-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs the SIRIUS → ZODIAC → CSI:FingerID stage of the Qemistree pipeline, in which fragmentation trees computed from MS/MS spectra are re-ranked for molecular formulas and then converted into 2936-property molecular substructure fingerprint vectors per chemical feature.

## When to use

Use when untargeted LC-MS/MS chemical features (MGF with MS1+MS2 peaks imported as MassSpectrometryFeatures) need predicted molecular fingerprints to build a Qemistree chemical hierarchy, rather than relying on sparse spectral library matches.

## When NOT to use

- Features lacking MS/MS data (MS1-only features are not considered — they cannot receive fingerprints).
- Analysis goals relying on spectral library matches for structural annotation, when library-match coverage (typically only 1–20% of features) is instead the limiting factor and fingerprints are needed.
- Datasets requiring ClassyFire taxonomy assignment or hierarchy construction only — those are downstream steps (make-hierarchy, get-classyfire-taxonomy) that consume fingerprints.qza, not part of this prediction stage.

## Inputs

- MGF file with MS1 + MS2 peaks for chemical features (from MZmine2 feature detection)
- QIIME 2 MassSpectrometryFeatures artifact
- SIRIUS installation (4.0.1 originally; >4.4.29 supported in recent q2-qemistree)
- PubChem reference database (SIRIUS release of 13 August 2017 used in the paper)

## Outputs

- fragmentation_trees.qza (SiriusFolder artifact)
- molecular_formulas.qza (ZodiacFolder artifact, ZODIAC-re-ranked formulas)
- fingerprints.qza (CSIFolder artifact with substructure probabilities spanning all 2936 molecular properties per feature)

## How to apply

Import the MGF (MS1 + MS2 peaks) into QIIME 2 with 'qiime tools import --type MassSpectrometryFeatures', confirming each MS1 entry has a corresponding MS2 entry since features without MS/MS are excluded. Run 'qiime qemistree compute-fragmentation-trees' with SIRIUS 4.0.1 (--p-ppm-max 15, --p-profile orbitrap, --p-ions-considered '[M+H]+', JVM flags -Xms16G -Xmx64G and a writable temp directory) to obtain fragmentation_trees.qza (SiriusFolder). Run 'qiime qemistree rerank-molecular-formulas' (ZODIAC) to re-rank candidate formulas, retaining those with ZODIAC score > 0.9841 and precursor m/z < 600 for the evaluation dataset, producing molecular_formulas.qza (ZodiacFolder). Finally run 'qiime qemistree predict-fingerprints' (CSI:FingerID) with --p-ppm-max 20, which applies kernel support vector machines to the fragmentation trees to predict the 2936 molecular properties (Supplementary Dataset 1) per feature, saving fingerprints.qza (CSIFolder).

## Related tools

- **SIRIUS** (Determines the molecular formula of each feature from isotope and fragmentation patterns and estimates the best fragmentation tree explaining the fragmentation spectrum) — https://github.com/biocore/q2-qemistree
- **ZODIAC** (Re-ranks candidate molecular formulas per feature (qiime qemistree rerank-molecular-formulas) before fingerprint prediction) — https://github.com/biocore/q2-qemistree
- **CSI:FingerID** (Operates on the fragmentation trees using kernel support vector machines to predict the 2936 molecular properties constituting the fingerprint vector) — https://github.com/biocore/q2-qemistree
- **QIIME 2** (Hosts the q2-qemistree plugin commands (compute-fragmentation-trees, rerank-molecular-formulas, predict-fingerprints) and artifact types) — https://github.com/biocore/q2-qemistree
- **Qemistree** (Wraps SIRIUS/CSI:FingerID to obtain predicted molecular fingerprints used to build the chemical hierarchy) — https://github.com/biocore/q2-qemistree

## Examples

```
qiime qemistree compute-fragmentation-trees --i-spectra sirius.mgf --p-ppm-max 15 --p-profile orbitrap --p-ions-considered '[M+H]+' --o-fragmentation-trees fragmentation_trees.qza && qiime qemistree predict-fingerprints --i-molecular-formulas molecular_formulas.qza --p-ppm-max 20 --o-fingerprints fingerprints.qza
```

## Evaluation signals

- fingerprints.qza is a CSIFolder artifact containing substructure probabilities spanning all 2936 molecular properties for each predicted feature.
- Intermediate artifacts fragmentation_trees.qza (SiriusFolder) and molecular_formulas.qza (ZodiacFolder) are present from the preceding steps.
- Fingerprint coverage of MS1 features falls in the expected ~70–90% range.
- Only features with MS/MS data receive fingerprints; MS1-only features are absent (and later filtered from the feature table).
- ZODIAC re-ranking respected the evaluation-dataset filters (ZODIAC score > 0.9841, precursor m/z < 600) when those thresholds are in scope.

## Limitations

- Qemistree depends on representing chemical features as molecular fingerprints and shares limitations with the underlying fingerprint prediction tool CSI:FingerID.
- Fingerprint prediction depends on the quality and coverage of MS/MS spectral databases available for training the predictive models.
- In negative ionization mode, fewer molecular fingerprints can be confidently predicted due to fewer publicly available reference spectra, resulting in less-extensive trees.
- Only 1–20% of features typically have spectral library matches, which is why fingerprints (with their own uncertainty) are used for hierarchical representation instead.
- Qemistree was initially developed under SIRIUS 4.0.1; since 4.0.1 reached end of life, q2-qemistree was adapted for SIRIUS >4.4.29.

## Evidence
