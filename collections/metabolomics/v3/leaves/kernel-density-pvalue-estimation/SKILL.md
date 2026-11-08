---
name: kernel-density-pvalue-estimation
description: Use when raw in silico structure-candidate scores must be converted to
  calibrated P values so that correct vs. incorrect annotations can be separated —
  e.g. building COSMIC-style confidence scores from CSI:FingerID hits.
when_to_use_negative:
- You want a literal probability that an annotation is correct — COSMIC's confidence
  score 'must not be mistaken as the probability that an annotation is correct'
- Input is raw MS/MS spectra rather than candidate scores — de novo formula assignment
  (SIRIUS 4, 10 ppm) and CSI:FingerID ranking must run first
- Estimating FDR for annotations far outside the reference-spectra training distribution
  — the authors note accurate FDR estimation from small-molecule fragmentation spectra
  will remain highly challenging
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: COSMIC
  role: Workflow whose first step calibrates CSI:FingerID hit scores via KDE-estimated
    P values and directional linear SVMs with Platt scaling
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Supplies the raw candidate ranking scores that are converted to P values as
    SVM features
  repo: https://github.com/sirius-ms/sirius
- name: SIRIUS 4
  role: Processes mzML/mzXML LC-MS/MS runs to produce the fragmentation trees and
    candidate scores upstream of the confidence-score computation
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:8758b78b85caf48fc10452be642d6695f572b1888f889bcc0f150948aa979c59
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - COSMIC
  - CSI:FingerID
  - SIRIUS 4
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# kernel-density-pvalue-estimation

## Summary

Calibrates raw structure-candidate scores (e.g. from CSI:FingerID) by fitting a kernel density estimate to the proxy-decoy (wrong-candidate) score distribution, converting each hit score into a P value that becomes a direction-constrained feature for the COSMIC confidence-score SVM.

## When to use

Use when raw in silico structure-candidate scores must be converted to calibrated P values so that correct vs. incorrect annotations can be separated — e.g. building COSMIC-style confidence scores from CSI:FingerID hits.

## When NOT to use

- You want a literal probability that an annotation is correct — COSMIC's confidence score 'must not be mistaken as the probability that an annotation is correct'
- Input is raw MS/MS spectra rather than candidate scores — de novo formula assignment (SIRIUS 4, 10 ppm) and CSI:FingerID ranking must run first
- Estimating FDR for annotations far outside the reference-spectra training distribution — the authors note accurate FDR estimation from small-molecule fragmentation spectra will remain highly challenging

## Inputs

- Combined reference-spectra training set (MassBank, GNPS, NIST 2017) with CSI:FingerID candidate scores for hits and proxy decoys
- Per-collision-energy spectra splits (20/40/60 eV and merged)
- Ranked structure-candidate lists restricted to the molecular formula of the top-scoring hit

## Outputs

- Kernel-density-estimated P values for each hit score per feature
- Direction-constrained feature vectors feeding the single-candidate linear SVMs
- Calibrated COSMIC confidence scores (cosmic_confidence_scores.csv)

## How to apply

Load the combined reference-spectra training set (MassBank, GNPS, NIST 2017) with candidate scores for true hits and proxy decoys, and split the data by collision energy (20/40/60 eV and merged spectra). For each feature, fit a kernel density estimate to the proxy-decoy score distribution and use it to convert raw candidate scores into P values. Restrict the candidate list to candidates sharing the molecular formula of the highest-scoring hit before scoring. Use these P values as feature vectors, enforcing feature directionality so each one can only push the SVM score toward the correct-annotation side, then train a per-candidate linear SVM (C chosen from {10^-5,...,10^5} by nested cross-validation) and map decision values to posterior probabilities via Platt scaling. Write calibrated scores (e.g. cosmic_confidence_scores.csv); a threshold of 0.64 roughly corresponds to 10% FDR on independent-data evaluation.

## Related tools

- **COSMIC** (Workflow whose first step calibrates CSI:FingerID hit scores via KDE-estimated P values and directional linear SVMs with Platt scaling) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (Supplies the raw candidate ranking scores that are converted to P values as SVM features) — https://github.com/sirius-ms/sirius
- **SIRIUS 4** (Processes mzML/mzXML LC-MS/MS runs to produce the fragmentation trees and candidate scores upstream of the confidence-score computation) — https://github.com/sirius-ms/sirius

## Evaluation signals

- Every P-value feature vector shows enforced directionality — each feature can only push the SVM decision toward the correct-annotation side
- SVM regularization parameter C is selected from {10^-5, 10^-4, ..., 10^5} by nested cross-validation for each single-candidate SVM
- Candidate lists contain only compounds with the same molecular formula as the top-scoring hit
- Confidence scores at or above 0.64 roughly correspond to ~10% FDR on the independent-data evaluation

## Limitations

- The confidence score is not the probability that an annotation is correct — this is impossible by design of the score
- The 0.64 threshold is practical guidance only, not a guarantee of FDR in either direction, especially on biological datasets with no ground truth
- Small structure modifications are hard or impossible to distinguish from MS/MS data alone, so P-value features cannot separate them

## Evidence

- [abstract] a confidence score consisting of kernel density P value estimation and a support vector machine with enforced directionality of features: "a confidence score consisting of kernel density P value estimation and a support vector machine with enforced directionality of features"
- [discussion] We map decision values to posterior probability estimates using Platt probabilities31.: "We map decision values to posterior probability estimates using Platt probabilities31."
- [discussion] Parameter C ∈ {10−5, 10−4, . . . , 105} of each SVM was chosen by a nested cross-validation.: "Parameter C ∈ {10−5, 10−4, . . . , 105} of each SVM was chosen by a nested cross-validation."
- [discussion] COSMIC’s confidence score must not be mistaken as the probability that an annotation is correct; this is impossible by design of the score.: "COSMIC’s confidence score must not be mistaken as the probability that an annotation is correct; this is impossible by design of the score."
- [readme] SIRIUS integrates a collection of our tools, including CSI:FingerID (with COSMIC): "SIRIUS integrates a collection of our tools, including CSI:FingerID (with COSMIC)"
