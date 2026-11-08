---
name: msms-spectral-noise-perturbation
description: Use when evaluating robustness of formula-identification methods on MS/MS
  spectra (MGF format) by systematically injecting intensity noise and comparing top-k
  accuracy degradation across instruments.
when_to_use_negative:
- The input is unlabeled spectra with no known ground-truth formulas — accuracy against
  reference formulas cannot be computed
- The goal is to augment training data rather than evaluate robustness — note the
  paper found noise augmentation helped Q-TOF but was not applied for Orbitrap training
  because it lowered similarity
- The spectra are chimeric/co-eluting — that robustness axis (spectral mixing) is
  a separate evaluation, not noise perturbation
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: FIDDLE
  role: Deep learning formula predictor whose noise robustness is evaluated; run via
    running_scripts/experiments_test_noised.sh with instrument-appropriate TCN and
    rescore weights
  repo: https://github.com/JosieHong/FIDDLE
- name: BUDDY
  role: Bottom-up baseline (msbuddy) run on the identical noised spectra for top-5
    accuracy comparison
  repo: https://github.com/Philipbear/msbuddy
- name: SIRIUS
  role: Top-down baseline (SIRIUS 6) run on the identical noised spectra for top-5
    accuracy comparison
  repo: https://github.com/boecker-lab/sirius
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:3dadd59e3dcbba475a7bef0a7df5c6211c7d5532d0b8aef47602a5904bcb13c0
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# msms-spectral-noise-perturbation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Perturbs MS/MS spectra with additive Gaussian noise (mean 0, standard deviation 0.1 intensity units) at increasing levels to benchmark how robust molecular formula identification tools (FIDDLE, BUDDY, SIRIUS) are to spectral noise across Q-TOF and Orbitrap instruments.

## When to use

Use when evaluating robustness of formula-identification methods on MS/MS spectra (MGF format) by systematically injecting intensity noise and comparing top-k accuracy degradation across instruments.

## When NOT to use

- The input is unlabeled spectra with no known ground-truth formulas — accuracy against reference formulas cannot be computed
- The goal is to augment training data rather than evaluate robustness — note the paper found noise augmentation helped Q-TOF but was not applied for Orbitrap training because it lowered similarity
- The spectra are chimeric/co-eluting — that robustness axis (spectral mixing) is a separate evaluation, not noise perturbation

## Inputs

- 1000 randomly sampled test MS/MS spectra in MGF format (title, precursor_mz, precursor_type, collision_energy fields required)
- Ground-truth molecular formulas for the sampled test spectra
- FIDDLE pre-trained TCN and rescore checkpoint weights per instrument (fiddle_tcn_qtof.pt, fiddle_rescore_qtof.pt, fiddle_tcn_orbitrap.pt, fiddle_rescore_orbitrap.pt) with corresponding config yml files

## Outputs

- Noised MGF spectra files at each of five noise levels
- Top-5 formula accuracy per tool per noise level per instrument type
- noised_accuracy_results.csv aggregating the robustness comparison

## How to apply

Randomly sample 1000 test MS/MS spectra in MGF format (with title, precursor_mz, precursor_type, collision_energy fields) from a benchmark test set whose molecule identities are known. Perturb each spectrum at five increasing intensity-noise levels, sampling noise from a Gaussian distribution with mean 0 and standard deviation scaled per level, mirroring the paper's Q-TOF augmentation convention (mean 0, sd 0.1); no augmentation/noise convention applies to Orbitrap since training used none. Run each tool on the identical noised spectra with instrument-specific settings: FIDDLE with fiddle_tcn_qtof.pt/fiddle_rescore_qtof.pt weights and 10 ppm precursor tolerance for Q-TOF, or fiddle_tcn_orbitrap.pt/fiddle_rescore_orbitrap.pt weights and 5 ppm tolerance for Orbitrap; run BUDDY (msbuddy) and SIRIUS 6 on the same noised input. Compute top-5 formula accuracy per tool, per noise level, and per instrument, writing results to noised_accuracy_results.csv.

## Related tools

- **FIDDLE** (Deep learning formula predictor whose noise robustness is evaluated; run via running_scripts/experiments_test_noised.sh with instrument-appropriate TCN and rescore weights) — https://github.com/JosieHong/FIDDLE
- **BUDDY** (Bottom-up baseline (msbuddy) run on the identical noised spectra for top-5 accuracy comparison) — https://github.com/Philipbear/msbuddy
- **SIRIUS** (Top-down baseline (SIRIUS 6) run on the identical noised spectra for top-5 accuracy comparison) — https://github.com/boecker-lab/sirius

## Examples

```
bash running_scripts/experiments_test_noised.sh --config configs/fiddle_qtof.yml --tcn fiddle_tcn_qtof.pt --rescore fiddle_rescore_qtof.pt --ppm 10
```

## Evaluation signals

- Top-5 accuracies at the highest noise level reproduce Fig. 2g: Q-TOF FIDDLE 95.0% vs BUDDY 93.5% vs SIRIUS 52.5%
- Orbitrap highest-noise top-5 accuracies match: FIDDLE 93.3% vs BUDDY 75.4% vs SIRIUS 63.1%
- noised_accuracy_results.csv exists and contains one row per tool × noise level × instrument combination
- Noise is drawn from a zero-mean Gaussian with standard deviation 0.1 convention used in the paper's Q-TOF augmentation
- Precursor mass tolerances are 10 ppm for Q-TOF and 5 ppm for Orbitrap runs

## Limitations

- No data augmentation (and by extension no noise-based convention) was applied for Orbitrap training, so noise response on Orbitrap reflects a model trained without intensity perturbation
- FIDDLE's experimental-condition effect is not completely eliminated; accuracy varies across conditions, especially those with less training data
- Noise perturbation tests intensity noise only — other degradation axes such as chimeric (co-eluting) spectral mixing degrade FIDDLE differently and are a separate evaluation

## Evidence

- [results] FIDDLE robustness result at highest noise: [evidence span withheld — anchor status: not_found]
