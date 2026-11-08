---
name: orbitrap-adduct-ppm-parameterization
description: Use when running q2-qemistree on Orbitrap-acquired MS/MS data (MGF with
  MS1+MS2 peaks) and you must choose --p-profile, --p-ions-considered, and --p-ppm-max
  values for compute-fragmentation-trees, rerank-molecular-formulas, and predict-fingerprints.
when_to_use_negative:
- Data were acquired on an instrument whose mass accuracy does not match the Orbitrap
  profile setting (the profile must match the MS platform).
- Input features lack MS/MS spectra — only chemical features with MS/MS data are included;
  MS1-only features are not considered.
- You only need spectral library matching (e.g. GNPS) rather than in silico fingerprint
  prediction; note that spectral matches typically cover only 1–20% of features.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: q2-qemistree
  role: QIIME 2 plugin exposing compute-fragmentation-trees, rerank-molecular-formulas,
    and predict-fingerprints with profile/adduct/ppm parameters
  repo: https://github.com/biocore/q2-qemistree
- name: SIRIUS
  role: Determines molecular formulas from isotope and fragmentation patterns and
    estimates the best fragmentation tree, run with --p-profile orbitrap and --p-ppm-max
    15
  repo: https://www.nature.com/articles/s41592-019-0344-8
- name: ZODIAC
  role: Re-ranks candidate molecular formulas on the fragmentation trees (rerank-molecular-formulas
    step)
- name: CSI:FingerID
  role: Predicts 2936 molecular properties from fragmentation trees via kernel support
    vector machines at --p-ppm-max 20
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:08f9afeacf32f91abb6ba224ef305c09bf5b6f6b107fefda0f5a1ab9f814c70b
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
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# orbitract-adduct-ppm-parameterization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parameterize the SIRIUS/ZODIAC/CSI:FingerID fingerprint-prediction stages of Qemistree for Orbitrap data: set instrument profile, precursor ion adduct, and per-stage ppm tolerances so fragmentation trees, formula re-ranking, and 2936-property fingerprints are computed under validated settings.

## When to use

Use when running q2-qemistree on Orbitrap-acquired MS/MS data (MGF with MS1+MS2 peaks) and you must choose --p-profile, --p-ions-considered, and --p-ppm-max values for compute-fragmentation-trees, rerank-molecular-formulas, and predict-fingerprints.

## When NOT to use

- Data were acquired on an instrument whose mass accuracy does not match the Orbitrap profile setting (the profile must match the MS platform).
- Input features lack MS/MS spectra — only chemical features with MS/MS data are included; MS1-only features are not considered.
- You only need spectral library matching (e.g. GNPS) rather than in silico fingerprint prediction; note that spectral matches typically cover only 1–20% of features.

## Inputs

- MGF file of MS1 + MS2 peaks imported as a QIIME 2 MassSpectrometryFeatures artifact (e.g. sirius.mgf)
- Feature table (.biom) with chemical features from MZmine2 pre-processing

## Outputs

- fragmentation_trees.qza (SiriusFolder artifact of estimated fragmentation trees)
- molecular_formulas.qza (ZodiacFolder artifact of ZODIAC-reranked molecular formulas)
- fingerprints.qza (CSIFolder artifact with 2936 molecular-property substructure probabilities per feature)

## How to apply

After importing the MGF into QIIME 2 as MassSpectrometryFeatures (verifying every MS1 entry has a paired MS2 entry), run 'qiime qemistree compute-fragmentation-trees' with SIRIUS 4.0.1 using --p-ppm-max 15, --p-profile orbitrap, and --p-ions-considered '[M]+H' (i.e. [M+H]+), with JVM flags -Xms16G -Xmx64G and a writable temp directory, producing fragmentation_trees.qza. Then run 'qiime qemistree rerank-molecular-formulas' (ZODIAC) on the trees to re-rank candidate molecular formulas. Finally run 'qiime qemistree predict-fingerprints' with --p-ppm-max 20 to predict the 2936-property fingerprint vector per feature. The rationale is that the ppm tolerance and instrument profile determine how SIRIUS scores isotope and fragmentation patterns, so they must match the Orbitrap mass accuracy; the looser 20 ppm in the fingerprint stage accommodates the formula-reranked candidates from ZODIAC.

## Related tools

- **q2-qemistree** (QIIME 2 plugin exposing compute-fragmentation-trees, rerank-molecular-formulas, and predict-fingerprints with profile/adduct/ppm parameters) — https://github.com/biocore/q2-qemistree
- **SIRIUS** (Determines molecular formulas from isotope and fragmentation patterns and estimates the best fragmentation tree, run with --p-profile orbitrap and --p-ppm-max 15) — https://www.nature.com/articles/s41592-019-0344-8
- **ZODIAC** (Re-ranks candidate molecular formulas on the fragmentation trees (rerank-molecular-formulas step))
- **CSI:FingerID** (Predicts 2936 molecular properties from fragmentation trees via kernel support vector machines at --p-ppm-max 20)

## Examples

```
qiime qemistree compute-fragmentation-trees --i-features sirius.qza --p-ppm-max 15 --p-profile orbitrap --p-ions-considered '[M+H]+' --o-fragmentation-trees fragmentation_trees.qza
```

## Evaluation signals

- fingerprints.qza is a CSIFolder artifact containing substructure probabilities spanning all 2936 molecular properties per predicted feature.
- Every MS1 entry in the imported MGF has a corresponding MS2 entry before tree computation.
- fragmentation_trees.qza (SiriusFolder) and molecular_formulas.qza (ZodiacFolder) exist as valid intermediate artifacts.
- Fingerprint coverage of MS1 features falls in the expected ~70–90% range.
- Molecular-formula candidates retained after ZODIAC re-ranking meet the evaluation-dataset filters (ZODIAC score > 0.9841, precursor m/z < 600).

## Limitations

- Fingerprint prediction depends on the quality and coverage of MS/MS spectral databases available for training the predictive models.
- In negative ionization mode, fewer molecular fingerprints can be confidently predicted due to fewer publicly available reference spectra, resulting in less-extensive trees.
- Qemistree was initially developed under SIRIUS 4.0.1, which reached end of life; it was later adapted to work with SIRIUS versions >4.4.29.
- The latest SIRIUS release uses a PubChem version downloaded on 13 August 2017, bounding in silico formula candidates.

## Evidence

- [results] Run 'qiime qemistree compute-fragmentation-trees' with SIRIUS 4.0.1 using --p-ppm-max 15, --p-profile orbitrap, and --p-ions-considered '[M+H]+': [evidence span withheld — anchor status: not_found]
