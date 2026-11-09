---
name: ppm-region-restricted-error-analysis
description: Use when comparing two NMR spectral reconstruction/quantification methods
  and whole-spectrum RMSE alone does not explain where errors concentrate — e.g. when
  ASICS-like linear-combination fits fail in specific ppm windows of plasma 1H NMR
  data.
when_to_use_negative:
- The goal is quantification accuracy of metabolite concentrations rather than spectral
  reconstruction quality — RMSE on spectra does not directly validate fitted concentrations
- The spectra have different chemical-shift axes or preprocessing across methods,
  so window subsetting (3–4 ppm) would not be comparable between the two reconstructions
- The comparison target is metabolite biomarker discovery via estimated relative concentrations,
  not whole-spectrum fitting fidelity
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NMRQNet
  role: Produces the reconstructed spectra whose residual/RMSE profile is being localized
    per ppm window, via the EDTA plasma tutorial workflow
  repo: https://github.com/LiuzLab/NMRQNet
- name: ASICS
  role: Baseline linear-combination fitting method whose reconstruction is evaluated;
    run with customized settings including the tuned max.shift parameter before computing
    window-restricted RMSE
- name: EDTA plasma analysis (NMRQNet tutorial)
  role: Reference workflow (NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb) for running
    NMRQNet on the EDTA plasma csv so the same samples feed both methods' RMSE computation
  repo: https://github.com/LiuzLab/NMRQNet
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:70eadd4921a34c78fce92873ea5b7b0fb4ef0250b65f784b3eef423f348a57d3
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# ppm-region-restricted-error-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A reconstruction-error localization technique for 1H NMR spectra: in addition to whole-spectrum RMSE, RMSE is recomputed within a narrow chemical-shift window (3–4 ppm) to pinpoint the region where a fitting method fails, as done when comparing NMRQNet against ASICS on EDTA plasma spectra.

## When to use

Use when comparing two NMR spectral reconstruction/quantification methods and whole-spectrum RMSE alone does not explain where errors concentrate — e.g. when ASICS-like linear-combination fits fail in specific ppm windows of plasma 1H NMR data.

## When NOT to use

- The goal is quantification accuracy of metabolite concentrations rather than spectral reconstruction quality — RMSE on spectra does not directly validate fitted concentrations
- The spectra have different chemical-shift axes or preprocessing across methods, so window subsetting (3–4 ppm) would not be comparable between the two reconstructions
- The comparison target is metabolite biomarker discovery via estimated relative concentrations, not whole-spectrum fitting fidelity

## Inputs

- Original 1H NMR spectra csv (chemical shifts in first column, one column per plasma sample), e.g. the 40 EDTA plasma MDS-family samples
- Per-sample reconstructed spectra from each method being compared (e.g. ASICS reconstructions with tuned max.shift, NMRQNet reconstructions from tutorial_for_EDTA_plasma.ipynb)

## Outputs

- Per-sample comparison table with columns: sample, ASICS whole-spectrum RMSE, NMRQNet whole-spectrum RMSE, and 3–4 ppm window RMSE for each method (e.g. rmse_comparison_edta.csv, 40 rows)
- Identification of the ppm window where the weaker method's reconstruction error concentrates

## How to apply

After running both reconstruction methods (e.g. ASICS with a tuned max.shift parameter and NMRQNet via the EDTA plasma tutorial) on the same spectra csv — first column chemical shifts, one column per sample — compute per-sample whole-spectrum RMSE between the original and each reconstructed spectrum. Then subset the spectra to the 3–4 ppm chemical-shift window and recompute RMSE there for every sample. Assemble a per-sample table (sample, ASICS RMSE, NMRQNet RMSE, plus the 3–4 ppm values) and save it, e.g. as rmse_comparison_edta.csv. The rationale is that a method may score acceptably overall yet fail badly in a diagnostic region: in the NMRQNet paper, ASICS's errors concentrated in 3–4 ppm while NMRQNet's per-sample whole-spectrum RMSE stayed consistently lower, so region-restricted RMSE separates 'globally good' from 'locally broken' fits.

## Related tools

- **NMRQNet** (Produces the reconstructed spectra whose residual/RMSE profile is being localized per ppm window, via the EDTA plasma tutorial workflow) — https://github.com/LiuzLab/NMRQNet
- **ASICS** (Baseline linear-combination fitting method whose reconstruction is evaluated; run with customized settings including the tuned max.shift parameter before computing window-restricted RMSE)
- **EDTA plasma analysis (NMRQNet tutorial)** (Reference workflow (NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb) for running NMRQNet on the EDTA plasma csv so the same samples feed both methods' RMSE computation) — https://github.com/LiuzLab/NMRQNet

## Examples

```
python compute_rmse.py --input edta_plasma.csv --methods ASICS,NMRQNet --window 3-4 --out rmse_comparison_edta.csv
```

## Evaluation signals

- The output CSV (e.g. rmse_comparison_edta.csv) exists with exactly 40 rows, one per EDTA plasma sample
- NMRQNet's per-sample whole-spectrum RMSE values are consistently lower than ASICS's, matching the qualitative pattern of Fig. 5a
- ASICS's largest errors are concentrated in the 3–4 ppm region: 3–4 ppm RMSE for ASICS markedly exceeds its whole-spectrum RMSE pattern
- Same csv, same chemical-shift axis and same samples are used for both methods before window subsetting, so RMSE values are directly comparable
- Both whole-spectrum and window-restricted RMSE are finite, non-negative, and computed per sample (no NaNs from misaligned shift columns)

## Limitations

- NMRQNet regularizes less on false-positive metabolite predictions, requiring a final manual visualization check to filter them — low residual RMSE alone does not guarantee correct metabolite assignments
- The 3–4 ppm window was chosen for this EDTA plasma comparison; other failure regions may exist and the specific window is dataset/method-dependent
- Findings were established on 40 MDS-family EDTA plasma samples; generalization to other anticoagulants (heparin, citrate) or biofluids is not demonstrated here

## Evidence

- [results] Whole-spectrum and window-restricted RMSE both computed per sample: [evidence span withheld — anchor status: not_found]
