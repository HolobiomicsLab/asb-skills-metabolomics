---
name: benchmark-evaluation-on-nplib1
description: Use when validating de novo chemical formula prediction from tandem MS/MS
  against reported top-1 accuracy on NPLIB1, or when quantifying how peak visibility
  (Np) affects model accuracy, including the MS1-only equivalence baseline.
when_to_use_negative:
- The input spectra are negative ion mode — MIST-CF models were trained only on positive-mode
  data, so accuracy claims do not transfer
- The goal is structure (fingerprint) prediction or adduct/isotopic-score integration
  rather than de novo formula annotation, since MIST-CF does not directly address
  MS1 isotopic score integration or adduct switching in MS2 subformula assignment
- The evaluation target is CASMI 2022 prospective challenge performance rather than
  the NPLIB1/NIST20-style benchmark; that is a separate evaluation protocol
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MIST-CF
  role: Energy-based formula transformer model whose top-1 formula predictions on
    NPLIB1 are the accuracy being benchmarked and peak-ablated
  repo: https://github.com/samgoldman97/mist-cf
- name: SIRIUS
  role: Provides SIRIUS decomp dynamic-programming enumeration of candidate formulae
    for each observed MS1 mass prior to model scoring
  repo: https://github.com/boecker-lab/sirius/releases/download/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jcim.3c01082
    title: 'MIST-CF: Chemical Formula Inference from Tandem Mass Spectra'
  source_content_hash: sha256:82f14af038149a45247cc409381678ff52374f4ce222acf73a5d238b609774b4
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/samgoldman97/mist-cf
  tools:
  - MIST-CF
  - NPLIB1
  - SIRIUS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# benchmark-evaluation-on-nplib1

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Reproduce benchmark evaluation of MIST-CF chemical formula annotation accuracy on the NPLIB1 test set (a public natural products dataset extracted from GNPS), including peak-ablation sweeps where each spectrum is subsampled to its Np highest-intensity, subformula-annotated MS2 peaks.

## When to use

Use when validating de novo chemical formula prediction from tandem MS/MS against reported top-1 accuracy on NPLIB1, or when quantifying how peak visibility (Np) affects model accuracy, including the MS1-only equivalence baseline.

## When NOT to use

- The input spectra are negative ion mode — MIST-CF models were trained only on positive-mode data, so accuracy claims do not transfer
- The goal is structure (fingerprint) prediction or adduct/isotopic-score integration rather than de novo formula annotation, since MIST-CF does not directly address MS1 isotopic score integration or adduct switching in MS2 subformula assignment
- The evaluation target is CASMI 2022 prospective challenge performance rather than the NPLIB1/NIST20-style benchmark; that is a separate evaluation protocol

## Inputs

- MIST-CF predictions or trained MIST-CF model checkpoint on the NPLIB1 test set (MGF spectra such as data/demo_specs.mgf style)
- NPLIB1 preprocessed dataset from Zenodo record 8151490 (public GNPS-derived natural products spectra)
- SIRIUS-decomposed formula candidate lists per MS1 precursor mass
- Subformula annotations per MS2 peak (peak-to-subformula assignment)

## Outputs

- Top-1 chemical formula accuracy per Np setting on the NPLIB1 test set
- subpeak_eval_results.csv with one row per Np restriction value (0, 1, 3, 5, 10, 20, 50) and the MS1-only equivalence baseline
- Evaluated model outputs from run_scripts/benchmarking/eval_models.py

## How to apply

Clone the samgoldman97/mist-cf repository and set up the environment (mamba env create -f environment.yml; pip install -r requirements.txt), installing SIRIUS 5.5.7 headless and setting SIRIUS_PATH because candidate formula enumeration relies on SIRIUS's fast exact-mass decomposition. Use the released NPLIB1-trained MIST-CF model (or a locally trained one via run_scripts/benchmarking/train_mist_cf.sh) and run evaluation with run_scripts/benchmarking/eval_models.py to produce top-1 formula accuracies on the NPLIB1 test set. For the peak-ablation sweep, for each Np restriction (0, 1, 3, 5, 10, 20, 50) subsample every spectrum to its Np intensity-ranked, subformula-annotated MS2 peaks, recompute top-1 accuracy per setting, and tabulate results (expected: 0.623 at Np=0 matching the MS1-only baseline; 0.673, 0.719, 0.729, 0.754, 0.756, 0.774 for increasing Np) into a per-Np CSV such as subpeak_eval_results.csv. Judge correctness by matching each per-Np accuracy within reasonable numerical tolerance and confirming one output row per Np setting.

## Related tools

- **MIST-CF** (Energy-based formula transformer model whose top-1 formula predictions on NPLIB1 are the accuracy being benchmarked and peak-ablated) — https://github.com/samgoldman97/mist-cf
- **SIRIUS** (Provides SIRIUS decomp dynamic-programming enumeration of candidate formulae for each observed MS1 mass prior to model scoring) — https://github.com/boecker-lab/sirius/releases/download/

## Examples

```
python run_scripts/benchmarking/eval_models.py  # after mamba env create -f environment.yml && export SIRIUS_PATH=/path/to/sirius
```

## Evaluation signals

- Per-Np top-1 accuracies match the reported sequence 0.623 (Np=0, MS1-only equivalence), 0.673, 0.719, 0.729, 0.754, 0.756, 0.774 within reasonable numerical tolerance
- subpeak_eval_results.csv contains exactly one row per Np setting (0, 1, 3, 5, 10, 20, 50) and the MS1-only baseline
- Accuracy is monotonically non-decreasing as Np increases, with Np=0 equal to the MS1-only baseline (0.623)
- Evaluation run completes only after SIRIUS_PATH is set, since formula candidate decomposition depends on the SIRIUS executable
- Evaluated model must be a positive-mode model trained on NPLIB1 (public) or NPLIB1+NIST20 data, not a negative-mode checkpoint

## Limitations

- Models were trained only for positive-mode data; negative-mode support is absent
- The publicly released quickstart model (NPLIB1/CANOPS-trained) may be less performant than NIST20-trained models, particularly for Orbitrap or higher-resolution data; NIST20-trained model links require a NIST license
- MIST-CF still relies on SIRIUS's algorithmic decomposition of exact masses into formula candidates, and does not integrate MS1 isotopic scores or consider adduct switching in MS2 subformula assignment

## Evidence

- [methods] NPLIB1: A public natural products dataset extracted from the GNPS database: "NPLIB1: A public natural products dataset extracted from the GNPS database"
- [methods] Evaluate models script: "*Evaluate models*: `run_scripts/benchmarking/eval_models.py`"
- [readme] NPLIB1 usage in README: "NPLIB1 is used for model training and evaluation."
- [discussion] Positive-mode only scope: "We have trained models only for positive-mode data"
- [discussion] SIRIUS dependence for candidate enumeration: "we still rely upon SIRIUS’s algorithmic decomposition of exact masses into formula candidates due to their fast implementation"
