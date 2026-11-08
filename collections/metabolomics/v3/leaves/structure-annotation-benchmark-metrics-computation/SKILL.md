---
name: structure-annotation-benchmark-metrics-computation
description: Use when you have held-out 1H-13C HSQC query spectra (normal or multiplicity
  edited) with ground-truth structures and need to score/reproduce Top-1 identification
  rates, annotation rates at cosine > 0.8, and precision/recall/F1@k against SMART
  2.0 or NMRShiftDB shift-search baselines.
when_to_use_negative:
- Query spectra are MS/MS fragmentation or bioactivity/genome-mining data (AntiSMASH,
  BiG-SCAPE/CORASON, GNPS) rather than 1H-13C HSQC spectra
- The goal is training or recalibrating the DeepSAT model itself, not evaluating annotation/search
  performance against baselines
- No ground-truth structures or known Morgan fingerprints exist for the queries, so
  no cosine-threshold scoring at 0.8 can be computed
tools:
- name: DeepSAT
  role: 'Primary annotator under test: infers fingerprint bit strings, molecular weights,
    and structure classes from HSQC spectra, then retrieves candidates by Morgan-fingerprint
    cosine similarity'
  repo: github.com/mwang87/DeepSAT
- name: SMART 2.0
  role: Baseline retrieval system whose shift-threshold matching performance (0.05
    ppm proton / 0.5 ppm carbon) is being scored against DeepSAT
- name: NMRShiftDB
  role: Open reference spectral library (n=53,954) searched by proton/carbon shifts
    as the second baseline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-023-00738-4
    title: 'DeepSAT: Learning Molecular Structures from Nuclear Magnetic Resonance
      Data'
  source_content_hash: sha256:e793a8f4c663d1ff4256bc8e2abd57a0a5da7dfefa2b68d66c52b3ace541fbfe
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
yes_rate: 0.8888888888888888
n_claims: 9
---

# structure-annotation-benchmark-metrics-computation

## Summary

Computes and compares benchmark metrics — Top-1 identification rates, annotation rates (cosine > 0.8 on Morgan fingerprints), and precision/recall/F1@k — for NMR-driven structure annotation systems, reproducing the DeepSAT vs SMART 2.0 vs NMRShiftDB comparison on held-out 1H-13C HSQC test spectra.

## When to use

Use when you have held-out 1H-13C HSQC query spectra (normal or multiplicity edited) with ground-truth structures and need to score/reproduce Top-1 identification rates, annotation rates at cosine > 0.8, and precision/recall/F1@k against SMART 2.0 or NMRShiftDB shift-search baselines.

## When NOT to use

- Query spectra are MS/MS fragmentation or bioactivity/genome-mining data (AntiSMASH, BiG-SCAPE/CORASON, GNPS) rather than 1H-13C HSQC spectra
- The goal is training or recalibrating the DeepSAT model itself, not evaluating annotation/search performance against baselines
- No ground-truth structures or known Morgan fingerprints exist for the queries, so no cosine-threshold scoring at 0.8 can be computed

## Inputs

- Held-out test query 1H-13C HSQC spectra, normal and multiplicity edited (n=3982)
- Ground-truth molecular structures with known Morgan fingerprints for each query spectrum
- DeepSAT-predicted fingerprint bit strings, molecular weights, and structure classes per query
- SMART 2.0 retrieval hits against the in-house HSQC reference library (n=143,467), matched with 0.05 ppm (proton) / 0.5 ppm (carbon) peak-shift thresholds
- NMRShiftDB (n=53,954 reference spectra) chemical-shift search results

## Outputs

- benchmark_comparison_table.csv with Top-1 identification rates, annotation rates, and precision/recall/F1@k per method
- Per-query correctness flags (Top-1 hit matches ground truth; annotation if fingerprint cosine > 0.8)

## How to apply

Load the held-out test query HSQC spectra (n=3982, normal and multiplicity edited) together with their ground-truth structures and known Morgan fingerprints. Run DeepSAT inference on each query to predict fingerprint bit strings, molecular weights, and structure classes, then retrieve candidates by cosine similarity of fingerprints against known small molecules — two structures count as 'similar' if the cosine score exceeds 0.8 on predicted vs known Morgan fingerprints. Run SMART 2.0 as baseline by matching query proton/carbon shifts against the in-house reference HSQC library (n=143,467) with peak shift difference thresholds of 0.05 ppm for proton and 0.5 ppm for carbon signals, and run automated NMRShiftDB (n=53,954) searches by proton and carbon shifts. Score every method identically: a Top-1 identification is correct when the top hit equals the ground truth; an annotation is counted when the cosine score between predicted and known fingerprints exceeds 0.8. Tabulate Top-1 rates, annotation rates, and precision/recall/F1@k into benchmark_comparison_table.csv, then validate reproduced numbers against the reported outcomes (DeepSAT 73.5% Top-1 annotation with edited HSQC vs 40.3% SMART 2.0; 41.0% vs 18.7% identification with normal HSQC; NMRShiftDB under 3%).

## Related tools

- **DeepSAT** (Primary annotator under test: infers fingerprint bit strings, molecular weights, and structure classes from HSQC spectra, then retrieves candidates by Morgan-fingerprint cosine similarity) — github.com/mwang87/DeepSAT
- **SMART 2.0** (Baseline retrieval system whose shift-threshold matching performance (0.05 ppm proton / 0.5 ppm carbon) is being scored against DeepSAT)
- **NMRShiftDB** (Open reference spectral library (n=53,954) searched by proton/carbon shifts as the second baseline)

## Examples

```
curl -X POST http://localhost/api/smart3/search -d '[{"1H": 3.70, "13C": 55.2}]'
```

## Evaluation signals

- DeepSAT Top-1 annotation reaches ~73.5% with multiplicity edited HSQC and ~40.3% for SMART 2.0 on the held-out test set
- Top-1 identification is 41.0% for DeepSAT (normal HSQC) versus 18.7% for SMART 2.0, i.e. over 2-fold
- NMRShiftDB shift search yields under 3% correct identifications
- Mean cosine score between predicted and ground-truth fingerprints is ~0.8450 (normal HSQC) and ~0.8574 (edited HSQC)
- Reproduced F1 at cosine threshold 0.8 shows a two-fold higher F1-score for DeepSAT over SMART 2.0

## Limitations

- Precision/recall/F1 absolute values were deferred to supplementary tables/figures rather than the main discussion text
- Occlusion-sensitivity interpretability was only demonstrated on a single compound (quercetin) and does not fully explain DeepSAT decisions
- Solvent-transfer robustness is incomplete: over 65% of results remained correctly identified across solvents, but not all
- Baseline SMART 2.0-style shift matching only finds candidates present in the reference HSQC library (n=143,467); structures outside it cannot be recovered

## Evidence

- [results] The thresholds for the peak shift differences were set as 0.5 ppm for carbon signals and 0.05 ppm for proton signals: "The thresholds for the peak shift differences were set as 0.5 ppm for carbon signals and 0.05 ppm for proton signals"
- [results] Cosine similarity threshold definition: "we consider two structures to be “similar” if the cosine score is over 0.8 based on the predicted and known Morgan fingerprints"
- [results] Annotation rate threshold: "For the measurement of annotation rate, cosine score of 0.8 was set as the threshold"
- [results] NMRShiftDB baseline outcome: "The results from searching NMRshiftDB by proton and carbon NMR shifts, however, achieved under 3% of correct identifications."
- [discussion] Expected comparative result: "These metrics revealed the significantly improved performance of DeepSAT over SMART 2.0 with a two-fold higher F1-score"
