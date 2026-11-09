---
name: qtof-msms-cosine-similarity-scoring
description: Use when comparing in-silico structure annotation (COSMIC/CSI:FingerID)
  against spectral library search on independent QTOF MS/MS query spectra, with correct-hit
  counts at FDR 5%.
when_to_use_negative:
- The goal is novel annotation of structures absent from spectral libraries — COSMIC
  is the appropriate primary method, and spectral library search matches only a small
  fraction of spectra
- De-replication where reference spectra for the target compounds are unavailable
  — the article notes COSMIC outperformed spectral library search for de-replication
  despite not being designed for it
- Query compounds whose own reference spectra must remain in the library — this skill
  requires the spectrum-disjoint evaluation setup
edam_operation: http://edamontology.org/operation_0474
edam_topics:
- http://edamontology.org/topic_3175
tools:
- name: SIRIUS 4
  role: Processes the QTOF LC-MS/MS runs, performs de novo molecular formula determination
    (isotope and fragmentation patterns, 10 ppm mass accuracy) and computes fragmentation
    trees feeding the CSI:FingerID comparison
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Ranks structure candidates under the formula restriction of the top-scoring
    hit; its training-structures endpoints supply the reference library for the spectral-library-search
    baseline
  repo: https://github.com/sirius-ms/sirius
- name: COSMIC
  role: In-silico structure annotation method being benchmarked; supplies confidence
    scores (kernel density P value estimation plus SVM with Platt probability mapping)
    used for FDR estimation
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:c0cf9210ab7598dba76b99d758c515b05eec786a8188c6903f187d11797ef9ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - COSMIC
  - CSI:FingerID
  - CSI:FingerID 1.2.0
  - SIRIUS 4
  edam_operation: http://edamontology.org/operation_0474
  edam_topics:
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# qtof-msms-cosine-similarity-scoring

## Summary

Builds spectral-library-search baselines for QTOF tandem MS/MS data using normalized dot-product cosine similarity (plus a square-root-intensity-transformed variant), benchmarked against COSMIC at FDR 5% under a spectrum-disjoint evaluation.

## When to use

Use when comparing in-silico structure annotation (COSMIC/CSI:FingerID) against spectral library search on independent QTOF MS/MS query spectra, with correct-hit counts at FDR 5%.

## When NOT to use

- The goal is novel annotation of structures absent from spectral libraries — COSMIC is the appropriate primary method, and spectral library search matches only a small fraction of spectra
- De-replication where reference spectra for the target compounds are unavailable — the article notes COSMIC outperformed spectral library search for de-replication despite not being designed for it
- Query compounds whose own reference spectra must remain in the library — this skill requires the spectrum-disjoint evaluation setup

## Inputs

- Independent QTOF query tandem MS/MS spectra (medium noise level), LC-MS/MS runs in .mzML or .mzXML format
- CSI:FingerID/COSMIC training structures reference spectral library (positive and negative ion mode)
- Collision-energy conditions: 20 eV spectra and merged fragmentation spectra per feature

## Outputs

- Comparison table (method x condition x correct hits at FDR 5%) written to fdr5_correct_hits_comparison.csv
- Cosine similarity (normalized dot-product) and sqrt-intensity-transformed similarity scores for each query-reference pair
- FDR-estimated top-k hit lists per scoring method

## How to apply

Load independent QTOF query tandem MS/MS spectra (medium noise) and the CSI:FingerID training-structures reference library via the trainingstructures endpoints for positive and negative ion mode. Build two library-search scoring variants: normalized dot-product cosine similarity and its square-root-intensity-transformed variant, restricting reference entries accordingly. Process each query spectrum with SIRIUS 4 for de novo molecular formula determination (10 ppm mass accuracy) and fragmentation trees, then rank structure candidates with CSI:FingerID restricted to candidates sharing the molecular formula of the top-scoring hit. Apply a spectrum-disjoint setup so each query compound's own reference spectrum is excluded from the library during its evaluation. Estimate FDR from the average posterior error probability (Platt-mapped SVM confidence scores) of the top-k hits, then count correct annotations at 5% FDR per method under the 20 eV collision-energy and merged-spectra conditions.

## Related tools

- **SIRIUS 4** (Processes the QTOF LC-MS/MS runs, performs de novo molecular formula determination (isotope and fragmentation patterns, 10 ppm mass accuracy) and computes fragmentation trees feeding the CSI:FingerID comparison) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (Ranks structure candidates under the formula restriction of the top-scoring hit; its training-structures endpoints supply the reference library for the spectral-library-search baseline) — https://github.com/sirius-ms/sirius
- **COSMIC** (In-silico structure annotation method being benchmarked; supplies confidence scores (kernel density P value estimation plus SVM with Platt probability mapping) used for FDR estimation) — https://github.com/sirius-ms/sirius

## Examples

```
import numpy as np; sim = np.dot(q, r) / (np.linalg.norm(q) * np.linalg.norm(r))
```

## Evaluation signals

- Tabulated correct hits at FDR 5% match the reported benchmark: COSMIC 1,415 vs 52 for spectral library search at 20 eV
- With merged spectra, COSMIC 1,701 vs 1 for library search at 5% FDR is reproduced
- Spectrum-disjointness is verified: each query compound's own reference spectrum is excluded from the library during its evaluation
- Cosine similarity scores fall in [0, 1] for the normalized dot-product variant on non-negative intensity vectors
- Formula restriction is enforced: candidates share the molecular formula of the highest-scoring candidate determined at 10 ppm mass accuracy

## Limitations

- COSMIC's confidence score must not be mistaken as the probability that an annotation is correct; this is impossible by design
- Accurate FDR estimation from small molecule fragmentation spectra is expected to remain highly challenging; threshold 0.64 is practical guidance only
- Evaluations rely on reference measurements, as correct answers for biological datasets are unknown
- Only the two-dimensional structure is considered; stereochemistry cannot be established from fragmentation spectra

## Evidence

- [abstract] COSMIC outperforms library search: "COSMIC annotates a substantial number of hits at low false discovery rates and outperforms spectral library search"
- [discussion] Merged spectra better: "Annotation rates as well as separation were consistently better when using merged spectra, usually followed by 40-eV fragmentation spectra."
- [discussion] Formula restriction: "we restrict the candidate list to those candidates with the same molecular formula as the highest-scoring candidate (hit)"
- [discussion] FDR from confidence scores: "we can use the confidence score to estimate the FDR of the top k hits"
- [readme] Training structures endpoints: "https://www.csi-fingerid.uni-jena.de/v3.0/api/fingerid/trainingstructures?predictor=1 (training structures for positive ion mode)"
