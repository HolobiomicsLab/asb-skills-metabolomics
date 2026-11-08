---
name: decoy-based-fdr-estimation
description: Use when annotating small-molecule LC–MS/MS spectra against a structure
  database (e.g. PubChem/HMDB) and you must separate correct from incorrect CSI:FingerID
  hits and estimate FDR, or when calibrating hit scores into posterior-like confidences.
when_to_use_negative:
- You need the probability that an annotation is correct — the paper states the confidence
  score must not be mistaken for this; it is impossible by design.
- You need to distinguish small structure modifications or stereochemistry from MS/MS
  data alone — the paper says these are hard or impossible to tell apart.
- Input is already a spectral-library search result (library spectra matched); the
  skill targets in silico annotations where no reference spectra exist.
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: COSMIC
  role: Computes the confidence score via kernel density P value estimation and directional
    SVM, enabling FDR estimation of structure annotations
  repo: https://bio.informatik.uni-jena.de/software/cosmic/
- name: CSI:FingerID
  role: Ranks structure candidates and provides the raw hit scores that are kernel-density-calibrated
    into P values as SVM features
  repo: https://github.com/sirius-ms/sirius
- name: SIRIUS 4
  role: Processes LC–MS/MS runs (mzML/mzXML), merges spectra per feature by agglomerative
    clustering on cosine distance, and determines molecular formulas de novo
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:ed11f7e6a9c2bb138c37b9d60b2adae89abd277466bc6216ee08b2ccd1e92d75
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COSMIC
  - CSI:FingerID
  - SIRIUS 4
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# decoy-based-fdr-estimation

## Summary

COSMIC's method for attaching calibrated confidence scores to in silico structure annotations, combining kernel density P/E-value calibration of CSI:FingerID scores, directional linear SVMs, and Platt scaling so that a score threshold can be used to estimate FDR of top-k hits.

## When to use

Use when annotating small-molecule LC–MS/MS spectra against a structure database (e.g. PubChem/HMDB) and you must separate correct from incorrect CSI:FingerID hits and estimate FDR, or when calibrating hit scores into posterior-like confidences.

## When NOT to use

- You need the probability that an annotation is correct — the paper states the confidence score must not be mistaken for this; it is impossible by design.
- You need to distinguish small structure modifications or stereochemistry from MS/MS data alone — the paper says these are hard or impossible to tell apart.
- Input is already a spectral-library search result (library spectra matched); the skill targets in silico annotations where no reference spectra exist.

## Inputs

- SIRIUS 4 output for LC–MS/MS runs (.mzML/.mzXML) with fragmentation trees and molecular formulas (10 ppm mass accuracy)
- CSI:FingerID-ranked structure candidate lists with raw scores, restricted to the top hit's molecular formula
- Combined reference-spectra training set (MassBank, GNPS, NIST 2017) with hit and proxy-decoy candidate scores, split by collision energy (20/40/60 eV, merged)

## Outputs

- Calibrated confidence score (posterior probability) per structure annotation
- cosmic_confidence_scores.csv with per-candidate confidence scores
- FDR estimate for the top k hits above a chosen confidence threshold (e.g. score >= 0.64 ~ FDR 10%)

## How to apply

First split reference-spectrum training data (MassBank, GNPS, NIST 2017) by collision energy (20/40/60 eV plus merged spectra) with candidate scores for hits and proxy decoys. For each feature, fit a kernel density estimate to the proxy-decoy score distribution (modelled as a mixture of log-normals) and convert raw candidate scores into P values, then assemble feature vectors with every feature directionally constrained to push the SVM score toward the correct-annotation side. Train one linear SVM per candidate per collision energy, choosing C from {10^-5,...,10^5} by nested cross-validation, and map decision values to posterior probability estimates via Platt scaling. Restrict the candidate list to candidates with the same molecular formula as the top-scoring hit before scoring, and write calibrated scores (e.g. to cosmic_confidence_scores.csv). A threshold of 0.64 roughly corresponded to 10% FDR in the independent-data evaluation and can be used to estimate the FDR of the top k hits, but this number is practical guidance, not a guarantee.

## Related tools

- **COSMIC** (Computes the confidence score via kernel density P value estimation and directional SVM, enabling FDR estimation of structure annotations) — https://bio.informatik.uni-jena.de/software/cosmic/
- **CSI:FingerID** (Ranks structure candidates and provides the raw hit scores that are kernel-density-calibrated into P values as SVM features) — https://github.com/sirius-ms/sirius
- **SIRIUS 4** (Processes LC–MS/MS runs (mzML/mzXML), merges spectra per feature by agglomerative clustering on cosine distance, and determines molecular formulas de novo) — https://github.com/sirius-ms/sirius

## Evaluation signals

- Confidence scores at or above the 0.64 threshold should roughly correspond to 10% FDR on the independent-data evaluation.
- SVM regularization parameter C must be selected from {10^-5, 10^-4, ..., 10^5} by nested cross-validation; deviations indicate a misconfigured run.
- Candidate list passed to scoring must be restricted to candidates sharing the top hit's molecular formula.
- Separation of correct vs. incorrect annotations should be best for merged spectra (or 40 eV), reflecting expected per-collision-energy behavior.
- Output file (e.g. cosmic_confidence_scores.csv) should contain one calibrated posterior-probability score per reference-spectrum annotation.

## Limitations

- The confidence score must not be interpreted as the probability that an annotation is correct; this is impossible by design of the score.
- The 0.64 threshold is practical guidance only and is no guarantee of FDR in either direction; accurate FDR estimation from small-molecule fragmentation spectra is expected to remain highly challenging.
- No similar FDR can be guaranteed for biological annotations, since no ground truth exists for biological datasets and evaluations rely on reference measurements.

## Evidence

- [abstract] a confidence score consisting of kernel density P value estimation and a support vector machine with enforced directionality of features: "a confidence score consisting of kernel density P value estimation and a support vector machine with enforced directionality of features"
- [discussion] We have used a threshold of 0.64 for the confidence score, roughly corresponding to FDR 10%: "We have used a threshold of 0.64 for the confidence score, roughly corresponding to FDR 10%"
- [methods] We map decision values to posterior probability estimates using Platt probabilities31.: "We map decision values to posterior probability estimates using Platt probabilities31."
- [readme] SIRIUS integrates a collection of our tools, including CSI:FingerID (with COSMIC): "SIRIUS integrates a collection of our tools, including CSI:FingerID (with COSMIC)"
