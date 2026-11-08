---
name: svm-confidence-score-training
description: Use when calibrating in silico structure annotation scores (e.g. CSI:FingerID)
  from reference-spectra training sets into posterior confidence scores for FDR estimation
  of novel-molecule annotations.
when_to_use_negative:
- The input is already a set of calibrated confidence scores or posterior probabilities
  rather than raw candidate scores needing calibration.
- You need to validate known library compounds by spectral library search rather than
  score and rank novel in silico structure candidates.
- The goal is establishing absolute or stereochemical structure identity — COSMIC
  cannot distinguish small structure modifications or stereochemistry from MS/MS alone.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: COSMIC
  role: Confidence-score training and calibration workflow over CSI:FingerID candidate
    scores
  repo: github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Produces ranked structure candidates and raw candidate scores that the SVM
    calibration model is trained on
- name: SIRIUS 4
  role: Processes LC–MS/MS runs, performs de novo formula determination, and feeds
    features into COSMIC
  repo: github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:be8c4f82cfcb8320fefe1997ff938cde86efa42677407eefad2824618ab4a8ea
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COSMIC
  - CSI:FingerID
  - SIRIUS 4
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# svm-confidence-score-training

## Summary

Trains COSMIC's confidence-score model for CSI:FingerID structure annotations: kernel density P-value estimation of proxy-decoy hit scores, per-candidate linear SVMs with enforced feature directionality, and Platt scaling to posterior probabilities.

## When to use

Use when calibrating in silico structure annotation scores (e.g. CSI:FingerID) from reference-spectra training sets into posterior confidence scores for FDR estimation of novel-molecule annotations.

## When NOT to use

- The input is already a set of calibrated confidence scores or posterior probabilities rather than raw candidate scores needing calibration.
- You need to validate known library compounds by spectral library search rather than score and rank novel in silico structure candidates.
- The goal is establishing absolute or stereochemical structure identity — COSMIC cannot distinguish small structure modifications or stereochemistry from MS/MS alone.

## Inputs

- Combined reference-spectra training set (MassBank, GNPS, NIST 2017) with CSI:FingerID hit and proxy-decoy candidate scores
- Fragmentation spectra grouped by collision energy (e.g. 20/40/60 eV, merged)
- Candidate score distributions per feature

## Outputs

- Trained single-candidate linear SVMs per collision energy with nested-CV-selected C
- Platt-scaled posterior probability estimates (confidence scores) for structure annotations
- FDR estimates for the top k hits derived from confidence scores

## How to apply

From a combined reference-spectra training set (MassBank, GNPS and the NIST 2017 database) with candidate scores for true hits and proxy decoys, split features by collision energy (e.g. 20/40/60 eV plus merged spectra). For each feature, fit a kernel density estimate to the proxy-decoy score distribution to convert raw candidate scores into P values. Assemble the P-value feature vectors and enforce directionality of every feature so each can only push the SVM decision toward the correct-annotation side. Train one linear SVM per candidate (single-candidate SVMs) per collision energy, selecting the regularization parameter C from {10^-5, ..., 10^5} by nested cross-validation. Map the resulting SVM decision values to posterior probability estimates using Platt probabilities; scores at or above the reported 0.64 threshold roughly correspond to 10% FDR on independent-data evaluation.

## Related tools

- **COSMIC** (Confidence-score training and calibration workflow over CSI:FingerID candidate scores) — github.com/sirius-ms/sirius
- **CSI:FingerID** (Produces ranked structure candidates and raw candidate scores that the SVM calibration model is trained on)
- **SIRIUS 4** (Processes LC–MS/MS runs, performs de novo formula determination, and feeds features into COSMIC) — github.com/sirius-ms/sirius

## Examples

```
python cosmic_confidence_scores.py --reference-spectra massbank_gnps_nist2017 --collision-energy 40 --svm-C nested-cv --platt-scaling
```

## Evaluation signals

- Confidence score threshold of 0.64 roughly corresponds to FDR 10% on independent-data evaluation
- All features must be directionally enforced so each can only push the SVM score toward the correct-annotation side
- Regularization parameter C is chosen by nested cross-validation from {10^-5, 10^-4, ..., 10^5}
- Per-candidate single-candidate SVMs trained per collision energy (20/40/60 eV, merged)

## Limitations

- COSMIC's confidence score must not be mistaken as the probability that an annotation is correct; impossible by design of the score.
- Accurate FDR estimation from fragmentation spectra of small molecules is expected to remain highly challenging; the 0.64 threshold is practical guidance only, not a guarantee.
- Stereochemistry cannot be established from fragmentation spectra; only the two-dimensional structure is considered.
- Small structure modifications are hard or potentially impossible to tell apart using MS/MS data alone.

## Evidence

- [abstract] Kernel density P-value estimation step: "a confidence score consisting of kernel density P value estimation and a support vector machine with enforced directionality of features"
- [discussion] Platt scaling step: "We map decision values to posterior probability estimates using Platt probabilities31."
- [discussion] Nested cross-validation of SVM parameter C: "Parameter C ∈ {10−5, 10−4, . . . , 105} of each SVM was chosen by a nested cross-validation."
- [discussion] Confidence-score to FDR calibration threshold: "We have used a threshold of 0.64 for the confidence score, roughly corresponding to FDR 10%"
- [abstract] Enforced feature directionality rationale: "a support vector machine with enforced directionality of features"
- [readme] SIRIUS 4 role in the workflow: "SIRIUS integrates a collection of our tools, including CSI:FingerID (with [COSMIC](https://bio.informatik.uni-jena.de/software/cosmic/)), [ZODIAC](https://bio.informatik.uni-jena.de/software/zodiac/),"
