---
name: false-discovery-rate-estimation-from-confidence-scores
description: Use when ranked in silico structure candidates (e.g., CSI:FingerID hits
  per compound) must be scored for correctness and an FDR estimated for the top k
  hits, e.g. to pick a confidence-score threshold such as 0.64 for ~10% FDR.
when_to_use_negative:
- The confidence score must not be interpreted as the probability that an annotation
  is correct — this is impossible by design of the score.
- When the goal is to distinguish stereochemistry or small structure modifications
  (e.g., positional isomers) from MS/MS data alone; fragmentation spectra cannot establish
  these.
- When a guaranteed FDR is required for biological datasets — no ground truth exists
  there, so the FDR estimated from reference measurements cannot be transferred with
  certainty.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: COSMIC
  role: Computes the confidence score (kernel density P value estimation + SVM) used
    for FDR estimation over structure candidates
  repo: https://github.com/sirius-ms/sirius
- name: SIRIUS 4
  role: Preprocesses LC-MS/MS runs (feature detection, spectral merging, de novo molecular
    formula at 10 ppm) to feed candidate ranking and scoring
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Produces the ranked structure candidates whose top k hits are assigned confidence
    scores and FDR estimates
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:951ff1aee630da080b16e460b32e2369f28c2e2b32714166b109a25dcc729392
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - CFM-ID
  - COSMIC
  - CSI:FingerID
  - CSI:FingerID IOKR
  - MAGMa+
  - MetFrag
  - SIRIUS 4
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# false-discovery-rate-estimation-from-confidence-scores

## Summary

Estimates the false discovery rate of in silico structural annotations (e.g., CSI:FingerID hits) from the COSMIC confidence score, which combines kernel density P value estimation with an SVM to separate correct from incorrect hits. Enables selecting a score threshold (e.g., 0.64 ≈ 10% FDR) and counting reliable annotations.

## When to use

Use when ranked in silico structure candidates (e.g., CSI:FingerID hits per compound) must be scored for correctness and an FDR estimated for the top k hits, e.g. to pick a confidence-score threshold such as 0.64 for ~10% FDR.

## When NOT to use

- The confidence score must not be interpreted as the probability that an annotation is correct — this is impossible by design of the score.
- When the goal is to distinguish stereochemistry or small structure modifications (e.g., positional isomers) from MS/MS data alone; fragmentation spectra cannot establish these.
- When a guaranteed FDR is required for biological datasets — no ground truth exists there, so the FDR estimated from reference measurements cannot be transferred with certainty.

## Inputs

- LC-MS/MS runs in .mzML or .mzXML format (centroided, positive ion mode) processed by SIRIUS 4
- Per-compound ranked structure candidate lists with hit scores (e.g., CSI:FingerID, MetFrag, MAGMa+, CFM-ID, CSI:FingerID IOKR)
- Candidate lists restricted to the molecular formula of the highest-scoring hit
- Known correct structures for benchmark compounds (e.g., CASMI 2016 queries)

## Outputs

- Confidence scores (Platt-calibrated posterior probabilities) per structure candidate
- Posterior error probabilities and estimated FDR for the top k hits
- Counts of correct hits at chosen FDR levels (e.g., 10% and 0%)
- ROC curves, hop plots, and mean ranks of correct hits (e.g., casmi2016_roc_hop_results.csv with AUC values)

## How to apply

Rank structure candidates per compound with CSI:FingerID (SIRIUS 4 processing of mzML/mzXML input, 10 ppm mass accuracy for de novo molecular formula), then restrict the candidate list to candidates sharing the molecular formula of the top-scoring hit. Compute the COSMIC confidence score for each candidate using kernel density P value estimation of features fed into a support vector machine with enforced directionality of features; map SVM decision values to posterior probabilities via Platt probabilities, choosing the SVM parameter C from {10^-5,...,10^5} by nested cross-validation. Estimate the FDR of the top k hits as the average posterior error probability derived from the confidence scores; a threshold of 0.64 roughly corresponded to FDR 10% in independent-data evaluation. Validate separation quality by ranking candidates against known correct structures and computing ROC curves, hop plots, and mean ranks of correct hits (expected contest-method AUC 0.40–0.55 versus COSMIC ~0.82).

## Related tools

- **COSMIC** (Computes the confidence score (kernel density P value estimation + SVM) used for FDR estimation over structure candidates) — https://github.com/sirius-ms/sirius
- **SIRIUS 4** (Preprocesses LC-MS/MS runs (feature detection, spectral merging, de novo molecular formula at 10 ppm) to feed candidate ranking and scoring) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (Produces the ranked structure candidates whose top k hits are assigned confidence scores and FDR estimates) — https://github.com/sirius-ms/sirius

## Evaluation signals

- On CASMI 2016, the confidence score achieves AUC ~0.82 on ROC curves, exceeding contest-method baselines (0.40–0.55 for MetFrag, MAGMa+, CFM-ID, CSI:FingerID original and IOKR).
- At FDR below 10% against the biomolecule structure database, 57 correct hits are counted over 123 queries; against ChemSpider, 16 hits at 0% FDR over 127 queries.
- Mean rank of correct hits improves from 56.5 to 33.9 when using the confidence score.
- A threshold of 0.64 on the confidence score corresponds roughly to 10% FDR in independent-data evaluation.
- Output CSV (e.g., casmi2016_roc_hop_results.csv) must contain AUC, correct-hit counts at both FDR levels, and mean ranks for every compared method.

## Limitations

- The confidence score must not be mistaken for the probability that an annotation is correct; this is impossible by design of the score.
- Accurate FDR estimation from small-molecule fragmentation spectra is expected to remain highly challenging; the 0.64 threshold is practical guidance only and no guarantee in either direction.
- A similar FDR cannot be guaranteed for biological annotations, since no ground truth exists for biological datasets and evaluations rely on reference measurements.
- Small structure modifications can be hard or impossible to tell apart using MS/MS data alone, which limits separation of correct from incorrect hits.

## Evidence

- [abstract] Confidence-score SVM calibration: "a confidence score consisting of kernel density P value estimation and a support vector machine with enforced directionality of features"
- [discussion] FDR from top k hits: "we can use the confidence score to estimate the FDR of the top k hits"
- [discussion] Threshold-FDR correspondence: "We have used a threshold of 0.64 for the confidence score, roughly corresponding to FDR 10%"
- [discussion] Platt probability calibration: "We map decision values to posterior probability estimates using Platt probabilities31."
- [readme] SIRIUS CLI/GUI retrieval of results: "Results are retrieved from the web service and can be displayed in the SIRIUS graphical user interface. This functionality is also available for the SIRIUS command-line tool."
