---
name: energy-based-candidate-scoring
description: Use when you already have per-precursor formula candidate sets (e.g.
  from SIRIUS decomp with 10 ppm window, RDBE-permissive, down-selected by FastFilter
  to top 256) and need to pick the most probable formula/adduct from positive-mode
  MS/MS spectra.
when_to_use_negative:
- Input is negative-mode MS/MS data — models were trained only for positive-mode data
- You need MS1 isotope-pattern scores combined with the model score — MIST-CF does
  not integrate MS1 isotopic information (notably it ties CASMI 2022 winning solution
  with zero MS1 isotopic input)
- You have no formula candidate enumeration step upstream — the skill only scores
  candidates, it does not decompose exact masses itself
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MIST-CF
  role: Energy-based model whose Chemical Formula Transformer scores each candidate
    formula/adduct against the MS/MS spectrum; used as the final ranking model (src/mist_cf/mist_cf_score/predict.py)
  repo: https://github.com/samgoldman97/mist-cf
- name: FastFilter
  role: Lightweight neural network that pre-ranks the large RDBE-permissive candidate
    sets down to the top 256, keeping scoring tractable (src/mist_cf/fast_form_score/predict.py;
    configs/fast_filter.yaml)
  repo: https://github.com/samgoldman97/mist-cf
- name: SIRIUS
  role: Provides the fast dynamic-programming 'decomp' module to enumerate formula
    candidates for an observed MS1 mass; fragmentation-tree scoring is replaced, not
    the mass decomposition
  repo: https://github.com/boecker-lab/sirius/releases/download/
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1021/acs.jcim.3c01082
    title: 'MIST-CF: Chemical Formula Inference from Tandem Mass Spectra'
  source_content_hash: sha256:5a10fbbe84874c69ecd191dca8c1cef4d3e46e8ac2d2b1eb6b0380981c58bf4f
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/samgoldman97/mist-cf
  tools:
  - FastFilter
  - MIST-CF
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# energy-based-candidate-scoring

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Ranks enumerated chemical formula and adduct candidates for an MS/MS precursor by scoring each candidate against the spectrum with the MIST-CF energy-based Chemical Formula Transformer, replacing SIRIUS fragmentation-tree scoring with a learned spectral likelihood-style model.

## When to use

Use when you already have per-precursor formula candidate sets (e.g. from SIRIUS decomp with 10 ppm window, RDBE-permissive, down-selected by FastFilter to top 256) and need to pick the most probable formula/adduct from positive-mode MS/MS spectra.

## When NOT to use

- Input is negative-mode MS/MS data — models were trained only for positive-mode data
- You need MS1 isotope-pattern scores combined with the model score — MIST-CF does not integrate MS1 isotopic information (notably it ties CASMI 2022 winning solution with zero MS1 isotopic input)
- You have no formula candidate enumeration step upstream — the skill only scores candidates, it does not decompose exact masses itself

## Inputs

- Preprocessed positive-mode MS/MS spectra with precursor masses (e.g. NPLIB1, NPLIB1+NIST20, or CASMI2022 spectra in .mgf format)
- Per-precursor enumerated formula candidate lists from SIRIUS decomp (RDBE-permissive, 10 ppm)
- FastFilter-ranked candidate sets (top 256 per spectrum) from src/mist_cf/fast_form_score/predict.py
- Trained MIST-CF model checkpoint (public NPLIB1/CANOPUS-trained or NIST20-trained model)

## Outputs

- Ranked formula (and adduct) candidate scores per spectrum with a top-1 prediction
- Evaluation metrics: top-1 formula accuracy, joint formula+adduct accuracy, SIRIUS failure/coverage rates, per-spectrum concordance (results_table.csv)
- FastFilter candidate-count distribution and top-256 true-formula recovery statistics (fast_filter_stats.csv)

## How to apply

Enumerate formula candidates for each MS1 precursor mass (SIRIUS 'decomp' dynamic-programming module, ~10 ppm window, RDBE-permissive), then down-select with the FastFilter energy model to the top 256 candidates per spectrum (it recovers the true formula ~99% of the time). Score each retained candidate against the MS/MS spectrum with the trained MIST-CF Chemical Formula Transformer (src/mist_cf/mist_cf_score/predict.py), which consumes internal peak-to-subformula assignments (not fragmentation trees), multiple adduct types beyond [M+H]+, sinusoidal formula embeddings, instrument-type covariates, and neutral-loss fragment formulas; the candidate with the top score is the prediction. Evaluate top-1 formula accuracy, joint formula+adduct accuracy, and SIRIUS-vs-MIST-CF concordance with analysis/evaluate_pred.py, writing results to results_table.csv. The rationale is that replacing fragmentation-tree kernels with a data-driven transformer yields ~71% top-1 accuracy on NPLIB1 versus SIRIUS's 48%, while still relying on SIRIUS only for fast mass decomposition.

## Related tools

- **MIST-CF** (Energy-based model whose Chemical Formula Transformer scores each candidate formula/adduct against the MS/MS spectrum; used as the final ranking model (src/mist_cf/mist_cf_score/predict.py)) — https://github.com/samgoldman97/mist-cf
- **FastFilter** (Lightweight neural network that pre-ranks the large RDBE-permissive candidate sets down to the top 256, keeping scoring tractable (src/mist_cf/fast_form_score/predict.py; configs/fast_filter.yaml)) — https://github.com/samgoldman97/mist-cf
- **SIRIUS** (Provides the fast dynamic-programming 'decomp' module to enumerate formula candidates for an observed MS1 mass; fragmentation-tree scoring is replaced, not the mass decomposition) — https://github.com/boecker-lab/sirius/releases/download/

## Examples

```
. quickstart/download_model.sh && . quickstart/run_model.sh
```

## Evaluation signals

- Top-1 formula accuracy on NPLIB1 should approach ~71% (80% joint formula+adduct), versus ~48% for SIRIUS 5.6.3 baseline; flag divergence in results_table.csv
- FastFilter top-256 recovery of the true formula should be ≈99%; verify via fast_filter_stats.csv
- Coverage invariant: MIST-CF predicts every spectrum while SIRIUS fails on ~10.77% of NPLIB1 spectra; check the SIRIUS failure-rate metric
- On CASMI2022 positive-mode challenge spectra, top-1 accuracy should be nearly equivalent to the winning SIRIUS solution without manual postprocessing
- Candidate-set invariant: >15% of training spectra (NPLIB1 and NIST20) have >5,000 RDBE-permissive candidates, so pre-filter candidate-count distributions should show heavy tails before FastFilter truncation

## Limitations

- Trained only on positive-mode data; negative-mode MS/MS is unsupported
- MS1 isotopic scores are not integrated with MIST-CF scores, and adduct switching in MS2 subformula assignment is not considered
- Still depends on SIRIUS's algorithmic decomposition of exact masses into formula candidates
- The public quickstart model (trained on NPLIB1/CANOPUS) may underperform the NIST20-trained model, particularly for Orbitrap or higher-resolution data; NIST20 models are available only upon reasonable request to NIST-licensed users

## Evidence

- [results] FastFilter top-256 recovery and candidate-set size: [evidence span withheld — anchor status: not_found]
- [results] NPLIB1 head-to-head accuracy: [evidence span withheld — anchor status: not_found]
