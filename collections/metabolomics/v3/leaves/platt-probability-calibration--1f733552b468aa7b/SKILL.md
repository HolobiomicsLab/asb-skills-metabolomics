---
name: platt-probability-calibration
description: Use when calibrating per-candidate SVM decision values from tandem MS/MS
  structure annotation (e.g. CSI:FingerID hits) into posterior confidence scores for
  FDR estimation and hit filtering.
when_to_use_negative:
- The SVM decision values are already calibrated probabilities or a downstream tool
  expects raw decision values rather than posteriors
- The task is spectral library search de-replication where binary hit/miss classification,
  not confidence calibration of in silico hits, is required
- Stereochemistry or exact 3D structure validation is the goal — the score calibrates
  two-dimensional structure annotation only
edam_operation: http://edamontology.org/operation_0321
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0154
tools:
- name: COSMIC
  role: Computes the kernel density P values, trains the directional single-candidate
    SVMs, and applies Platt scaling to produce the confidence score
- name: CSI:FingerID
  role: Supplies the candidate hit scores and rankings whose per-hit score distribution
    is calibrated into the confidence score
- name: SIRIUS 4
  role: Runs the upstream de novo molecular formula determination and CSI:FingerID
    candidate ranking whose per-candidate SVM decision values are Platt-scaled
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:905f9f6b59b40aff84e5788e036944743df620ca940237dde919a7dc68ae47c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - COSMIC
  - CSI:FingerID
  - SIRIUS 4
  edam_operation: http://edamontology.org/operation_0321
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# platt-probability-calibration

## Summary

Calibrates COSMIC's single-candidate linear SVM decision values into posterior probability estimates using Platt scaling, turning the SVM score into the COSMIC confidence score that separates correct from incorrect structural annotations of in silico hit candidates.

## When to use

Use when calibrating per-candidate SVM decision values from tandem MS/MS structure annotation (e.g. CSI:FingerID hits) into posterior confidence scores for FDR estimation and hit filtering.

## When NOT to use

- The SVM decision values are already calibrated probabilities or a downstream tool expects raw decision values rather than posteriors
- The task is spectral library search de-replication where binary hit/miss classification, not confidence calibration of in silico hits, is required
- Stereochemistry or exact 3D structure validation is the goal — the score calibrates two-dimensional structure annotation only

## Inputs

- Feature vectors of kernel density P values computed from proxy-decoy (PubChem candidate) score distributions per feature and collision energy
- SVM decision values from single-candidate linear SVMs trained on the combined MassBank/GNPS/NIST 2017 reference-spectra training set

## Outputs

- Posterior probability estimates per hit (COSMIC confidence scores)
- Average posterior error probability of the top k hits for FDR estimation
- Calibrated confidence score per structure annotation for threshold filtering

## How to apply

Train one linear SVM per candidate per collision energy (20/40/60 eV, plus merged spectra), assembling feature vectors from kernel-density-estimated P values of candidate scores with every feature's directionality enforced so each can only push the SVM score toward the correct-annotation side. Select the regularization parameter C ∈ {10^-5, 10^-4, …, 10^5} by nested cross-validation. Map the resulting SVM decision values to posterior probability estimates using Platt probabilities; the posterior probability of the top-scoring hit becomes its confidence score, which is used to estimate FDR of the top k hits. Practical guidance: a confidence score threshold of 0.64 roughly corresponded to FDR 10% on the independent-data evaluation.

## Related tools

- **COSMIC** (Computes the kernel density P values, trains the directional single-candidate SVMs, and applies Platt scaling to produce the confidence score)
- **CSI:FingerID** (Supplies the candidate hit scores and rankings whose per-hit score distribution is calibrated into the confidence score)
- **SIRIUS 4** (Runs the upstream de novo molecular formula determination and CSI:FingerID candidate ranking whose per-candidate SVM decision values are Platt-scaled) — https://github.com/sirius-ms/sirius

## Examples

```
sirius --input spectra.mgf --formula --structure --confidence
```

## Evaluation signals

- Confidence score threshold 0.64 should roughly correspond to FDR 10% in independent-data evaluation — check separation of correct vs incorrect hits at that cutoff
- Directionality check: every SVM feature must only push the decision value toward the correct-annotation side (enforced directionality)
- Posterior probability values fall in [0,1] and Platt-scaled scores show clear separation between hits and proxy decoys
- Merged spectra and 40-eV spectra typically yield better separation than other collision energies — if separation is worse at these settings, revisit P-value features or SVM training
- All three collision energies (20/40/60 eV) must be available for a compound before computing the confidence score

## Limitations

- The confidence score must not be mistaken as the probability that an annotation is correct; this is impossible by design of the score.
- Accurate FDR estimation from small-molecule fragmentation spectra is expected to remain highly challenging; the 0.64 threshold is practical guidance, no guarantee in either direction.
- Requires proxy decoy score distributions from a large candidate database (PubChem candidates) for kernel density estimation — not applicable to small curated databases.
- Applicable to two-dimensional structure annotation only; cannot establish stereochemistry from fragmentation spectra.

## Evidence

- [methods] We map decision values to posterior probability estimates using Platt probabilities: "We map decision values to posterior probability estimates using Platt probabilities"
- [abstract] a confidence score consisting of kernel density P value estimation and a support vector machine with enforced directionality of features: "a confidence score consisting of kernel density P value estimation and a support vector machine with enforced directionality of features"
- [methods] Parameter C ∈ {10−5, 10−4, . . . , 105} of each SVM was chosen by a nested cross-validation.: "Parameter C ∈ {10−5, 10−4, . . . , 105} of each SVM was chosen by a nested cross-validation."
- [discussion] We have used a threshold of 0.64 for the confidence score, roughly corresponding to FDR 10%: "We have used a threshold of 0.64 for the confidence score, roughly corresponding to FDR 10%"
- [discussion] COSMIC’s confidence score must not be mistaken as the probability that an annotation is correct; this is impossible by design of the score.: "COSMIC’s confidence score must not be mistaken as the probability that an annotation is correct; this is impossible by design of the score."
