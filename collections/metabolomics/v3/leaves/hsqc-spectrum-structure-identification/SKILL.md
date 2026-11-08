---
name: hsqc-spectrum-structure-identification
description: Use when the input consists of query 1H-13C HSQC or multiplicity-edited
  HSQC peak lists/spectra and the goal is top-k small-molecule identification or annotation
  (fingerprint cosine similarity) against large structure databases such as the Moliverse
  library (n=143,467).
when_to_use_negative:
- Query spectra are 1D proton-only or carbon-only spectra, or other MS/MS data — DeepSAT
  is trained specifically on 1H-13C HSQC spectra
- Authentic reference HSQC spectra of the target compound exist in a spectral library
  — direct library matching is then more appropriate
- The task requires fully elucidated novel structures, not annotation of the nearest
  known analogue — DeepSAT identifies related known compounds and 'does not fully
  explain the decision-making process'
tools:
- name: DeepSAT
  role: CNN-based multi-task inference engine that predicts fingerprint bit strings,
    molecular weights, and structure classes from 1H-13C HSQC spectra
  repo: github.com/mwang87/DeepSAT
- name: SMART 2.0
  role: Baseline structure retrieval by matching query proton/carbon chemical shifts
    against the in-house HSQC reference library with 0.05 ppm (proton) and 0.5 ppm
    (carbon) thresholds
- name: NMRShiftDB
  role: Open reference spectral database searched automatically by query proton and
    carbon NMR shifts as a second baseline (under 3% correct identifications)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-023-00738-4
    title: 'DeepSAT: Learning Molecular Structures from Nuclear Magnetic Resonance
      Data'
  source_content_hash: sha256:b03016c9b6b482df0bfb0040d3b171e11a47df56581648a82b0437512e4fdf9c
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
yes_rate: 0.6666666666666666
n_claims: 9
---

# hsqc-spectrum-structure-identification

## Summary

Structure identification and annotation of small molecules from 1H-13C HSQC (or multiplicity-edited HSQC) spectra using DeepSAT, a CNN-based multi-task model that predicts Morgan fingerprint bit strings, molecular weights, and structure classes, benchmarked against SMART 2.0 shift matching and NMRShiftDB chemical-shift searches.

## When to use

Use when the input consists of query 1H-13C HSQC or multiplicity-edited HSQC peak lists/spectra and the goal is top-k small-molecule identification or annotation (fingerprint cosine similarity) against large structure databases such as the Moliverse library (n=143,467).

## When NOT to use

- Query spectra are 1D proton-only or carbon-only spectra, or other MS/MS data — DeepSAT is trained specifically on 1H-13C HSQC spectra
- Authentic reference HSQC spectra of the target compound exist in a spectral library — direct library matching is then more appropriate
- The task requires fully elucidated novel structures, not annotation of the nearest known analogue — DeepSAT identifies related known compounds and 'does not fully explain the decision-making process'

## Inputs

- Query 1H-13C HSQC peak lists/spectra (normal and multiplicity-edited)
- Ground-truth molecular structures with known Morgan fingerprints for the held-out test spectra
- In-house reference HSQC spectral library (Moliverse, n=143,467) for SMART 2.0 baseline retrieval
- NMRShiftDB reference spectra (n=53,954) for automated shift search baseline

## Outputs

- Predicted fingerprint bit strings, molecular weights, and structure classes per query spectrum
- Ranked candidate structures by fingerprint cosine similarity
- benchmark_comparison_table.csv with Top-1 identification/annotation rates and precision/recall/F1@k for DeepSAT, SMART 2.0, and NMRShiftDB search

## How to apply

