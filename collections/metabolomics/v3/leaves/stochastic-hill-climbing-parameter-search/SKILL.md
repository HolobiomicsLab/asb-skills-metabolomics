---
name: stochastic-hill-climbing-parameter-search
description: Use when per-metabolite spectral fits from an initial CRNN/deep-learning
  quantification still leave elevated residuals between reconstructed and original
  NMR spectra, and small per-component concentration/position corrections should be
  searched greedily.
when_to_use_negative:
- When the metabolite library or spectral fit requires manual adjustment (e.g. tools
  like rDolphin or AQuA are used instead) rather than automated local refinement
- When the input is not a chemical-shift-by-sample NMR spectrum CSV and no prior per-component
  estimates exist to initialize the search
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: NMRQNet
  role: Pipeline whose stochastic hill-climbing stage performs the per-metabolite
    concentration/position optimization; output written to shc_optimized_results.csv
  repo: https://github.com/LiuzLab/NMRQNet
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:c4c2180439710eadba282d34475f371b4453a7d755fd438a870771503a17db66
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# stochastic-hill-climbing-parameter-search

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A stochastic hill-climbing (SHC) local optimization stage in NMRQNet that fine-tunes each metabolite's concentration and spectral position after initial CRNN quantification by greedily accepting only perturbations that lower the mean squared error (MSE) between the reconstructed and original 1H NMR spectra.

## When to use

Use when per-metabolite spectral fits from an initial CRNN/deep-learning quantification still leave elevated residuals between reconstructed and original NMR spectra, and small per-component concentration/position corrections should be searched greedily.

## When NOT to use

- When the metabolite library or spectral fit requires manual adjustment (e.g. tools like rDolphin or AQuA are used instead) rather than automated local refinement
- When the input is not a chemical-shift-by-sample NMR spectrum CSV and no prior per-component estimates exist to initialize the search

## Inputs

- NMRQNet input CSV (first column: chemical shift values; each subsequent column: one 1H NMR plasma spectrum), e.g. NMRQNet/data/tutorial_data/control_human_plasma.csv
- Initial per-component metabolite concentration and position estimates from the preceding CRNN fitting stage

## Outputs

- shc_optimized_results.csv containing optimized per-component concentrations and positions plus the final reconstructed spectrum
- MSE trajectory across iterations (convergence curve, cf. Supplementary Fig. 5)

## How to apply

Load the NMRQNet input CSV (chemical shifts in the first column, one 1H NMR spectrum per subsequent column) together with the initial per-component estimates from the preceding fitting stage, and initialize the optimizer with the current MSE between the reconstructed and original spectra. At each iteration, randomly propose a concentration scaling factor in the range 0.2-10x and a position shift in the range -50 to 50 steps for each metabolite component. Recompute the MSE and accept a proposed perturbation only if it decreases; otherwise revert to the previous parameter set. Iterate until convergence within approximately 500 iterations, tracking the MSE trajectory (as shown in Supplementary Fig. 5), then write optimized concentrations and positions plus the final reconstructed spectrum to shc_optimized_results.csv.

## Related tools

- **NMRQNet** (Pipeline whose stochastic hill-climbing stage performs the per-metabolite concentration/position optimization; output written to shc_optimized_results.csv) — https://github.com/LiuzLab/NMRQNet

## Evaluation signals

- Final MSE after SHC is lower than the initial MSE between reconstructed and original spectra
- The proposal loop terminates within approximately 500 iterations
- Accepted parameters stay within the specified bounds (concentration scaling 0.2-10x, position shift -50 to 50 steps)
- shc_optimized_results.csv is produced with per-component concentrations, positions, and the final reconstructed spectrum
- The reconstructed spectrum shows low residuals relative to the original spectrum, consistent with NMRQNet's reported reconstruction quality

## Limitations

- Convergence is not guaranteed before the ~500-iteration cap; proposals are random so trajectories vary run to run
- NMRQNet regularizes less on false-positive predictions, so a final manual visualization check is still required to filter metabolites with false-positive predictions

## Evidence

- [methods] In the stochastic hill-climbing local optimization, each metabolite is fine-tuned in concentration and position in small steps, and an update is accepted only when the mean square error decreases for that metabolite.: [evidence span withheld — anchor status: not_found]
- [methods] At each iteration, randomly propose a concentration scaling factor in the range 0.2-10x and a position shift in the range -50 to 50 steps for each metabolite component.: [evidence span withheld — anchor status: not_found]
