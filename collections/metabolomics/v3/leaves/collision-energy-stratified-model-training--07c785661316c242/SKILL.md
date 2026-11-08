---
name: collision-energy-stratified-model-training
description: Use when calibrating structure-annotation confidence from CSI:FingerID
  candidate scores and reference spectra (MassBank, GNPS, NIST 2017) split by collision
  energy.
when_to_use_negative:
- You need the probability that an annotation is correct — the article states the
  confidence score 'must not be mistaken as the probability that an annotation is
  correct; this is impossible by design of the score.'
- The input is raw LC-MS/MS data rather than CSI:FingerID candidate scores; SIRIUS
  4 processing and structure candidate ranking must happen first.
- Collision energies other than the 20/40/60 eV (or merged) spectra the models were
  trained on, without retraining.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: COSMIC
  role: Workflow that computes confidence scores from kernel-density P values and
    directional SVMs over CSI:FingerID scores
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Produces ranked structure candidate scores that are calibrated into P values
    and SVM features
  repo: https://github.com/sirius-ms/sirius
- name: SIRIUS 4
  role: Processes LC-MS/MS runs (mzML/mzXML) and generates fragmentation trees and
    candidate scores upstream
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:3274f21ee0de14ced8256835c89a3cf23d82e71deb88cdbdaa16db41d761f65e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - COSMIC
  - CSI:FingerID
  - SIRIUS 4
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# collision-energy-stratified-model-training

## Summary

Trains COSMIC's confidence-score machinery separately per collision energy (20/40/60 eV and merged spectra): kernel-density P/E-value calibration of CSI:FingerID scores, directional linear SVMs, and Platt scaling to posterior probabilities.

## When to use

Use when calibrating structure-annotation confidence from CSI:FingerID candidate scores and reference spectra (MassBank, GNPS, NIST 2017) split by collision energy.

## When NOT to use

- You need the probability that an annotation is correct — the article states the confidence score 'must not be mistaken as the probability that an annotation is correct; this is impossible by design of the score.'
- The input is raw LC-MS/MS data rather than CSI:FingerID candidate scores; SIRIUS 4 processing and structure candidate ranking must happen first.
- Collision energies other than the 20/40/60 eV (or merged) spectra the models were trained on, without retraining.

## Inputs

- Combined reference-spectra training set (MassBank, GNPS, NIST 2017) with candidate scores for hits and proxy decoys
- CSI:FingerID candidate scores per structure candidate, split by collision energy (20, 40, 60 eV, merged)
- Feature vectors of kernel-density-estimated P values per candidate

## Outputs

- Per-candidate calibrated confidence scores (posterior probabilities via Platt scaling)
- Trained linear SVMs (one per candidate per collision energy) with nested-CV-selected C
- cosmic_confidence_scores.csv with calibrated confidence scores

## How to apply

Load the combined reference-spectra training set (MassBank, GNPS, NIST 2017) with hit and proxy-decoy candidate scores and split by collision energy (20/40/60 eV plus merged spectra); the article reports merged spectra and 40 eV give the best annotation rates and separation. For each feature, fit a kernel density estimate to the proxy-decoy score distribution to convert raw candidate scores into P values, modeling hit-score distributions as a mixture of log-normal distributions. Assemble feature vectors from these P values, enforcing directionality so each feature can only push the SVM score toward the correct-annotation side. Train one linear SVM per candidate per collision energy, selecting C from {10^-5,...,10^5} by nested cross-validation, then map SVM decision values to posterior probability estimates using Platt probabilities. Validate by checking that a confidence score threshold of 0.64 roughly corresponds to 10% FDR on independent-data evaluation.

## Related tools

- **COSMIC** (Workflow that computes confidence scores from kernel-density P values and directional SVMs over CSI:FingerID scores) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (Produces ranked structure candidate scores that are calibrated into P values and SVM features) — https://github.com/sirius-ms/sirius
- **SIRIUS 4** (Processes LC-MS/MS runs (mzML/mzXML) and generates fragmentation trees and candidate scores upstream) — https://github.com/sirius-ms/sirius

## Examples

```
sirius --input spectra.mzML --formula --structure --confidence  # then score all reference-spectrum annotations and write cosmic_confidence_scores.csv
```

## Evaluation signals

- Confidence scores at or above the 0.64 threshold should roughly correspond to 10% FDR on the independent-data evaluation
- Annotation rates and hit/decoy separation should be best for merged spectra, usually followed by 40 eV spectra
- SVM regularization parameter C falls within the grid {10^-5, 10^-4, ..., 10^5} chosen by nested cross-validation
- Every feature directionality constraint holds: each feature only pushes the SVM score toward the correct-annotation side
- Calibrated outputs are posterior probability estimates in [0,1] written to cosmic_confidence_scores.csv

## Limitations

- The confidence score must not be mistaken as the probability that an annotation is correct; this is impossible by design of the score.
- The 0.64 threshold is practical guidance only and is no guarantee of FDR in either direction.
- Accurate FDR estimation from small molecule fragmentation spectra is expected to remain highly challenging, so a similar FDR cannot be guaranteed for biological annotations.
- Small structure modifications are hard or impossible to tell apart using MS/MS data alone; stereochemistry cannot be established.

## Evidence

- [abstract] kernel density P value estimation and a support vector machine with enforced directionality of features: "a confidence score consisting of kernel density P value estimation and a support vector machine with enforced directionality of features"
- [methods] nested cross-validation for C: "Parameter C ∈ {10−5, 10−4, . . . , 105} of each SVM was chosen by a nested cross-validation."
- [methods] Platt probabilities mapping: "We map decision values to posterior probability estimates using Platt probabilities31."
- [results] merged spectra best performance: "Annotation rates as well as separation were consistently better when using merged spectra, usually followed by 40-eV fragmentation spectra."
- [discussion] threshold 0.64 corresponds to 10% FDR: "We have used a threshold of 0.64 for the confidence score, roughly corresponding to FDR 10%"
- [discussion] score not a probability: "COSMIC’s confidence score must not be mistaken as the probability that an annotation is correct; this is impossible by design of the score."
