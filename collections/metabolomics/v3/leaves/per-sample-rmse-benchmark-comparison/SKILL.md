---
name: per-sample-rmse-benchmark-comparison
description: Use when a new metabolite identification/quantification method's output
  reconstructions must be validated against an established tool on identical 1H NMR
  spectra, to demonstrate lower per-sample reconstruction error.
when_to_use_negative:
- When only aggregate or mean RMSE across samples is needed and per-sample consistency
  is not being assessed — the whole point of this skill is per-sample comparison tables
- When the goal is quantification accuracy (concentration estimates) rather than spectral
  reconstruction error — the article's evaluation also covers Pearson correlation
  and residual checks, and this skill targets RMSE on reconstructed spectra
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: NMRQNet
  role: 'New method under benchmark: produces per-sample reconstructed spectra whose
    RMSE is compared against the baseline'
  repo: github.com/LiuzLab/NMRQNet
- name: ASICS
  role: 'Classical baseline: models the mixture spectrum as a linear combination of
    library reference spectra; its reconstructions supply the per-sample RMSE values
    the new method must beat'
- name: EDTA plasma analysis (NMRQNet tutorial)
  role: Reference workflow for running NMRQNet on the plasma spectra csv so that the
    comparison settings match the article's protocol
  repo: NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:d82749eb946cd1a8ac18ff7d82b9834a954efd2f9813d796437ac2c1682e26ca
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LiuzLab/NMRQNet
  tools:
  - ASICS
  - EDTA plasma analysis (NMRQNet tutorial)
  - NMRQNet
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# per-sample-rmse-benchmark-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks a new NMR spectral reconstruction/quantification pipeline (e.g. NMRQNet) against a classical baseline (e.g. ASICS) by computing per-sample whole-spectrum RMSE on the same 1H NMR spectra csv, plus regional RMSE over windows such as 3-4 ppm where the baseline is known to fail.

## When to use

Use when a new metabolite identification/quantification method's output reconstructions must be validated against an established tool on identical 1H NMR spectra, to demonstrate lower per-sample reconstruction error.

## When NOT to use

- When only aggregate or mean RMSE across samples is needed and per-sample consistency is not being assessed — the whole point of this skill is per-sample comparison tables
- When the goal is quantification accuracy (concentration estimates) rather than spectral reconstruction error — the article's evaluation also covers Pearson correlation and residual checks, and this skill targets RMSE on reconstructed spectra

## Inputs

- csv file of 1H NMR spectra: first column chemical shift values, one column per plasma sample (NMRQNet input format)
- Baseline method (ASICS) fitted/reconstructed spectra with customized comparison settings (e.g. tuned max.shift)
- New method (NMRQNet) per-sample reconstructed spectra from the matching tutorial workflow (e.g. EDTA plasma notebook)

## Outputs

- Per-sample comparison table (e.g. rmse_comparison_edta.csv) with columns: sample, baseline whole-spectrum RMSE, new-method whole-spectrum RMSE, regional (e.g. 3-4 ppm) RMSE values
- Qualitative confirmation that the new method's per-sample RMSE is consistently lower than the baseline's, with baseline errors concentrated in the regional window

## How to apply

Load the spectra as a csv whose first column holds chemical shift values and whose remaining columns each hold one sample's 1H NMR spectrum (the NMRQNet input format, e.g. control_human_plasma.csv). Run the baseline (ASICS) on the same spectra with any customized settings used in the comparison (e.g. a tuned max.shift parameter) to get per-sample reconstructed spectra, and run the new method (NMRQNet) via its corresponding tutorial workflow (e.g. the EDTA plasma notebook) on the same csv. For every sample compute whole-spectrum RMSE between the original and each method's reconstruction, then additionally compute RMSE restricted to diagnostic windows (such as 3-4 ppm, where ASICS's reconstruction failure concentrates) to localize where the improvement comes from. Assemble a per-sample table (sample, baseline RMSE, new-method RMSE, plus regional values) and verify the expected qualitative pattern before claiming superiority, since the paper's Fig. 5a-style result rests on consistent per-sample wins, not just an aggregate mean.

## Related tools

- **NMRQNet** (New method under benchmark: produces per-sample reconstructed spectra whose RMSE is compared against the baseline) — github.com/LiuzLab/NMRQNet
- **ASICS** (Classical baseline: models the mixture spectrum as a linear combination of library reference spectra; its reconstructions supply the per-sample RMSE values the new method must beat)
- **EDTA plasma analysis (NMRQNet tutorial)** (Reference workflow for running NMRQNet on the plasma spectra csv so that the comparison settings match the article's protocol) — NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb

## Examples

```
N/A — the per-sample RMSE computation is a downstream analysis on the csv outputs of NMRQNet and ASICS; no standalone CLI/script is provided in the article or README, so a grounded invocation cannot be given.
```

## Evaluation signals

- The per-sample comparison table has exactly one row per spectrum/sample (e.g. 40 rows for the 40 MDS-family EDTA plasma samples)
- The new method's whole-spectrum RMSE is lower than the baseline's for the consistent majority of samples, matching the Fig. 5a-style qualitative pattern
- Regional RMSE (e.g. 3-4 ppm) is markedly worse for the baseline than for the new method, localizing the baseline's failure region
- Both methods were run on the identical input csv (same chemical-shift column and same sample columns), not on independently preprocessed spectra

## Limitations

- NMRQNet regularizes insufficiently on false-positive metabolite predictions, so low RMSE does not guarantee the absence of false positives; a final visualization check is still required
- RMSE comparison reflects reconstruction quality, not quantification or biomarker validity — potential biomarkers from downstream concentration analysis require further experimental confirmation
- Comparisons are specific to the tested plasma matrices (e.g. EDTA-anticoagulated plasma); other biofluids or anticoagulants may not transfer without their own analysis pipeline

## Evidence
