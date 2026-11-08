---
name: ppm-mass-tolerance-peak-matching
description: Use when comparing predicted spectra from C2MS models (FraGNNet, GrAFF-MS,
  ICEBERG) against real NIST20 MS/MS spectra and you need peak-level or formula-annotation-level
  agreement metrics under a ppm mass-accuracy tolerance.
when_to_use_negative:
- Input spectra are glycan or peptide spectra — these were explicitly excluded from
  the formula annotation analysis to focus on small molecules
- You need absolute peak-intensity error or spectral similarity metrics (e.g. cosine)
  rather than mass-matched annotation overlap
- Formula annotations with masses more than 10 ppm from their associated peak mass
  have not been filtered — leaving them in corrupts the precision/recall denominators
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: FraGNNet
  role: C2MS model producing predicted peaks with formula annotations; reference model
    expected to attain AR 0.81, AP 0.98, AWF1 0.95
  repo: github.com/FraGNNet/fragnnet
- name: GrAFF-MS
  role: Baseline formula-predictor C2MS model whose predicted peaks are matched under
    the same 10 ppm tolerance
- name: ICEBERG
  role: Baseline C2MS model compared in the formula annotation metrics table
- name: Pytorch
  role: Framework in which the models and matching/evaluation code were implemented
  repo: Lightning-AI/pytorch-lightning
- name: RDKit
  role: Used for data preprocessing and molecule featurization before spectrum prediction
    and matching
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:01fae927e09511fa0daffdcff30898468f7b0857bddfe72008d228dd5e347ace
  source_papers:
  - doi: 10.48550/arxiv.2404.02360
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FraGNNet
  - GrAFF-MS
  - ICEBERG
  - Python
  - Pytorch
  - RDKit
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# ppm-mass-tolerance-peak-matching

## Summary

Matches predicted MS/MS peaks to real peaks using a relative mass tolerance of 10 ppm (epsilon = 1e-5, scaled by max(m, 200)) to enable formula annotation recall/precision (AR/AP/AWR/AWP) comparison of C2MS models on NIST20 scaffold-split spectra.

## When to use

Use when comparing predicted spectra from C2MS models (FraGNNet, GrAFF-MS, ICEBERG) against real NIST20 MS/MS spectra and you need peak-level or formula-annotation-level agreement metrics under a ppm mass-accuracy tolerance.

## When NOT to use

- Input spectra are glycan or peptide spectra — these were explicitly excluded from the formula annotation analysis to focus on small molecules
- You need absolute peak-intensity error or spectral similarity metrics (e.g. cosine) rather than mass-matched annotation overlap
- Formula annotations with masses more than 10 ppm from their associated peak mass have not been filtered — leaving them in corrupts the precision/recall denominators

## Inputs

- Held-out NIST20 MS/MS scaffold-split test spectra with expert-curated formula annotations
- Predicted spectra from FraGNNet-D4, GrAFF-MS, and ICEBERG, each peak with a nonempty set of formula annotations
- Per-peak intensities for weighted recall/precision computations

## Outputs

- Matched peak pairs (real peak m, predicted peak m') within 10 ppm relative tolerance
- Per-model formula annotation metrics: AR, AWR, AP, AWP, AF1, AWF1 (Table 3 equivalent)
- formula_annotation_metrics.csv

## How to apply

Load held-out NIST20 scaffold-split test spectra with expert-curated formula annotations, excluding glycan and peptide spectra and dropping annotations whose formula mass lies more than 10 ppm from the associated peak mass. Run each model to produce predicted peaks, each carrying its nonempty set of formula annotations from the model's formula distribution. Match predicted peaks to real peaks using a relative mass tolerance of 10 ppm with epsilon = 1e-5, with tolerance scaled by max(m, 200) to avoid over-tight matching at very low masses. For each matched peak pair, intersect the real annotation set A(m) with the predicted set A(m') to compute annotation recall (AR, Eq. 31), intensity-weighted recall (AWR, Eq. 32), annotation precision (AP, Eq. 33), and weighted precision (AWP, Eq. 34), then derive harmonic-mean F1 scores AF1 and AWF1 (Eqs. 35-36).

## Related tools

- **FraGNNet** (C2MS model producing predicted peaks with formula annotations; reference model expected to attain AR 0.81, AP 0.98, AWF1 0.95) — github.com/FraGNNet/fragnnet
- **GrAFF-MS** (Baseline formula-predictor C2MS model whose predicted peaks are matched under the same 10 ppm tolerance)
- **ICEBERG** (Baseline C2MS model compared in the formula annotation metrics table)
- **Pytorch** (Framework in which the models and matching/evaluation code were implemented) — Lightning-AI/pytorch-lightning
- **RDKit** (Used for data preprocessing and molecule featurization before spectrum prediction and matching)

## Examples

```
python scripts/evaluate_formula_annotations.py --split scaffold --tolerance_ppm 10 --epsilon 1e-5 --out formula_annotation_metrics.csv
```

## Evaluation signals

- FraGNNet-D4 should attain AR 0.81, AP 0.98, and AWF1 0.95 on the NIST20 scaffold split
- FraGNNet-D4 should show the best recall/precision balance (superior F1) among FraGNNet-D4, GrAFF-MS, and ICEBERG
- All annotations retained must satisfy |formula mass - peak mass| <= 10 ppm; annotations beyond this were excluded
- Every predicted peak used in scoring carries a nonempty set of formula annotations derived from the model's formula distribution
- Output CSV (formula_annotation_metrics.csv) matches the Table 3 per-model metric schema (AR, AWR, AP, AWP, AF1, AWF1)

## Limitations

- The fragmentation algorithm fails to account for some fragments, evidenced by unexplained Outside-of-Support (OS) peaks, which caps achievable annotation recall
- Unexplained peaks may arise from complex chemistry such as cyclizations that dramatically expand the fragment search space and are challenging to model
- Performance has not been evaluated on unmerged spectra or under broader experimental conditions (different instrument platforms and precursor adducts); the benchmark covers Orbitrap HCD data with [M+H]+ adducts

## Evidence

- [readme] NIST20 benchmark data: [evidence span withheld — anchor status: external_doc]
