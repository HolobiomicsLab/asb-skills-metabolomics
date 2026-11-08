---
name: msms-peak-formula-annotation-matching
description: Use when comparing predicted formula annotations against expert-curated
  peak annotations on held-out MS/MS spectra (e.g., NIST20 scaffold-split test set)
  to score formula-level agreement of C2MS models such as FraGNNet, GrAFF-MS, or ICEBERG.
when_to_use_negative:
- Input spectra are glycan or peptide spectra — the analysis explicitly excluded glycan
  and peptide spectra and focused on small molecules
- Peak annotations lack formula masses within 10 ppm of their associated peak mass
  — such annotations are excluded and cannot support reliable AP/AR computation
- The goal is compound retrieval ranking (MS2C top-k retrieval by Tanimoto similarity),
  not formula-level peak annotation agreement
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FraGNNet
  role: C2MS model whose predicted formula annotations are evaluated; reference implementation
    with analysis notebooks and scripts
  repo: github.com/FraGNNet/fragnnet
- name: Pytorch
  role: Framework in which FraGNNet and baselines were implemented for spectrum prediction
    and metric computation
- name: Pytorch Lightning
  role: Training/experiment framework used for the underlying C2MS models
  repo: Lightning-AI/pytorch-lightning
- name: RDKit
  role: Data preprocessing and molecule featurization for constructing formula libraries
    and fragment representations
- name: GrAFF-MS
  role: Baseline formula-predictor model compared in the annotation matching evaluation
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:350e8266758534618398fde79f9ec513973f509d726d9b70c70b32bc9919478d
  source_papers:
  - doi: 10.48550/arxiv.2404.02360
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FraGNNet
  - GrAFF-MS
  - Python
  - Pytorch
  - Pytorch Lightning
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# msms-peak-formula-annotation-matching

## Summary

Evaluates formula annotation quality of predicted MS/MS spectra by matching predicted peaks to real peaks under a 10 ppm tolerance and computing annotation recall (AR), annotation precision (AP), and intensity-weighted variants (AWR, AWP) plus harmonic-mean F1 (AF1, AWF1). It treats formula annotation — linking each peak to subformulae of the precursor molecular formula — as a classification problem to quantify whether a C2MS model preserves the chemical information needed for peak annotation.

## When to use

Use when comparing predicted formula annotations against expert-curated peak annotations on held-out MS/MS spectra (e.g., NIST20 scaffold-split test set) to score formula-level agreement of C2MS models such as FraGNNet, GrAFF-MS, or ICEBERG.

## When NOT to use

- Input spectra are glycan or peptide spectra — the analysis explicitly excluded glycan and peptide spectra and focused on small molecules
- Peak annotations lack formula masses within 10 ppm of their associated peak mass — such annotations are excluded and cannot support reliable AP/AR computation
- The goal is compound retrieval ranking (MS2C top-k retrieval by Tanimoto similarity), not formula-level peak annotation agreement

## Inputs

- Held-out NIST20 MS/MS scaffold-split test spectra with expert-curated formula annotations (mzML/merged spectra exported from the NIST20 MS/MS spectral library)
- Predicted MS/MS spectra from FraGNNet-D4, GrAFF-MS, and ICEBERG, each predicted peak carrying a nonempty set of formula annotations derived from the model's formula distribution
- Peak mass tolerance parameters: epsilon = 1e-5, tolerance scaled by max(m, 200)

## Outputs

- Per-model annotation metrics (AR, AWR, AP, AWP, AF1, AWF1) assembled as a Table 3 equivalent
- formula_annotation_metrics.csv

## How to apply

Load held-out scaffold-split NIST20 MS/MS spectra with expert-curated formula annotations, excluding glycan and peptide spectra to focus on small molecules, and drop annotations whose formula mass lies more than 10 ppm from the associated peak mass. Run each model (FraGNNet-D4, GrAFF-MS, ICEBERG) to obtain predicted spectra where each predicted peak carries a nonempty set of formula annotations sampled from the model's formula distribution. Match predicted peaks to real peaks with a relative mass tolerance of 10 ppm (epsilon = 1e-5, tolerance scaled by max(m, 200)), then for each matched peak pair compute set overlap between the real annotation set A(m) and predicted set A(m') to obtain AR (Eq. 31), intensity-weighted recall AWR (Eq. 32), annotation precision AP (Eq. 33), and weighted precision AWP (Eq. 34), followed by harmonic-mean F1 scores AF1 and AWF1 (Eqs. 35-36). Assemble the per-model metrics into a Table 3-style comparison written to formula_annotation_metrics.csv. The rationale is that formula distribution over fragments induces formula annotations per peak, so matching sets — not just masses — reveals whether the model preserves interpretable chemical structure assignments.

## Related tools

- **FraGNNet** (C2MS model whose predicted formula annotations are evaluated; reference implementation with analysis notebooks and scripts) — github.com/FraGNNet/fragnnet
- **Pytorch** (Framework in which FraGNNet and baselines were implemented for spectrum prediction and metric computation)
- **Pytorch Lightning** (Training/experiment framework used for the underlying C2MS models) — Lightning-AI/pytorch-lightning
- **RDKit** (Data preprocessing and molecule featurization for constructing formula libraries and fragment representations)
- **GrAFF-MS** (Baseline formula-predictor model compared in the annotation matching evaluation)

## Examples

```
ppm_tol = 1e-5 * max(peak_mass, 200)  # 10 ppm relative tolerance, epsilon=1e-5, scaled by max(m, 200)
```

## Evaluation signals

- FraGNNet-D4 should attain AR 0.81, AP 0.98, and AWF1 0.95 on the NIST20 scaffold split and show the best recall/precision balance (superior F1) among compared models
- Every matched peak pair must satisfy the relative mass tolerance of 10 ppm (epsilon = 1e-5, scaled by max(m, 200))
- Annotations whose formula mass deviates more than 10 ppm from the associated peak mass must be dropped before scoring
- Glycan and peptide spectra must be absent from the evaluation set
- Each predicted peak should carry a nonempty set of formula annotations for overlap computation against the real annotation set A(m)

## Limitations

- The underlying fragmentation algorithm fails to account for some fragments, as evidenced by unexplained OS peaks in the spectra
- Unexplained peaks may arise from complex chemistry such as cyclizations that dramatically expand the fragment search space and are challenging to model
- Performance was not yet evaluated on unmerged spectra or under broader experimental conditions (different instrument platforms and precursor adducts); the evaluation relies on Orbitrap HCD [M+H]+ data
- Cases where only a single fragment is possible (degenerate formulae) are excluded from agreement analysis

## Evidence

- [readme] Interpretability via formula annotations: [evidence span withheld — anchor status: external_doc]
