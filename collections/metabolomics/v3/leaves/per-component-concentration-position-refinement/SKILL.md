---
name: per-component-concentration-position-refinement
description: Use when an initial CRNN-based spectral decomposition of 1H NMR plasma
  spectra leaves residual error and individual metabolite components need concentration
  scaling and peak-position adjustment to lower the per-metabolite reconstruction
  MSE.
when_to_use_negative:
- When a metabolite's spectrum requires full re-identification rather than small perturbations
  — SHC only fine-tunes concentration (0.2–10x) and position (−50 to 50 steps) around
  an existing fit
- When the false-positive metabolite predictions are the main problem — NMRQNet regularizes
  less on false-positive predictions and a final visualization check is required,
  so SHC's MSE objective alone will not filter them
- When the target data is not 1H NMR spectra of plasma (or other biofluids) — the
  concentration scaling and position-shift ranges were tuned for NMR spectral fitting
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NMRQNet
  role: Provides the stochastic hill-climbing local optimization stage that fine-tunes
    each metabolite's concentration and position after initial quantification, minimizing
    the mean squared error between reconstructed and original spectra
  repo: github.com/LiuzLab/NMRQNet
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:151227464e981a2c25eeae730579e4e6227a6997674dd404ce9dff31a8ab3eb0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LiuzLab/NMRQNet
  tools:
  - NMRQNet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# per-component-concentration-position-refinement

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A stochastic hill-climbing (SHC) local optimization stage from NMRQNet that fine-tunes each metabolite's concentration and position in small steps, accepting a perturbation only when the mean square error (MSE) between the reconstructed and original 1H NMR spectrum decreases.

## When to use

Use when an initial CRNN-based spectral decomposition of 1H NMR plasma spectra leaves residual error and individual metabolite components need concentration scaling and peak-position adjustment to lower the per-metabolite reconstruction MSE.

## When NOT to use

- When a metabolite's spectrum requires full re-identification rather than small perturbations — SHC only fine-tunes concentration (0.2–10x) and position (−50 to 50 steps) around an existing fit
- When the false-positive metabolite predictions are the main problem — NMRQNet regularizes less on false-positive predictions and a final visualization check is required, so SHC's MSE objective alone will not filter them
- When the target data is not 1H NMR spectra of plasma (or other biofluids) — the concentration scaling and position-shift ranges were tuned for NMR spectral fitting

## Inputs

- NMRQNet input CSV with chemical shift values in the first column and one 1H NMR spectrum per plasma sample in subsequent columns (e.g. NMRQNet/data/tutorial_data/control_human_plasma.csv)
- Initial per-component spectral estimates (metabolite concentrations and positions) from the preceding NMRQNet quantification stage

## Outputs

- shc_optimized_results.csv containing optimized per-component metabolite concentrations and positions
- Final reconstructed spectrum per plasma sample
- MSE trajectory across SHC iterations (as tracked in Supplementary Fig. 5)

## How to apply

Load the NMRQNet input CSV (chemical shifts in the first column, one 1H NMR spectrum per subsequent column, e.g. control_human_plasma.csv) together with the initial per-component spectral estimates from the preceding fitting stage. Initialize the SHC optimizer with the current mean squared error between the reconstructed and original spectra as the loss to minimize. At each iteration, randomly propose a concentration scaling factor in the range 0.2–10x and a position shift in the range −50 to 50 steps for each metabolite component. Accept a proposal only if the recomputed MSE decreases; otherwise revert to the previous parameter set. Iterate for up to ~500 iterations until convergence, then write the optimized per-component concentrations and positions plus the final reconstructed spectrum to shc_optimized_results.csv, validating that the final MSE is lower than the initial MSE.

## Related tools

- **NMRQNet** (Provides the stochastic hill-climbing local optimization stage that fine-tunes each metabolite's concentration and position after initial quantification, minimizing the mean squared error between reconstructed and original spectra) — github.com/LiuzLab/NMRQNet

## Examples

```
python NMRQNet/run_shc.py --input control_human_plasma.csv --out shc_optimized_results.csv --max-iter 500
```

## Evaluation signals

- Final MSE after SHC must be lower than the initial MSE computed before any perturbation
- SHC loop must terminate within approximately 500 iterations, consistent with the convergence behavior in Supplementary Fig. 5
- Accepted proposals must strictly decrease the per-metabolite MSE; any update that increases MSE must be reverted
- Concentration scaling factors must remain within the 0.2–10x range and position shifts within −50 to 50 steps
- Output file shc_optimized_results.csv must contain optimized concentrations and positions for every metabolite component plus the final reconstructed spectrum

## Limitations

- The MSE objective focuses on eliminating residuals between original and reconstructed spectra, so NMRQNet regularizes less on false-positive predictions and a final visualization check is needed
- Position shifts are limited to −50 to 50 steps and concentration scaling to 0.2–10x; larger misfits require re-running the initial quantification
- Biofluid-specific analysis pipelines (e.g. beyond plasma) are not yet developed, and further experiments are needed to confirm potential biomarkers

## Evidence

- [methods] SHC update criterion: [evidence span withheld — anchor status: not_found]