Load the query HSQC (normal or multiplicity-edited) peak lists and run DeepSAT inference to predict fingerprint bit strings, molecular weights, and structure classes. Retrieve candidates by cosine similarity between the predicted fingerprints and known Morgan fingerprints generated from small-molecule databases, treating structures as 'similar' when the cosine score exceeds 0.8; annotation rate also uses this 0.8 cosine threshold. For benchmarking, run SMART 2.0 by matching query proton/carbon shifts against the in-house reference HSQC spectral library (n=143,467) with peak shift difference thresholds of 0.05 ppm for proton and 0.5 ppm for carbon signals, and search NMRShiftDB (n=53,954 spectra) by query proton and carbon NMR shifts. Score Top-1 identification (top hit matches ground truth), annotation rate (cosine > 0.8), and precision/recall/F1@k for all three methods into a benchmark comparison table (e.g. benchmark_comparison_table.csv). Validate against the reported outcomes: DeepSAT Top-1 annotation of 73.5% (edited HSQC) versus 40.3% for SMART 2.0, over 2-fold higher Top-1 identification than SMART 2.0 (41.0% vs 18.7%, normal HSQC), and under 3% correct identifications for NMRShiftDB shift search.

## Related tools

- **DeepSAT** (CNN-based multi-task inference engine that predicts fingerprint bit strings, molecular weights, and structure classes from 1H-13C HSQC spectra) — github.com/mwang87/DeepSAT
- **SMART 2.0** (Baseline structure retrieval by matching query proton/carbon chemical shifts against the in-house HSQC reference library with 0.05 ppm (proton) and 0.5 ppm (carbon) thresholds)
- **NMRShiftDB** (Open reference spectral database searched automatically by query proton and carbon NMR shifts as a second baseline (under 3% correct identifications))

## Examples

```
curl -X POST https://deepsat.ucsd.edu/api/smart3/search -d '[{"1H": 7.2, "13C": 128.5}, {"1H": 6.9, "13C": 115.3}]'
```

## Evaluation signals

- Average cosine score between predicted and ground-truth fingerprints ~0.8450 (normal HSQC) and ~0.8574 (multiplicity-edited HSQC)
- Molecular weight prediction R-squared ~0.9183 (normal HSQC) and ~0.9336 (multiplicity-edited HSQC)
- Top-1 structure-class prediction accuracy ~90.4% (normal) and ~90.8% (edited HSQC)
- Reproduced benchmark deltas: DeepSAT Top-1 annotation 73.5% vs 40.3% for SMART 2.0 (edited HSQC); 41.0% vs 18.7% Top-1 identification (normal HSQC); NMRShiftDB under 3%
- Similarity calls and annotation rates both gated at cosine score threshold 0.8 on Morgan fingerprints

## Limitations

- Searching NMRshiftDB by proton and carbon shifts achieved under 3% correct identifications, so shift-only baseline retrieval is very weak
- DeepSAT's interpretability analysis (occlusion sensitivity) was limited to a single compound (quercetin) and 'these results do not fully explain the decision-making process of DeepSAT'
- Solvent-transfer robustness is incomplete: only over 65% of results were correctly identified when data were recorded in different solvents
- Absolute precision/recall/F1 values are deferred to supplementary tables/figures, and discussion-level text lacks DOI/model-weight availability details

## Evidence

- [abstract] Here, we introduce DeepSAT, a neural network-based structure annotation and scaffold prediction system: "Here, we introduce DeepSAT, a neural network-based structure annotation and scaffold prediction system"
- [results] The thresholds for the peak shift differences were set as 0.5 ppm for carbon signals and 0.05 ppm for proton signals: "The thresholds for the peak shift differences were set as 0.5 ppm for carbon signals and 0.05 ppm for proton signals"
- [results] the average cosine score was 0.8450 for the normal HSQC model, and 0.8574 for the Multiplicity Edited HSQC model: "the average cosine score was 0.8450 for the normal HSQC model, and 0.8574 for the Multiplicity Edited HSQC model"
- [results] we consider two structures to be "similar" if the cosine score is over 0.8 based on the predicted and known Morgan fingerprints: "we consider two structures to be “similar” if the cosine score is over 0.8 based on the predicted and known Morgan fingerprints"
- [readme] You can put in your peaks as a json list of dicts, with 1H,13C as headers: "You can put in your peaks as a json list of dicts, with 1H,13C as headers."
