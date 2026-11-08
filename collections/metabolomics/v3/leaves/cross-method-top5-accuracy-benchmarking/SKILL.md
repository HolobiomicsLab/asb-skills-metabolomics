---
name: cross-method-top5-accuracy-benchmarking
description: Use when you need to quantify how robust competing molecular-formula
  identification methods are to increasing spectral noise, on matched MGF test spectra
  from Q-TOF or Orbitrap instruments.
when_to_use_negative:
- Spectra are already clean/no noise-robustness question — use the standard test-set
  evaluation instead
- Input spectra are chimeric (co-eluting mixes) — the article evaluates chimeric degradation
  separately, not via Gaussian-noise augmentation
- Input compounds fall outside the frequent-atom set (C, H, O, N, F, S, Cl, P, B,
  I, Br, Na, K) or exceed 1500 m/z, outside the benchmark's filter scope
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FIDDLE
  role: Primary deep-learning formula predictor evaluated under noise; run with instrument-specific
    TCN and rescore weights via running_scripts/experiments_test_noised.sh
  repo: https://github.com/JosieHong/FIDDLE
- name: BUDDY
  role: Bottom-up baseline formula identifier (msbuddy) run on identical noised spectra
    for comparison
  repo: https://github.com/Philipbear/msbuddy
- name: SIRIUS
  role: Top-down baseline formula identifier (SIRIUS 6) run on identical noised spectra
    for comparison
  repo: https://github.com/boecker-lab/sirius
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:8c578b457a52b86ade72b38820575839465b59d61dd6df8bc0afb14ca499d320
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/JosieHong/FIDDLE
  repo_url: https://github.com/JosieHong/FIDDLE
  tools:
  - BUDDY
  - FIDDLE
  - Q-TOF
  - SIRIUS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# Cross-method top-5 accuracy benchmarking under spectral noise

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks formula-identification tools (FIDDLE, BUDDY, SIRIUS) by perturbing test MS/MS spectra with Gaussian noise and comparing top-5 formula accuracy per instrument (Q-TOF vs Orbitrap), reproducing Fig. 2g-style robustness curves.

## When to use

Use when you need to quantify how robust competing molecular-formula identification methods are to increasing spectral noise, on matched MGF test spectra from Q-TOF or Orbitrap instruments.

## When NOT to use

- Spectra are already clean/no noise-robustness question — use the standard test-set evaluation instead
- Input spectra are chimeric (co-eluting mixes) — the article evaluates chimeric degradation separately, not via Gaussian-noise augmentation
- Input compounds fall outside the frequent-atom set (C, H, O, N, F, S, Cl, P, B, I, Br, Na, K) or exceed 1500 m/z, outside the benchmark's filter scope

## Inputs

- 1000 randomly sampled test MS/MS spectra in MGF format with title, precursor_mz, precursor_type, and collision_energy fields
- FIDDLE pre-trained TCN and rescore checkpoints per instrument (fiddle_tcn_qtof.pt, fiddle_rescore_qtof.pt, fiddle_tcn_orbitrap.pt, fiddle_rescore_orbitrap.pt)
- FIDDLE config yml per instrument
- Ground-truth molecular formulas for the test spectra

## Outputs

- noised_accuracy_results.csv with top-5 formula accuracy per tool, per noise level, per instrument
- Robustness curves comparing FIDDLE, BUDDY, and SIRIUS accuracy vs noise intensity (Fig. 2g equivalent)

## How to apply

Randomly sample 1000 test MS/MS spectra in MGF format (with title, precursor_mz, precursor_type, and collision_energy fields) from the benchmark test set. Perturb each spectrum with additive Gaussian noise (mean 0, standard deviation 0.1 — the paper's augmentation convention) at five increasing intensity-noise levels. Run FIDDLE using instrument-matched pre-trained weights (fiddle_tcn_qtof.pt + fiddle_rescore_qtof.pt, or fiddle_tcn_orbitrap.pt + fiddle_rescore_orbitrap.pt) via running_scripts/experiments_test_noised.sh, applying precursor mass tolerances of 10 ppm for Q-TOF and 5 ppm for Orbitrap. Run BUDDY (msbuddy) and SIRIUS 6 on the identical noised spectra, then compute top-5 formula accuracy per tool, per noise level, per instrument, writing results to noised_accuracy_results.csv.

## Related tools

- **FIDDLE** (Primary deep-learning formula predictor evaluated under noise; run with instrument-specific TCN and rescore weights via running_scripts/experiments_test_noised.sh) — https://github.com/JosieHong/FIDDLE
- **BUDDY** (Bottom-up baseline formula identifier (msbuddy) run on identical noised spectra for comparison) — https://github.com/Philipbear/msbuddy
- **SIRIUS** (Top-down baseline formula identifier (SIRIUS 6) run on identical noised spectra for comparison) — https://github.com/boecker-lab/sirius

## Examples

```
bash running_scripts/experiments_test_noised.sh
```

## Evaluation signals

- At the highest noise level, top-5 accuracies should reproduce Fig. 2g values: Q-TOF FIDDLE 95.0% vs BUDDY 93.5% vs SIRIUS 52.5%
- At the highest noise level on Orbitrap, expect FIDDLE 93.3% vs BUDDY 75.4% vs SIRIUS 63.1%
- noised_accuracy_results.csv must contain entries for all 3 tools × 5 noise levels × 2 instrument types
- FIDDLE overall test performance sanity check: 88.3% top-1 and 93.6% top-5 accuracy on the clean evaluation set
- Per-spectrum MGF inputs must contain the required title, precursor_mz, precursor_type, and collision_energy fields before perturbation

## Limitations

- The Gaussian noise convention (mean 0, sd 0.1) was a Q-TOF training augmentation; no data augmentation was applied for Orbitrap training
- FIDDLE's accuracy varies across experimental conditions with less training data; results may not transfer to underrepresented instruments or adduct types
- SIRIUS web services are for academic research/education use only; commercial redistribution is prohibited

## Evidence
