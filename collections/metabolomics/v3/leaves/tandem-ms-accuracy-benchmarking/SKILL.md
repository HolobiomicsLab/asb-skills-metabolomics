---
name: tandem-ms-accuracy-benchmarking
description: Use when comparing MS/MS formula-prediction methods on a labeled test
  set (e.g. NPLIB1, CASMI2022) and need rigorous metrics such as top-1 accuracy, joint
  formula+adduct accuracy, and per-spectrum concordance.
when_to_use_negative:
- When the test set is unlabeled or ground-truth formulae are unavailable — accuracy
  benchmarking requires per-spectrum annotations
- When only a single method's predictions need rescoring rather than a head-to-head
  comparison across competing pipelines
- When evaluating negative-ion-mode data — models in this paper were trained only
  on positive-mode spectra
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: MIST-CF
  role: Candidate scoring model whose formula predictions are benchmarked against
    SIRIUS on the test set
  repo: github.com/samgoldman97/mist-cf
- name: SIRIUS
  role: Baseline de novo formula annotation tool run headless with tree/compound timeouts;
    also supplies candidate decomposition via `SIRIUS decomp`
  repo: https://github.com/boecker-lab/sirius
- name: FastFilter
  role: Lightweight neural network that pre-ranks SIRIUS-decomp candidates to the
    top-256 per precursor before MIST-CF scoring
  repo: github.com/samgoldman97/mist-cf
- name: MZmine
  role: Cited as a widely used scoring tool comparable to SIRIUS in the MS/MS formula-annotation
    landscape
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jcim.3c01082
    title: 'MIST-CF: Chemical Formula Inference from Tandem Mass Spectra'
  source_content_hash: sha256:16c93ef358599545830e61151e6a83ac945def855b0ed4cc01d0f422423b0839
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - FastFilter
  - MIST-CF
  - MZmine
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# tandem-ms-accuracy-benchmarking

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Head-to-head benchmarking of de novo chemical formula annotation models (MIST-CF vs SIRIUS 5.6.3) on tandem MS test sets like NPLIB1, comparing top-1 formula accuracy, joint formula+adduct accuracy, and failure rates.

## When to use

Use when comparing MS/MS formula-prediction methods on a labeled test set (e.g. NPLIB1, CASMI2022) and need rigorous metrics such as top-1 accuracy, joint formula+adduct accuracy, and per-spectrum concordance.

## When NOT to use

- When the test set is unlabeled or ground-truth formulae are unavailable — accuracy benchmarking requires per-spectrum annotations
- When only a single method's predictions need rescoring rather than a head-to-head comparison across competing pipelines
- When evaluating negative-ion-mode data — models in this paper were trained only on positive-mode spectra

## Inputs

- Preprocessed MS/MS test spectra (MGF format, e.g. canopus_train NPLIB1 subset)
- Candidate formula lists per precursor (from SIRIUS decomp or equivalent)
- Trained MIST-CF model checkpoint and FastFilter model
- SIRIUS 5.6.3 headless binary with SIRIUS_PATH configured
- Ground-truth formula/adduct labels for the test set

## Outputs

- Per-spectrum top-1 formula predictions for each method
- Metrics table (results_table.csv): top-1 accuracy, joint formula+adduct accuracy, failure rate, per-spectrum concordance
- Divergence flags when reproduced numbers deviate from published reference values

## How to apply

Load the preprocessed test spectra (e.g. NPLIB1 from the canopus_train split) and run each candidate pipeline end-to-end: for MIST-CF, enumerate formula candidates via SIRIUS's dynamic programming `SIRIUS decomp` module, down-select with the FastFilter model to the top-256 per precursor (10 ppm candidate masses), then score with the trained MIST-CF energy-based model. For SIRIUS, run headless with its default fragmentation-tree scoring and explicit tree/compound timeouts (300 s). Evaluate both outputs with a shared evaluator (e.g. analysis/evaluate_pred.py) computing top-1 formula accuracy, joint formula+adduct accuracy, failure rate (spectra with no prediction), and per-spectrum concordance including cases where one model fails and the other succeeds. Aggregate into a single results table (e.g. results_table.csv) and compare against published reference numbers — the paper reports MIST-CF at 71% top-1 / 80% joint vs SIRIUS 48% on NPLIB1 with a ~10.77% SIRIUS failure rate — flagging any divergence.

## Related tools

- **MIST-CF** (Candidate scoring model whose formula predictions are benchmarked against SIRIUS on the test set) — github.com/samgoldman97/mist-cf
- **SIRIUS** (Baseline de novo formula annotation tool run headless with tree/compound timeouts; also supplies candidate decomposition via `SIRIUS decomp`) — https://github.com/boecker-lab/sirius
- **FastFilter** (Lightweight neural network that pre-ranks SIRIUS-decomp candidates to the top-256 per precursor before MIST-CF scoring) — github.com/samgoldman97/mist-cf
- **MZmine** (Cited as a widely used scoring tool comparable to SIRIUS in the MS/MS formula-annotation landscape)

## Examples

```
. run_scripts/public_data_train/train_mist_cf.sh; . run_scripts/benchmarking/eval_models.py
```

## Evaluation signals

- Reproduced top-1 formula accuracy should land near the published 71% for MIST-CF and 48% for SIRIUS on NPLIB1; large deviations indicate a pipeline misconfiguration
- SIRIUS failure rate (spectra with no prediction) should be ~10.77%; a much lower value suggests spectra were incorrectly preprocessed or the timeout was too generous
- Joint formula+adduct accuracy for MIST-CF should be higher than top-1-only (~80% vs 71%), confirming adduct predictions are consistent with formula predictions
- Per-spectrum concordance should show a nonzero fraction of spectra where MIST-CF is correct and SIRIUS fails (paper: 36% of SIRIUS-failed spectra); zero would indicate a data-label mismatch
- The output results_table.csv should contain the same metric columns for both methods, computed on the identical spectrum subset, so comparison is fair

## Limitations

- MIST-CF models were trained only for positive-mode data; negative-mode benchmarking is out of scope
- Results depend on the candidate candidate-generation step — MIST-CF still relies on SIRIUS's dynamic programming decomposition of exact masses, so errors in candidate enumeration propagate
- The quickstart MIST-CF model (public NPLIB1/CANOPUS training) may underperform on Orbitrap/high-resolution data compared to models trained on commercial NIST20, so benchmark numbers are training-set dependent

## Evidence

- [results] Head-to-head metric: [evidence span withheld — anchor status: not_found]
