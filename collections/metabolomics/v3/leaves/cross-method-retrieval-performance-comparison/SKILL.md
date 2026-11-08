---
name: cross-method-retrieval-performance-comparison
description: Use when a small-molecule NMR structure-search system (e.g. a neural
  fingerprint predictor) must be benchmarked against spectral-library shift-matching
  baselines on the same query spectra and ground-truth structures.
when_to_use_negative:
- Query spectra are not 1H-13C HSQC (the baselines and thresholds 0.05 ppm proton
  / 0.5 ppm carbon are calibrated specifically for HSQC peak shifts)
- Only one method is being evaluated — this skill's value is the like-for-like comparison
  across DeepSAT, SMART 2.0, and shift-search baselines on shared queries
- No ground-truth structures are available for the query spectra, so correct Top-1
  identification and fingerprint-cosine annotation cannot be scored
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: DeepSAT
  role: 'Primary method under benchmark: CNN multi-task model predicting fingerprints,
    molecular weights, and structure classes from HSQC spectra for candidate retrieval'
  repo: https://github.com/mwang87/DeepSAT
- name: SMART 2.0
  role: Baseline retrieval method matching query proton/carbon chemical shifts against
    the in-house HSQC reference library with 0.05 ppm (proton) and 0.5 ppm (carbon)
    thresholds
  repo: https://github.com/mwang87/DeepSAT
- name: NMRShiftDB
  role: Open spectral library searched automatically by raw proton and carbon NMR
    shifts as the weakest baseline (under 3% correct identifications)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-023-00738-4
    title: 'DeepSAT: Learning Molecular Structures from Nuclear Magnetic Resonance
      Data'
  source_content_hash: sha256:8c02c5b8bbbe1c940c7825d649c83b57e946d7092d35b162cd0f6618446e43bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mwang87/DeepSAT
  tools:
  - DeepSAT
  - NMRShiftDB
  - SMART 2.0
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# cross-method-retrieval-performance-comparison

## Summary

Benchmark DeepSAT's fingerprint-based HSQC structure retrieval against SMART 2.0 shift-matching and NMRShiftDB chemical-shift searches on a held-out test set of 1H-13C HSQC spectra, scoring Top-1 identification, annotation rate, and F1@k under a cosine-0.8 Morgan fingerprint threshold.

## When to use

Use when a small-molecule NMR structure-search system (e.g. a neural fingerprint predictor) must be benchmarked against spectral-library shift-matching baselines on the same query spectra and ground-truth structures.

## When NOT to use

- Query spectra are not 1H-13C HSQC (the baselines and thresholds 0.05 ppm proton / 0.5 ppm carbon are calibrated specifically for HSQC peak shifts)
- Only one method is being evaluated — this skill's value is the like-for-like comparison across DeepSAT, SMART 2.0, and shift-search baselines on shared queries
- No ground-truth structures are available for the query spectra, so correct Top-1 identification and fingerprint-cosine annotation cannot be scored

## Inputs

- Held-out test set of 1H-13C HSQC spectra, normal and multiplicity edited (n=3982 query spectra)
- Ground-truth molecular structures paired with each query spectrum
- Known Morgan fingerprints for all ground-truth structures
- In-house reference HSQC spectral library (n=143,467) for SMART 2.0 baseline
- NMRShiftDB reference database (n=53,954 spectra)

## Outputs

- benchmark_comparison_table.csv with Top-1 identification rate, annotation rate, and precision/recall/F1@k per method (DeepSAT, SMART 2.0, NMRShiftDB)
- Per-method Top-1 annotation rates, e.g. 73.5% (DeepSAT, multiplicity-edited) vs 40.3% (SMART 2.0)
- Per-method Top-1 identification rates, e.g. 41.0% (DeepSAT) vs 18.7% (SMART 2.0) vs <3% (NMRShiftDB shift search)

## How to apply

