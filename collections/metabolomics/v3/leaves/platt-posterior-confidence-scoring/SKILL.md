---
name: platt-posterior-confidence-scoring
description: Use when ranking CSI:FingerID structure candidates and needing to distinguish
  correct from incorrect in silico annotations, or when estimating FDR for top-k metabolite
  structure annotations lacking spectral library matches.
when_to_use_negative:
- When the confidence score is needed as the probability that an annotation is correct
  — the authors state this is 'impossible by design of the score'.
- When stereochemistry must be resolved — fragmentation spectra cannot establish stereochemistry,
  so only two-dimensional structures are considered.
- When accurate FDR guarantees are required for biological annotations — FDR cannot
  be accurately estimated and the 0.64 threshold 'is clearly no guarantee in either
  direction'.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: COSMIC
  role: Workflow that assigns confidence scores (kernel density P value estimation
    + SVM with enforced feature directionality, Platt probabilities) to CSI:FingerID
    structure annotations
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Ranks structure candidates and supplies decision values that are mapped to
    Platt posterior probabilities for confidence scoring
  repo: https://github.com/sirius-ms/sirius
- name: SIRIUS 4
  role: Determines molecular formulas de novo (10 ppm) and preprocesses LC–MS/MS runs
    feeding the confidence-score pipeline
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:18a5f34b661fcef9a61b8f00101213f3196c0cda666d220bc056a2362591b729
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - COSMIC
  - CSI:FingerID
  - SIRIUS 4
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# platt-posterior-confidence-scoring

## Summary

Compute COSMIC confidence scores by mapping SVM decision values to Platt posterior probabilities and combining kernel density P value estimation with a directionality-enforced SVM, so that in silico structure annotations from CSI:FingerID can be filtered at a calibrated threshold (~0.64, roughly FDR 10%).

## When to use

Use when ranking CSI:FingerID structure candidates and needing to distinguish correct from incorrect in silico annotations, or when estimating FDR for top-k metabolite structure annotations lacking spectral library matches.

## When NOT to use

- When the confidence score is needed as the probability that an annotation is correct — the authors state this is 'impossible by design of the score'.
- When stereochemistry must be resolved — fragmentation spectra cannot establish stereochemistry, so only two-dimensional structures are considered.
- When accurate FDR guarantees are required for biological annotations — FDR cannot be accurately estimated and the 0.64 threshold 'is clearly no guarantee in either direction'.

## Inputs

- SIRIUS 4 output: fragmentation trees and CSI:FingerID structure candidate rankings with SVM decision values
- LC–MS/MS runs in .mzML or .mzXML format (centroided, positive ion mode)
- Structure database candidate lists (e.g. HMDB structure database with 113,983 compounds, biomolecule structure database)
- Reference training data with correct/incorrect annotation labels (e.g. combined MassBank, GNPS, NIST 2017 database)

## Outputs

- Per-annotation COSMIC confidence scores (posterior-probability-based, NOT the probability the annotation is correct)
- High-confidence annotation set at threshold 0.64 (e.g. 911 of 24,554 annotations)
- FDR estimates for top-k hits (average posterior error probability)

## How to apply

After SIRIUS 4 determines molecular formulas de novo (10 ppm mass accuracy) and CSI:FingerID ranks structure candidates (restricted to candidates sharing the top hit's molecular formula, compounds >700 Da discarded), train a support vector machine with enforced directionality of features; the SVM C parameter is selected from {10^-5, ..., 10^5} via nested cross-validation. Map SVM decision values to posterior probability estimates using Platt probabilities, and combine these with kernel density P value estimation to form the COSMIC confidence score. Filter annotations at the confidence score threshold 0.64, which roughly corresponds to FDR 10% in independent-data evaluation, and estimate the FDR of the top k hits as the average posterior error probability of those hits. In the flagship application this pipeline filtered 24,554 structure annotations to 911 high-confidence ones, and applying the threshold to 17,400 metabolomics experiments yielded 1,715 annotations absent from spectral libraries.

## Related tools

- **COSMIC** (Workflow that assigns confidence scores (kernel density P value estimation + SVM with enforced feature directionality, Platt probabilities) to CSI:FingerID structure annotations) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (Ranks structure candidates and supplies decision values that are mapped to Platt posterior probabilities for confidence scoring) — https://github.com/sirius-ms/sirius
- **SIRIUS 4** (Determines molecular formulas de novo (10 ppm) and preprocesses LC–MS/MS runs feeding the confidence-score pipeline) — https://github.com/sirius-ms/sirius

## Examples

```
sirius --input runs.mzML --formula --csi-fingerid --cosmic --confidence-threshold 0.64
```

## Evaluation signals

- Confidence score threshold 0.64 applied; retained annotations all carry score >= 0.64 (911 of 24,554 annotations in the reference run).
- Independent-data evaluation: threshold 0.64 roughly corresponded to FDR 10%.
- Estimated FDR from average posterior error probability of top-k hits matches reported ranges.
- SVM C parameter chosen from {10^-5,...,10^5} by nested cross-validation is documented in the run configuration.
- Repurposing 20,080 LC–MS/MS runs yielded ~49% of hits with no reference spectra in training data — a sanity check that scores are not trivially memorization-driven.

## Limitations

- The confidence score must not be mistaken as the probability the annotation is correct — impossible by design of the score.
- Accurate FDR estimation from small-molecule fragmentation spectra is expected to remain highly challenging; a similar FDR cannot be guaranteed for biological annotations.
- Small structure modifications are hard or impossible to tell apart using MS/MS data alone; orthogonal information is required.
- Evaluations rely on reference measurements because the correct answers for biological datasets are unknown.

## Evidence

- [abstract] confidence score consisting of kernel density P value estimation and a support vector machine with enforced directionality of features: "confidence score consisting of kernel density P value estimation and a support vector machine with enforced directionality of features"
- [discussion] We map decision values to posterior probability estimates using Platt probabilities31.: "We map decision values to posterior probability estimates using Platt probabilities31."
- [discussion] Parameter C ∈ {10−5, 10−4, . . . , 105} of each SVM was chosen by a nested cross-validation.: "Parameter C ∈ {10−5, 10−4, . . . , 105} of each SVM was chosen by a nested cross-validation."
- [discussion] We now filter the 24,554 structure annotations for high confidence (score threshold 0.64), resulting in 911 structure annotations.: "We now filter the 24,554 structure annotations for high confidence (score threshold 0.64), resulting in 911 structure annotations."
- [discussion] COSMIC’s confidence score must not be mistaken as the probability that an annotation is correct; this is impossible by design of the score.: "COSMIC’s confidence score must not be mistaken as the probability that an annotation is correct; this is impossible by design of the score."
