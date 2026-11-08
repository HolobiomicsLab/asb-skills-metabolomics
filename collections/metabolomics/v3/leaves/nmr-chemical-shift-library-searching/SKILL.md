---
name: nmr-chemical-shift-library-searching
description: Use when a query HSQC spectrum (list of 1H/13C shift pairs) must be identified
  or annotated by peak-shift matching against a reference library, or when reproducing
  the SMART 2.0 / NMRShiftDB baseline of a DeepSAT benchmark.
when_to_use_negative:
- Query spectra are 1D NMR or non-HSQC data — the in-house library and thresholds
  here are specific to 1H-13C HSQC peak shifts
- The goal is annotation of compounds absent from any reference spectral library —
  pure shift searching cannot go beyond library coverage, which is why DeepSAT searches
  structural databases via predicted fingerprints instead
tools:
- name: SMART 2.0
  role: 'Baseline retrieval engine: matches query proton/carbon chemical shifts against
    the in-house HSQC reference library with 0.05 ppm (1H) / 0.5 ppm (13C) thresholds'
  repo: github.com/mwang87/DeepSAT
- name: NMRShiftDB
  role: Open spectral library (n=53,954 reference spectra) searched automatically
    by proton and carbon NMR chemical shifts as the second baseline
- name: DeepSAT
  role: Fingerprint-prediction comparator whose Top-1/annotation rates (73.5% edited
    HSQC; 41.0% normal) benchmark the shift-search results
  repo: github.com/mwang87/DeepSAT
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-023-00738-4
    title: 'DeepSAT: Learning Molecular Structures from Nuclear Magnetic Resonance
      Data'
  source_content_hash: sha256:7641e283eb6b23abca084a27b1368677d63604d4111c418ecc36369a24430362
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepSAT
  - NMRShiftDB
  - SMART 2.0
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# nmr-chemical-shift-library-searching

## Summary

Baseline small-molecule identification by directly matching query 1H-13C HSQC proton/carbon chemical shifts against reference spectral libraries (in-house HSQC library via SMART 2.0, n=143,467; NMRShiftDB, n=53,954), serving as the comparator against DeepSAT fingerprint-based retrieval.

## When to use

Use when a query HSQC spectrum (list of 1H/13C shift pairs) must be identified or annotated by peak-shift matching against a reference library, or when reproducing the SMART 2.0 / NMRShiftDB baseline of a DeepSAT benchmark.

## When NOT to use

- Query spectra are 1D NMR or non-HSQC data — the in-house library and thresholds here are specific to 1H-13C HSQC peak shifts
- The goal is annotation of compounds absent from any reference spectral library — pure shift searching cannot go beyond library coverage, which is why DeepSAT searches structural databases via predicted fingerprints instead

## Inputs

- Query 1H-13C HSQC peak list (proton and carbon chemical shifts in ppm; normal or multiplicity edited)
- In-house reference HSQC spectral library (n=143,467) for SMART 2.0 matching
- NMRShiftDB reference spectra (n=53,954) for automated proton/carbon shift searches
- Ground-truth structures and known Morgan fingerprints for the held-out test spectra

## Outputs

- Ranked list of candidate structures per query spectrum from shift-based library matching
- benchmark_comparison_table.csv with Top-1 identification rates, annotation rates, and precision/recall/F1@k for SMART 2.0 and NMRShiftDB search (relative to DeepSAT)

## How to apply

Collect the query proton and carbon chemical shifts from the 1H-13C HSQC spectrum (normal or multiplicity edited) and match them against reference spectra in the in-house HSQC library (n=143,467) via SMART 2.0 or against NMRShiftDB (n=53,954 reference spectra). Apply peak shift-difference thresholds of 0.5 ppm for carbon signals and 0.05 ppm for proton signals to accept a reference-spectrum match. Rank retrieved candidates and score results using the same conventions as the DeepSAT comparison: a Top-1 identification is correct when the top hit matches ground truth, and structures are 'similar' when the cosine score over Morgan fingerprints exceeds 0.8. Tabulate Top-1 identification rate, annotation rate (cosine > 0.8), and precision/recall/F1@k alongside the DeepSAT results to quantify the benchmark gap.

## Related tools

- **SMART 2.0** (Baseline retrieval engine: matches query proton/carbon chemical shifts against the in-house HSQC reference library with 0.05 ppm (1H) / 0.5 ppm (13C) thresholds) — github.com/mwang87/DeepSAT
- **NMRShiftDB** (Open spectral library (n=53,954 reference spectra) searched automatically by proton and carbon NMR chemical shifts as the second baseline)
- **DeepSAT** (Fingerprint-prediction comparator whose Top-1/annotation rates (73.5% edited HSQC; 41.0% normal) benchmark the shift-search results) — github.com/mwang87/DeepSAT

## Evaluation signals

- Shift-matching thresholds reproduce the paper exactly: 0.05 ppm for proton and 0.5 ppm for carbon peak-shift differences
- NMRShiftDB proton/carbon shift search yields under 3% correct Top-1 identifications on the 3982-spectrum test set
- SMART 2.0 reaches ~18.7% Top-1 identification (normal HSQC) and ~40.3% Top-1 annotation (multiplicity edited), versus 41.0% / 73.5% for DeepSAT
- Similarity scoring uses the shared convention: cosine score over predicted vs known Morgan fingerprints above 0.8
- Recomputed precision/recall/F1@k shows roughly a two-fold higher F1-score for DeepSAT over SMART 2.0

## Limitations

- Shift-search baselines fail on library-uncovered chemistry: NMRShiftDB searching achieved under 3% correct identifications, and open NMR databases cover limited chemical diversity
- Strict shift thresholds (0.05 ppm proton) make matching sensitive to solvent and acquisition-condition differences; DeepSAT solvent-transfer tests still missed some results across solvents
- Absolute precision/recall/F1 values are deferred to supplementary tables/figures rather than the main text, so full reproduction requires the SI

## Evidence

- [results] Baseline thresholds are explicit and method-defining: "The thresholds for the peak shift differences were set as 0.5 ppm for carbon signals and 0.05 ppm for proton signals"
- [results] NMRShiftDB baseline performance ceiling: "The results from searching NMRshiftDB by proton and carbon NMR shifts, however, achieved under 3% of correct identifications."
- [intro] Size of the NMRShiftDB reference resource: "open-sourced spectral libraries have been introduced, such as NMRShiftDB (n = 53,954 reference spectra)"
- [results] Size of the in-house SMART 2.0 reference library: "the in-house reference HSQC spectral library (n=143,467) was used to retrieve candidate molecular structures"
- [results] Shared scoring convention for identification/annotation: "we consider two structures to be “similar” if the cosine score is over 0.8 based on the predicted and known Morgan fingerprints"
- [discussion] Benchmark head-to-head outcome being reproduced: "These metrics revealed the significantly improved performance of DeepSAT over SMART 2.0 with a two-fold higher F1-score"
