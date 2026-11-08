---
name: nmr-spectral-residual-minimization
description: Use when refining per-metabolite concentration and position estimates
  after an initial deep-learning (CRNN) NMR spectral fit, when reconstruction residuals
  must be locally minimized via MSE.
when_to_use_negative:
- No initial per-component estimates exist yet — the spectra have not passed the preceding
  CRNN quantification stage, so there are no concentration/position parameters to
  fine-tune.
- The goal is metabolite candidate identification or biomarker discovery from scratch
  rather than residual minimization of an existing fit.
- The dominant need is filtering false-positive metabolite predictions — the article
  notes NMRQNet 'requires final visualization check to filter metabolites with false-positive
  predictions', which SHC does not address.
tools:
- name: NMRQNet
  role: Provides the SHC local-optimization stage that fine-tunes each metabolite's
    concentration and position against the MSE loss after the initial CRNN quantification,
    writing results to shc_optimized_results.csv
  repo: https://github.com/LiuzLab/NMRQNet
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:1cd88626802c6e8bcbbf61fb5cd6da6afc726912cb5e4adb46ee5916f67ba012
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LiuzLab/NMRQNet
  repo_url: https://github.com/LiuzLab/NMRQNet
  tools:
  - NMRQNet
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# nmr-spectral-residual-minimization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A stochastic hill-climbing (SHC) local optimization stage from NMRQNet that fine-tunes each metabolite's concentration and position after initial CRNN quantification, accepting only updates that lower the mean square error (MSE) between reconstructed and original 1H NMR spectra.

## When to use

Use when refining per-metabolite concentration and position estimates after an initial deep-learning (CRNN) NMR spectral fit, when reconstruction residuals must be locally minimized via MSE.

## When NOT to use

- No initial per-component estimates exist yet — the spectra have not passed the preceding CRNN quantification stage, so there are no concentration/position parameters to fine-tune.
- The goal is metabolite candidate identification or biomarker discovery from scratch rather than residual minimization of an existing fit.
- The dominant need is filtering false-positive metabolite predictions — the article notes NMRQNet 'requires final visualization check to filter metabolites with false-positive predictions', which SHC does not address.

## Inputs

- NMRQNet input CSV: first column holds chemical shift values, each subsequent column is one 1H NMR plasma spectrum (e.g. NMRQNet/data/tutorial_data/control_human_plasma.csv)
- Initial per-component metabolite spectral estimates (concentration and position) from the preceding CRNN/fitting stage

## Outputs

- shc_optimized_results.csv containing optimized per-metabolite concentrations and positions together with the final reconstructed spectrum
- MSE trajectory across SHC iterations for convergence tracking (as visualized in Supplementary Fig. 5)

## How to apply

Load the NMRQNet input CSV (chemical shift values in the first column, one 1H NMR plasma spectrum per subsequent column, e.g. NMRQNet/data/tutorial_data/control_human_plasma.csv) together with the initial per-component spectral estimates from the preceding CRNN fitting stage. Initialize the SHC optimizer with the current MSE between the reconstructed and original spectra as the loss. At each iteration, randomly propose a concentration scaling factor in the range 0.2-10x and a position shift in the range -50 to 50 steps for each metabolite component, and accept a perturbation only if the recomputed MSE decreases; otherwise revert to the previous parameter set. Repeat until convergence within approximately 500 iterations, tracking the MSE trajectory (Supplementary Fig. 5). Write the optimized per-component concentrations and positions plus the final reconstructed spectrum to shc_optimized_results.csv.

## Related tools

- **NMRQNet** (Provides the SHC local-optimization stage that fine-tunes each metabolite's concentration and position against the MSE loss after the initial CRNN quantification, writing results to shc_optimized_results.csv) — https://github.com/LiuzLab/NMRQNet

## Examples

```
jupyter notebook NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb
```

## Evaluation signals

- The final MSE after SHC must be lower than the initial MSE computed from the CRNN estimates.
- The optimization loop must terminate within approximately 500 iterations, matching the convergence behavior shown in Supplementary Fig. 5.
- Every accepted update must correspond to a decrease in per-metabolite MSE; rejected proposals must be reverted to the previous parameter set.
- All proposed concentration scaling factors must fall within 0.2-10x and position shifts within -50 to +50 steps.
- shc_optimized_results.csv must contain optimized per-component concentrations, positions, and the final reconstructed spectrum.

## Limitations

- SHC accepts only MSE-decreasing updates, so its result depends on the quality of the initial CRNN estimates and it can stall in local minima of the reconstruction error.
- NMRQNet focuses on eliminating residuals between original and reconstructed spectra and 'regularized less on the false-positive predictions', so SHC may refine spurious metabolite components — a final manual visualization check is required.
- The ~500-iteration budget is approximate; convergence behavior should be verified against the Supplementary Fig. 5 trajectory rather than assumed.

## Evidence

- [supplementary] SHC acceptance criterion: [evidence span withheld — anchor status: not_found]
