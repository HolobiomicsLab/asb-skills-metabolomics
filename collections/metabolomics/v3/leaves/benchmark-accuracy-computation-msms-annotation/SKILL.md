---
name: benchmark-accuracy-computation-msms-annotation
description: Use when you have held-out annotated spectra (e.g. NPLIB1 or CASMI22)
  plus candidate formula/adduct sets per spectrum and need a fair top-1/top-k accuracy
  comparison across competing scoring architectures.
when_to_use_negative:
- You only need to predict a formula for a single unknown spectrum without comparing
  architectures or a held-out labelled benchmark (use the MIST-CF predict path instead).
- Your evaluation set is not held out / lacks ground-truth formula labels, so no top-1
  accuracy can be defined.
- You are training or hyperparameter-tuning the models rather than scoring and re-ranking
  their checkpoints.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SIRIUS
  role: Enumerates candidate formulae for each MS1 precursor mass via SIRIUS decomp,
    providing the shared candidate pool that each architecture re-ranks
  repo: https://github.com/boecker-lab/sirius
- name: MIST-CF
  role: Primary scoring model whose checkpoint is evaluated by re-ranking the candidate
    formulae per spectrum
  repo: github.com/samgoldman97/mist-cf
- name: FastFilter
  role: Light-weight neural formula filtering model; one of the benchmarked scoring
    architectures in the eval script
- name: Chemical Formula Transformer
  role: Transformer architecture baseline (xformer_score) whose top-1 accuracy is
    compared against MIST-CF
  repo: github.com/samgoldman97/mist-cf
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jcim.3c01082
    title: 'MIST-CF: Chemical Formula Inference from Tandem Mass Spectra'
  source_content_hash: sha256:92a7bb5454371dfdce0a0ade9fd6a3ad66d4fa4b1e2af6cf9f911122a7fcf259
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/boecker-lab/sirius
  tools:
  - Chemical Formula Transformer
  - FastFilter
  - MIST-CF
  - SIRIUS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# benchmark-accuracy-computation-msms-annotation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Benchmarking skill for comparing top-1 (and top-k) formula-annotation accuracy of energy-based MS/MS models (MIST-CF vs FFN, MS1-Only, Chemical Formula Transformer) by re-ranking enumerated formula/adduct candidates per held-out spectrum and checking each model's top-ranked guess against the ground-truth formula.

## When to use

Use when you have held-out annotated spectra (e.g. NPLIB1 or CASMI22) plus candidate formula/adduct sets per spectrum and need a fair top-1/top-k accuracy comparison across competing scoring architectures.

## When NOT to use

- You only need to predict a formula for a single unknown spectrum without comparing architectures or a held-out labelled benchmark (use the MIST-CF predict path instead).
- Your evaluation set is not held out / lacks ground-truth formula labels, so no top-1 accuracy can be defined.
- You are training or hyperparameter-tuning the models rather than scoring and re-ranking their checkpoints.

## Inputs

- Held-out annotated MS/MS spectra (e.g. NPLIB1/canopus_train split, CASMI22 mgf)
- Per-spectrum ground-truth molecular formula (and adduct) labels
- Enumerated formula/adduct candidate lists per spectrum (SIRIUS decomp output, capped at 256)
- Trained scoring-model checkpoints (MIST-CF, FFN, Xformer, MS1-only)

## Outputs

- benchmarking_results.csv with per-model top-1 (and optionally top-k) accuracy
- Candidate rankings per spectrum per model with scores
- Comparison table against the paper's reported NPLIB1 accuracies (e.g. ~0.741 MIST-CF NPLIB1-only)

## How to apply

For each held-out spectrum, enumerate candidate formula and adduct assignments with SIRIUS decomp (capped at 256 candidates per spectrum), score every candidate with each model variant (MIST-CF, FFN, Xformer, MS1-only), and re-rank candidates by score. Read out the rank of the ground-truth formula and compute top-1 accuracy as the fraction of spectra where the top-scored candidate is correct; optionally record top-k accuracy. Run the same candidate enumeration and candidate cap for every architecture so accuracy differences reflect only the scoring model, not candidate-set differences; the paper reports MIST-CF improving top-1 by roughly 10% over the next best architecture on NPLIB1. Persist the compiled comparison in benchmarking_results.csv.

## Related tools

- **SIRIUS** (Enumerates candidate formulae for each MS1 precursor mass via SIRIUS decomp, providing the shared candidate pool that each architecture re-ranks) — https://github.com/boecker-lab/sirius
- **MIST-CF** (Primary scoring model whose checkpoint is evaluated by re-ranking the candidate formulae per spectrum) — github.com/samgoldman97/mist-cf
- **FastFilter** (Light-weight neural formula filtering model; one of the benchmarked scoring architectures in the eval script)
- **Chemical Formula Transformer** (Transformer architecture baseline (xformer_score) whose top-1 accuracy is compared against MIST-CF) — github.com/samgoldman97/mist-cf

## Examples

```
python run_scripts/benchmarking/eval_models.py && head benchmarking_results.csv
```

## Evaluation signals

- Per-spectrum candidate count is identical across all architectures (same SIRIUS decomp enumeration, ≤256 candidates) so accuracy comparisons are attributable to the scoring model
- Top-1 accuracy on NPLIB1-only split for MIST-CF is approximately 0.741, rising to ~0.769 when NIST20 training spectra are added
- MIST-CF exceeds the next-best architecture by roughly a 10% absolute top-1 accuracy margin over FFN/MS1-Only/Xformer baselines
- Results file benchmarking_results.csv exists and contains one accuracy entry per evaluated model checkpoint
- On CASMI22, MIST-CF accuracy should be nearly equivalent to the winning SIRIUS positive-mode solution

## Limitations

- Results depend on candidate-enumeration quality; candidates outside the SIRIUS decomp list can never be top-1.
- Models described here are trained for positive-mode data only; negative-mode benchmarking is out of scope.
- Adding NIST20 (commercial, licensed) training spectra changes accuracy (~0.741 → ~0.769); public-only evaluations cannot use those checkpoints.
- README notes the public NPLIB1/CANOPUS-trained model 'may be less performant than the model trained on the commercial NIST20 Library (particularly for Orbitrap or higher resolution data)'.
- Does not integrate MIST-CF scores with MS1 isotopic scores or account for adduct switching in MS2 subformula assignment.

## Evidence

- [readme] Run benchmarking evaluation script to re-rank candidates and compute top-1 accuracy: "*Evaluate models*: `run_scripts/benchmarking/eval_models.py`"
- [abstract] MIST-CF improves top-1 accuracy by ~10% over other architectures and gains from NIST20 training: "showing an absolute improvement of 10% top 1 accuracy over other neural network architectures"
- [intro] NPLIB1-only vs NIST20-inclusive training accuracies cited in the source card: "By training and evaluating our model both with and without data from NIST20, we enable reproducible evaluation of select trained models even in the absence of a NIST20 license."
- [discussion] Candidate enumeration relies on SIRIUS decomposition of exact masses: "we still rely upon SIRIUS’s algorithmic decomposition of exact masses into formula candidates due to their fast implementation"
- [intro] Evaluation datasets named as NPLIB1 (GNPS-derived) and NIST20 variant: "We rigorously evaluate the performance of MIST-CF on two datasets: a public dataset subset from the GNPS15 we term NPLIB125,26 and a variant of NPLIB1 including spectra from the commercial NIST20"
- [readme] Quickstart model download and prediction commands for MIST-CF: ". quickstart/download_model.sh
. quickstart/run_model.sh"
