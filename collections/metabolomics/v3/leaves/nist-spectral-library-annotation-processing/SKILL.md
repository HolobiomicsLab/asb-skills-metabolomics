---
name: nist-spectral-library-annotation-processing
description: Reference for exporting spectra from the NIST20 MS/MS spectral library
  and cleaning expert-curated formula annotations (10 ppm mass consistency, glycan/peptide
  removal) before evaluating formula-annotation precision/recall or retrieval.
when_to_use_negative:
- When the task is de novo spectral denoising or peak picking rather than preparing
  curated annotation ground truth — NIST20 already provides expert-curated formula
  annotations.
- When spectra include glycans, peptides, or unmerged instrument conditions outside
  Orbitrap HCD [M+H]+ — the paper explicitly excluded glycan and peptide spectra and
  limited formula-annotation analysis to small molecules.
- When annotations are predicted by a C2MS model rather than expert-curated — this
  skill prepares ground-truth sets, not model outputs (model predictions are matched
  at 10 ppm tolerance downstream).
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: FraGNNet
  role: Model suite whose data preprocessing pipeline exports and cleans the NIST20
    spectral library and formula annotations used as evaluation ground truth
  repo: github.com/FraGNNet/fragnnet
- name: Pytorch
  role: Implementation framework for FraGNNet and baselines that consume the processed
    annotation data
  repo: github.com/FraGNNet/fragnnet
- name: RDKit
  role: Used for data preprocessing and molecule featurization of NIST20 compounds
    during annotation processing
- name: Python
  role: Language for the preprocessing and analysis scripts that build formula annotation
    tables
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:23b6b6df70ebacc4c5bf39f44a20710f1e7aede01094f91c4a6e5844121bbc88
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
  - RDKit
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# nist-spectral-library-annotation-processing

## Summary

Process expert-curated MS/MS spectra and formula annotations from the NIST20 spectral library to prepare evaluation data: filter glycan/peptide spectra, prune mass-inconsistent annotations, and export spectra for downstream C2MS model comparison.

## When to use

Reference for exporting spectra from the NIST20 MS/MS spectral library and cleaning expert-curated formula annotations (10 ppm mass consistency, glycan/peptide removal) before evaluating formula-annotation precision/recall or retrieval.

## When NOT to use

- When the task is de novo spectral denoising or peak picking rather than preparing curated annotation ground truth — NIST20 already provides expert-curated formula annotations.
- When spectra include glycans, peptides, or unmerged instrument conditions outside Orbitrap HCD [M+H]+ — the paper explicitly excluded glycan and peptide spectra and limited formula-annotation analysis to small molecules.
- When annotations are predicted by a C2MS model rather than expert-curated — this skill prepares ground-truth sets, not model outputs (model predictions are matched at 10 ppm tolerance downstream).

## Inputs

- NIST20 MS/MS expert-curated spectral library spectra (Orbitrap, HCD, [M+H]+ adduct type)
- Expert-curated formula annotations (subformulae of the precursor molecular formula per peak)
- Molecular structures for spectra compounds (for glycan/peptide class filtering and featurization)

## Outputs

- Cleaned per-spectrum formula annotation table mapping peak mass m to annotation set A(m)
- Exported held-out scaffold-split MS/MS spectra suitable for formula-annotation evaluation (glycan/peptide spectra removed, mass-inconsistent annotations dropped)
- Per-peak ground-truth formula sets for computing AR, AWR, AP, AWP, AF1, AWF1 (Eqs. 31-36)

## How to apply

Export spectra from the NIST20 MS/MS spectral library (the paper follows an established export pipeline documented in a GitHub repository), keeping spectra of small molecules and excluding glycan and peptide spectra, which have formula semantics the small-molecule fragment model cannot handle. For each spectrum, use the NIST20 expert-curated formula annotations and drop any annotation whose formula mass lies more than 10 ppm from its associated peak mass, so that formula sets are mass-consistent with the Orbitrap HCD [M+H]+ peaks they annotate. Retain the remaining nonempty per-peak annotation sets as ground truth for classification-style evaluation of predicted formula annotations (Eq. 31-36: AR, AWR, AP, AWP, AF1, AWF1). The resulting annotation table pairs each real peak mass m with set A(m) of subformulae of the precursor formula, and should be merged with the held-out scaffold-split spectra (with expert formula annotations) that models such as FraGNNet-D4 and GrAFF-MS will be scored against.

## Related tools

- **FraGNNet** (Model suite whose data preprocessing pipeline exports and cleans the NIST20 spectral library and formula annotations used as evaluation ground truth) — github.com/FraGNNet/fragnnet
- **Pytorch** (Implementation framework for FraGNNet and baselines that consume the processed annotation data) — github.com/FraGNNet/fragnnet
- **RDKit** (Used for data preprocessing and molecule featurization of NIST20 compounds during annotation processing)
- **Python** (Language for the preprocessing and analysis scripts that build formula annotation tables)

## Examples

```
bash env/env_setup_gpu_py311_cu121_250602.sh; conda activate FRAGNNET-GPU; python -m fragnnet.preproc_scripts.export_nist20_annotations --nist20_export <nist20.msp> --out annotations.csv
```

## Evaluation signals

- Every retained annotation satisfies |formula_mass - peak_mass| / peak_mass <= 10 ppm (epsilon = 1e-5, tolerance scaled by max(m, 200)).
- No glycan or peptide spectra remain in the exported annotation table.
- Each spectrum retains only nonempty annotation sets A(m) derived from NIST20 expert curation (single-fragment degenerate cases excluded where noted).
- Sanity check against published scaffold-split metrics: FraGNNet-D4 should reach AR 0.81, AP 0.98, AWF1 0.95 when this ground truth is used in evaluation.
- Spectra should match the intended acquisition context: Orbitrap instrument with higher-energy collisional dissociation (HCD) and [M+H]+ adduct type.

## Limitations

- Glycan and peptide spectra were deliberately excluded; formula-annotation evaluation is limited to small molecules.
- Annotations whose formula mass deviates by more than 10 ppm from the associated peak are dropped, which can remove ground-truth peaks for some spectra.
- Only Orbitrap HCD [M+H]+ data from NIST20 was used; broader experimental conditions and unmerged spectra remain untested.
- The recursive fragmentation used to generate candidate formulae is not parallelized and may fail to account for complex chemistry such as cyclizations, leaving some OS peaks unexplained.

## Evidence

- [methods] matching tolerance: [evidence span withheld — anchor status: not_found]
- [readme] source code: [evidence span withheld — anchor status: external_doc]
- [readme] interpretability via formula annotations: [evidence span withheld — anchor status: external_doc]
- [readme] preprocessing scripts pointer: [evidence span withheld — anchor status: external_doc]
