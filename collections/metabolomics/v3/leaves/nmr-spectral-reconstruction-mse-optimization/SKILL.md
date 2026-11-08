---
name: nmr-spectral-reconstruction-mse-optimization
description: Use when refining per-metabolite concentration and chemical-shift-position
  estimates from an initial NMR spectral fitting (e.g., CRNN output) so the reconstructed
  1H NMR spectrum minimizes MSE against the original plasma spectrum.
when_to_use_negative:
- When the input is not a fitted NMR spectrum with per-component estimates (e.g.,
  raw preprocessed spectra without an initial metabolite library fit) — the SHC stage
  refines an existing fit, it does not perform initial identification
- When metabolite concentrations/positions are already optimized and only statistical
  analysis of relative concentrations for biomarker extraction is needed
tools:
- name: NMRQNet
  role: Implements the stochastic hill-climbing local optimization stage that fine-tunes
    each metabolite's concentration and position to minimize reconstruction MSE after
    initial CRNN quantification
  repo: github.com/LiuzLab/NMRQNet
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:bcd3d5ca98e27ce6c01ddd47f6003162100200417072e5177b8f5b55d7692196
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# nmr-spectral-reconstruction-mse-optimization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stochastic hill-climbing (SHC) local optimization from the NMRQNet pipeline that fine-tunes each metabolite's concentration and position after initial deep-learning fitting by accepting only perturbations that decrease the mean square error (MSE) between reconstructed and original 1H NMR spectra.

## When to use

Use when refining per-metabolite concentration and chemical-shift-position estimates from an initial NMR spectral fitting (e.g., CRNN output) so the reconstructed 1H NMR spectrum minimizes MSE against the original plasma spectrum.

## When NOT to use

- When the input is not a fitted NMR spectrum with per-component estimates (e.g., raw preprocessed spectra without an initial metabolite library fit) — the SHC stage refines an existing fit, it does not perform initial identification
- When metabolite concentrations/positions are already optimized and only statistical analysis of relative concentrations for biomarker extraction is needed

## Inputs

- NMRQNet input CSV: chemical shift values in the first column, one 1H NMR spectrum per plasma sample in each subsequent column (e.g., NMRQNet/data/tutorial_data/control_human_plasma.csv)
- Initial per-component spectral estimates (concentrations and positions) from the preceding CRNN fitting stage

## Outputs

- shc_optimized_results.csv containing optimized per-component concentrations and positions plus the final reconstructed spectrum
- MSE trajectory across SHC iterations (as shown in Supplementary Fig. 5)

## How to apply

Load the NMRQNet input CSV (chemical shifts in the first column, one 1H NMR spectrum per subsequent column) together with the initial per-component spectral estimates from the preceding fitting stage, and initialize the SHC optimizer with the current MSE between the reconstructed and original spectra as the loss. At each iteration, randomly propose a concentration scaling factor in the range 0.2-10x and a position shift in the range -50 to 50 steps for each metabolite component. Accept a proposed perturbation only if the recomputed MSE decreases; otherwise revert to the previous parameter set, repeating proposals until convergence within approximately 500 iterations while tracking the MSE trajectory across iterations. Write the optimized per-component concentrations and positions together with the final reconstructed spectrum to shc_optimized_results.csv. The rationale is that greedy, per-metabolite acceptance guards against large joint perturbations degrading the global reconstruction while still correcting residual positional and concentration errors left by the initial fit.

## Related tools

- **NMRQNet** (Implements the stochastic hill-climbing local optimization stage that fine-tunes each metabolite's concentration and position to minimize reconstruction MSE after initial CRNN quantification) — github.com/LiuzLab/NMRQNet

## Evaluation signals

- Final MSE after SHC must be lower than the initial MSE between reconstructed and original spectra
- The optimization loop terminates within approximately 500 iterations, consistent with the convergence behavior shown in Supplementary Fig. 5
- Accepted perturbations respect the parameter ranges: concentration scaling factors within 0.2-10x and position shifts within -50 to 50 steps
- Output file shc_optimized_results.csv contains per-component concentrations, positions, and the final reconstructed spectrum
- Whole-spectrum residuals between reconstructed and original spectra should remain very low, consistent with NMRQNet's reported low-residual reconstruction

## Limitations

- NMRQNet regularized less on false-positive predictions because its loss focuses on eliminating residuals between original and reconstructed spectra, so SHC-driven MSE reduction can still leave false-positive metabolites requiring a final manual visualization check
- Convergence is bounded at approximately 500 iterations; non-convex local minima may prevent finding the global MSE optimum since only MSE-decreasing moves are accepted
- The pipeline is tuned for plasma 1H NMR data; a biofluid-specific analysis pipeline for other biofluid samples was not yet developed

## Evidence

- [other] In the stochastic hill-climbing local optimization, each metabolite is fine-tuned in concentration and position in small steps, and an update is accepted only when the mean square error decreases for that metabolite.: [evidence span withheld — anchor status: not_found]
- [supplementary] SHC convergence tracking in Supplementary Fig. 5: [evidence span withheld — anchor status: not_found]
