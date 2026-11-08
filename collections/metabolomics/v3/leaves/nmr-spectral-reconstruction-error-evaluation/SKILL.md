---
name: nmr-spectral-reconstruction-error-evaluation
description: Use when you have raw 1H NMR spectra (csv with chemical shifts in column
  1, one column per sample) plus a method's reconstructed spectra and want a per-sample
  fidelity metric, especially to expose regional reconstruction failure.
when_to_use_negative:
- You only need metabolite concentrations or biomarker lists, not spectral fidelity
  — use quantification/statistical analysis outputs instead.
- Your data are not whole-spectrum 1H NMR mixtures with per-sample columns in a csv
  (e.g. already-summarized peak tables or LC-MS data).
- No reconstructed spectra exist yet for the method you want to evaluate — this skill
  compares reconstruction outputs, it does not generate them.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: NMRQNet
  role: Produces reconstructed whole NMR spectra to be scored with RMSE/Pearson/residual
    evaluation; run via the EDTA plasma tutorial
  repo: https://github.com/LiuzLab/NMRQNet
- name: ASICS
  role: Baseline linear-combination library-fitting method whose reconstructed spectra
    are benchmarked against NMRQNet using RMSE (with tuned max.shift parameter)
- name: EDTA plasma analysis (NMRQNet tutorial)
  role: Reference workflow for loading the EDTA plasma csv and generating NMRQNet
    reconstructions used in the comparison
  repo: https://github.com/LiuzLab/NMRQNet
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:a94c752120e135693f99ef0ba81c5290f2046e720d5170126b4c3c39c228eb79
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
  - ASICS
  - EDTA plasma analysis (NMRQNet tutorial)
  - NMRQNet
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# nmr-spectral-reconstruction-error-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how faithfully a metabolite-quantification method reconstructs the original 1H NMR spectrum by computing whole-spectrum and regional (e.g. 3–4 ppm) RMSE between the original and reconstructed spectra, as used to benchmark NMRQNet against ASICS on 40 MDS EDTA plasma samples.

## When to use

Use when you have raw 1H NMR spectra (csv with chemical shifts in column 1, one column per sample) plus a method's reconstructed spectra and want a per-sample fidelity metric, especially to expose regional reconstruction failure.

## When NOT to use

- You only need metabolite concentrations or biomarker lists, not spectral fidelity — use quantification/statistical analysis outputs instead.
- Your data are not whole-spectrum 1H NMR mixtures with per-sample columns in a csv (e.g. already-summarized peak tables or LC-MS data).
- No reconstructed spectra exist yet for the method you want to evaluate — this skill compares reconstruction outputs, it does not generate them.

## Inputs

- csv file of 1H NMR spectra: first column = chemical shift values, each subsequent column = one plasma sample's spectrum
- per-sample reconstructed spectra from each quantification method (e.g. ASICS and NMRQNet outputs)

## Outputs

- per-sample RMSE table comparing methods (sample, method A RMSE, method B RMSE, optional regional ppm-window RMSE), e.g. rmse_comparison_edta.csv
- regional (3–4 ppm) RMSE values localizing reconstruction failure regions
- supplementary Pearson correlation and residual summaries per sample

## How to apply

Load the 1H NMR csv where the first column holds chemical shift values and each subsequent column is one plasma sample's spectrum (e.g. the 40 EDTA plasma samples from the NMRQNet tutorial_for_EDTA_plasma.ipynb). Run each quantification method on the identical csv — ASICS with the tuned max.shift parameter and NMRQNet via its EDTA plasma tutorial — to obtain per-sample reconstructed spectra. For every sample compute whole-spectrum RMSE between the original and each method's reconstruction; additionally compute RMSE restricted to the 3–4 ppm window, the region where ASICS's reconstruction fails in the paper's Fig. 5a. Assemble a per-sample comparison table and inspect whether one method's RMSE values are consistently lower, with largest errors concentrated in the problematic ppm region. Complement RMSE with Pearson correlation and residual inspection, as NMRQNet's evaluation used RMSE, Pearson correlation and residuals to show improved consistency among samples.

## Related tools

- **NMRQNet** (Produces reconstructed whole NMR spectra to be scored with RMSE/Pearson/residual evaluation; run via the EDTA plasma tutorial) — https://github.com/LiuzLab/NMRQNet
- **ASICS** (Baseline linear-combination library-fitting method whose reconstructed spectra are benchmarked against NMRQNet using RMSE (with tuned max.shift parameter))
- **EDTA plasma analysis (NMRQNet tutorial)** (Reference workflow for loading the EDTA plasma csv and generating NMRQNet reconstructions used in the comparison) — https://github.com/LiuzLab/NMRQNet

## Examples

```
Follow NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb to load the EDTA plasma csv and obtain per-sample reconstructions, then compute RMSE against the original spectra.
```

## Evaluation signals

- The comparison CSV exists with exactly one row per sample (40 rows for the MDS EDTA plasma set).
- NMRQNet's per-sample whole-spectrum RMSE values are consistently lower than ASICS's, mirroring Fig. 5a qualitatively.
- ASICS's largest errors localize to the 3–4 ppm region when regional RMSE is computed.
- RMSE results are corroborated by Pearson correlation and residual inspection, which improve and show better consistency among samples for the winning method.

## Limitations

- NMRQNet regularizes less on false-positive metabolite predictions, so low spectral RMSE does not guarantee all assigned metabolites are real — a final manual visualization check is required.
- Regional evaluation depends on choosing an appropriate ppm window (e.g. 3–4 ppm for EDTA plasma artifact region); other artifacts may need different windows.
- The evaluation was demonstrated on 1H NMR plasma spectra across anticoagulants (EDTA, heparin, citrate) and species from mice to humans; other biofluids lack a validated pipeline.

## Evidence

- [results] NMRQNet consistently outperformed ASICS, performing best in both regional explorations and whole-spectrum reconstruction across the tested plasma samples.: [evidence span withheld — anchor status: not_found]