Load a held-out test set of 1H-13C HSQC spectra (normal and multiplicity edited) with ground-truth structures and known Morgan fingerprints, then run DeepSAT inference on each query spectrum to predict fingerprint bit strings, molecular weights, and structure classes, retrieving candidates by cosine similarity. Run SMART 2.0 as a baseline by matching query proton/carbon shifts against the in-house HSQC reference library (n=143,467) with peak-shift thresholds of 0.05 ppm for proton and 0.5 ppm for carbon, and run automated searches of NMRShiftDB (n=53,954) using raw query proton and carbon shifts. Score all three methods identically: a Top-1 identification counts only when the top hit matches ground truth, an annotation counts when the cosine score between predicted and known Morgan fingerprints exceeds 0.8, and compute precision, recall, and F1@k at the same 0.8 threshold. Tabulate all metrics into a benchmark_comparison_table.csv and check reproduction against the reported outcome: DeepSAT Top-1 annotation of 73.5% vs SMART 2.0's 40.3% with multiplicity-edited HSQC, ~2-fold higher Top-1 identification (41.0% vs 18.7%) with normal HSQC, and NMRShiftDB shift searches reaching under 3% correct identifications. Report the expected ~two-fold F1-score advantage of DeepSAT over SMART 2.0.

## Related tools

- **DeepSAT** (Primary method under benchmark: CNN multi-task model predicting fingerprints, molecular weights, and structure classes from HSQC spectra for candidate retrieval) — https://github.com/mwang87/DeepSAT
- **SMART 2.0** (Baseline retrieval method matching query proton/carbon chemical shifts against the in-house HSQC reference library with 0.05 ppm (proton) and 0.5 ppm (carbon) thresholds) — https://github.com/mwang87/DeepSAT
- **NMRShiftDB** (Open spectral library searched automatically by raw proton and carbon NMR shifts as the weakest baseline (under 3% correct identifications))

## Evaluation signals

- Reproduced DeepSAT Top-1 annotation with multiplicity-edited HSQC ≈ 73.5% and SMART 2.0 ≈ 40.3%
- Reproduced Top-1 identification with normal HSQC: DeepSAT 41.0% vs SMART 2.0 18.7% (~2-fold)
- NMRShiftDB shift-search correct identifications fall below 3%
- Similarity threshold applied consistently: annotation counted when predicted-vs-known Morgan fingerprint cosine score exceeds 0.8; precision/recall/F1@k computed at the same 0.8 threshold
- Average cosine score between predicted and ground-truth fingerprints in ~0.85 range (0.8450 normal, 0.8574 multiplicity-edited), confirming the predictor is healthy before retrieval comparison

## Limitations

- Precision/recall/F1 values in the main discussion are deferred to supplementary tables/figures; absolute values are not stated in the discussion text
- NMRShiftDB shift-search baseline uses raw chemical-shift lookups without DeepSAT-style learned representations and performs poorly (under 3%), so it bounds rather than demonstrates the shift-search approach
- The comparison is specific to 1H-13C HSQC inputs and a 143,467-spectrum in-house library; solvents affect retrieval robustness (over 65% correct identification persisted across different solvents, but not all)
- Similarity judgments depend on cosine score over 0.8 of Morgan fingerprints, a similarity — not identity — criterion

## Evidence

- [results] Baseline SMART 2.0 shift-matching thresholds: "The thresholds for the peak shift differences were set as 0.5 ppm for carbon signals and 0.05 ppm for proton signals"
- [results] Similarity threshold definition: "we consider two structures to be “similar” if the cosine score is over 0.8 based on the predicted and known Morgan fingerprints"
- [results] NMRShiftDB baseline result: "The results from searching NMRshiftDB by proton and carbon NMR shifts, however, achieved under 3% of correct identifications."
- [results] In-house library size for baseline retrieval: "the in-house reference HSQC spectral library (n=143,467) was used to retrieve candidate molecular structures"
- [intro] DeepSAT architecture context: "DeepSAT was trained using a convolutional neural network (CNN)-based multi-task supervised learning architecture with 143,467 1H-13C HSQC spectra"
