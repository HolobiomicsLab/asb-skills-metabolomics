---
name: annotation-precision-recall-f1-evaluation
description: Use when comparing predicted peak-level formula annotations against expert-curated
  annotations (e.g. NIST20) to score a spectrum-prediction model's annotation quality
  with recall, precision, and F1.
when_to_use_negative:
- Input spectra lack expert-curated formula annotations, so no ground-truth annotation
  sets A(m) exist to compare against
- The goal is retrieval-based MS2C ranking (Top-k candidate scoring by Tanimoto similarity),
  not per-peak annotation accuracy
- Spectra are glycan or peptide spectra — the original analysis excluded these and
  focused on small molecules
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0091
tools:
- name: FraGNNet
  role: Primary model whose formula distribution yields predicted peak annotation
    sets to be scored
  repo: https://github.com/FraGNNet/fragnnet
- name: GrAFF-MS
  role: Baseline formula predictor whose top product and neutral loss formulae are
    scored with the same annotation metrics
- name: Pytorch
  role: Implementation framework for running the compared models and computing the
    annotation metrics
- name: PyTorch Lightning
  role: Training/experiment harness for the model and baselines whose annotations
    are evaluated
  repo: https://github.com/Lightning-AI/pytorch-lightning
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:e453f9e87c46cb4f97d88f0c8a07fd4daaa3d79a7d6288cee40760f14a4398a0
  source_papers:
  - doi: 10.48550/arxiv.2404.02360
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/FraGNNet/fragnnet
  tools:
  - FraGNNet
  - GrAFF-MS
  - PyTorch Lightning
  - Python
  - Pytorch
  - RDKit
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# annotation-precision-recall-f1-evaluation

## Summary

Evaluates predicted formula annotations on MS/MS peaks by matching predicted peaks to expert-curated real peaks within a 10 ppm tolerance, then computing annotation recall (AR), intensity-weighted recall (AWR), annotation precision (AP), weighted precision (AWP), and their harmonic-mean F1 scores (AF1, AWF1). This scores how faithfully a C2MS model's formula distribution reproduces the chemical information needed for peak annotation.

## When to use

Use when comparing predicted peak-level formula annotations against expert-curated annotations (e.g. NIST20) to score a spectrum-prediction model's annotation quality with recall, precision, and F1.

## When NOT to use

- Input spectra lack expert-curated formula annotations, so no ground-truth annotation sets A(m) exist to compare against
- The goal is retrieval-based MS2C ranking (Top-k candidate scoring by Tanimoto similarity), not per-peak annotation accuracy
- Spectra are glycan or peptide spectra — the original analysis excluded these and focused on small molecules

## Inputs

- Held-out NIST20 MS/MS scaffold-split test spectra with expert-curated formula annotations (glycan and peptide spectra excluded)
- Predicted spectra from FraGNNet-D4, GrAFF-MS, and ICEBERG, each predicted peak carrying a nonempty set of formula annotations
- Real peak masses and intensities with associated annotation sets A(m)

## Outputs

- Per-model AR, AWR, AP, AWP, AF1, AWF1 metrics (Table 3 equivalent)
- formula_annotation_metrics.csv with per-model annotation metrics

## How to apply

Load held-out test spectra (NIST20 MS/MS scaffold split) with expert-curated formula annotations, excluding glycan and peptide spectra and dropping annotations whose formula mass lies more than 10 ppm from the associated peak mass. Run the C2MS models (FraGNNet-D4, GrAFF-MS, ICEBERG) so each predicted peak carries its nonempty set of formula annotations from the model's formula distribution. Match predicted peaks to real peaks with a relative mass tolerance of 10 ppm (epsilon = 1e-5, tolerance scaled by max(m, 200)). For each matched peak pair, compute overlap between real annotation set A(m) and predicted annotation set A(m') to get AR (Eq. 31), AWR (Eq. 32), AP (Eq. 33), and AWP (Eq. 34), then take harmonic means for AF1 and AWF1 (Eqs. 35-36). Assemble per-model metrics into the Table 3 equivalent and export as formula_annotation_metrics.csv.

## Related tools

- **FraGNNet** (Primary model whose formula distribution yields predicted peak annotation sets to be scored) — https://github.com/FraGNNet/fragnnet
- **GrAFF-MS** (Baseline formula predictor whose top product and neutral loss formulae are scored with the same annotation metrics)
- **Pytorch** (Implementation framework for running the compared models and computing the annotation metrics)
- **PyTorch Lightning** (Training/experiment harness for the model and baselines whose annotations are evaluated) — https://github.com/Lightning-AI/pytorch-lightning

## Examples

```
matched = abs(m_pred - m_ref) <= 1e-5 * max(m_ref, 200)  # 10 ppm tolerance scaled by max(m, 200), then AR/AP from |A(m) & A(m')|
```

## Evaluation signals

- FraGNNet-D4 should attain AR 0.81, AP 0.98, AWF1 0.95 on the NIST20 scaffold split and show the best recall/precision balance (superior F1) among compared models
- Peak matching must use tolerance = 1e-5 * max(m, 200), i.e. 10 ppm scaled by max(mass, 200)
- Harmonic-mean identities must hold: AF1 = 2*AR*AP/(AR+AP) and AWF1 = 2*AWR*AWP/(AWR+AWP)
- Every predicted peak must carry a nonempty formula annotation set before matching
- Output CSV must contain one row per model with all six metrics (AR, AWR, AP, AWP, AF1, AWF1)

## Limitations

- Annotations with formula masses more than 10 ppm away from their associated peak mass are excluded, so very-low-mass peaks (below 200 Da) use a wider effective tolerance floor
- Glycan and peptide spectra are excluded; results apply to small molecules
- Cases where only a single fragment is possible are excluded from the agreement analysis as degenerate
- The fragmentation algorithm fails to account for some fragments (unexplained Outside-of-Support peaks), which can depress recall, especially for complex chemistry such as cyclizations

## Evidence

- [methods] Annotation precision/recall computed via set overlap at 10 ppm peak tolerance: "we excluded annotations with formula masses more than 10 ppm away from their associated peak mass"
- [methods] Formula annotation task framed as classification, enabling precision/recall metrics: "We pose the formula annotation task as a classification problem"
- [methods] Scope restriction to small molecules excludes glycan/peptide spectra: "we excluded glycan and peptide spectra from our formula annotation analysis and focused on small molecules"
- [readme] Model interpretability via formula/fragment annotations is the capability being scored: [evidence span withheld — anchor status: external_doc]
