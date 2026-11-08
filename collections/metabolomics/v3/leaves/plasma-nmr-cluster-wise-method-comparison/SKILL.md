---
name: plasma-nmr-cluster-wise-method-comparison
description: Use when comparing metabolite quantification accuracy of NMRQNet vs.
  rDolphin vs. Batman on shared 1H NMR plasma spectra, evaluated by RMSE and Pearson
  correlation within defined chemical-shift clusters.
when_to_use_negative:
- Input spectra are urine, fecal extract, or other biofluids rather than plasma (rDolphin
  ships distinct ROI profiles for urine, blood, and fecal extract, and NMRQNet has
  separate tutorials for EDTA vs. heparin/citrate plasma)
- The goal is biomarker discovery or relative-concentration statistics rather than
  head-to-head reconstruction-accuracy comparison of quantification methods
- Only one quantification method is of interest and no cross-method RMSE/Pearson comparison
  on shared spectra is intended
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0593
tools:
- name: NMRQNet
  role: Deep-learning pipeline run on preprocessed spectra following the EDTA plasma
    tutorial; its reconstruction residuals are one arm of the per-cluster RMSE/Pearson
    comparison
  repo: github.com/LiuzLab/NMRQNet
- name: rDolphin
  role: Targeted profiling method run with the blood ROI library at 0.02 ppm tolerance
    on all 40 spectra to provide a second arm of the comparison
  repo: github.com/danielcanueto/rDolphin
- name: Batman
  role: Bayesian MCMC quantification tool configured with a customized 31-metabolite
    library overlapping NMRQNet candidates, providing the third arm of the comparison
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:c3275031c8e881a3738dcb05ce376634c42d9489ada72d34e89860ffae56cfe2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LiuzLab/NMRQNet
  tools:
  - Batman
  - NMRQNet
  - rDolphin
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# plasma-nmr-cluster-wise-method-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks targeted/bayesian metabolite quantification pipelines (NMRQNet, rDolphin, Batman) against each other on the same set of 1H NMR plasma spectra by computing per-sample RMSE and Pearson correlation between reconstructed and original spectra within manually annotated chemical-shift clusters, and reproducing the Fig. 4a head-to-head comparison.

## When to use

Use when comparing metabolite quantification accuracy of NMRQNet vs. rDolphin vs. Batman on shared 1H NMR plasma spectra, evaluated by RMSE and Pearson correlation within defined chemical-shift clusters.

## When NOT to use

- Input spectra are urine, fecal extract, or other biofluids rather than plasma (rDolphin ships distinct ROI profiles for urine, blood, and fecal extract, and NMRQNet has separate tutorials for EDTA vs. heparin/citrate plasma)
- The goal is biomarker discovery or relative-concentration statistics rather than head-to-head reconstruction-accuracy comparison of quantification methods
- Only one quantification method is of interest and no cross-method RMSE/Pearson comparison on shared spectra is intended

## Inputs

- Single csv file of 1H NMR plasma spectra: first column chemical shift values, one column per plasma sample (e.g. 40 MDS-family EDTA plasma spectra)
- Manually annotated chemical-shift cluster definitions used to segment the spectra for per-cluster scoring
- rDolphin blood ROI library with 0.02 ppm tolerance
- Customized Batman library of 31 metabolites overlapping NMRQNet's candidate set

## Outputs

- fig4a_method_comparison.csv — per-cluster, per-sample RMSE and Pearson correlation for NMRQNet, rDolphin, and Batman
- fig4a_method_comparison.png — Fig. 4a-style plot comparing RMSE and Pearson correlation across methods
- Reconstructed spectra for each method to compute residuals against the original spectra

## How to apply

Load the shared plasma spectra as a csv whose first column holds chemical shift values and each subsequent column holds one 1H NMR spectrum per sample (e.g. 40 MDS-family plasma samples), then preprocess all spectra identically using the NMRQNet data_preprocessing tutorial so reconstruction errors are comparable. Run each method on the same preprocessed spectra: rDolphin targeted profiling with the blood ROI library at 0.02 ppm tolerance, Batman Bayesian quantification via MCMC with a customized library of the 31 metabolites overlapping NMRQNet's candidates, and NMRQNet via the EDTA plasma tutorial notebook. For every method compute RMSE and Pearson correlation between the reconstructed and original spectrum within each of the three manually annotated chemical-shift clusters, rather than over the full spectrum, so per-region fit quality is isolated. Compare the three methods per cluster and per sample, reproducing the Fig. 4a pattern where NMRQNet shows the lowest RMSE and highest Pearson correlation across most of the 40 MDS plasma samples; export the numeric table (fig4a_method_comparison.csv) and the comparison plot (fig4a_method_comparison.png) for downstream inspection.

## Related tools

- **NMRQNet** (Deep-learning pipeline run on preprocessed spectra following the EDTA plasma tutorial; its reconstruction residuals are one arm of the per-cluster RMSE/Pearson comparison) — github.com/LiuzLab/NMRQNet
- **rDolphin** (Targeted profiling method run with the blood ROI library at 0.02 ppm tolerance on all 40 spectra to provide a second arm of the comparison) — github.com/danielcanueto/rDolphin
- **Batman** (Bayesian MCMC quantification tool configured with a customized 31-metabolite library overlapping NMRQNet candidates, providing the third arm of the comparison)

## Examples

```
run NMRQNet per tutorial_for_EDTA_plasma.ipynb; run rDolphin with blood ROI library @ 0.02 ppm tolerance; run Batman with 31-metabolite MCMC library; compute per-cluster RMSE & Pearson correlation
```

## Evaluation signals

- Per-cluster RMSE and Pearson correlation are computed against the ORIGINAL spectrum, not against another method's reconstruction — verify residuals are formed from original vs. reconstructed spectra only
- All three methods must be run on the identical preprocessed csv spectra (same samples, same chemical-shift grid) so RMSE/Pearson are directly comparable
- Reproduce the Fig. 4a ordering: NMRQNet achieves lowest RMSE and highest Pearson correlation for most of the 40 MDS plasma samples
- Per-cluster scoring must be restricted to the three manually annotated clusters, not the full spectral width
- Output table contains one row per (method, sample, cluster) with both RMSE and Pearson correlation columns

## Limitations

- rDolphin requires manual library adjustment or dedicated sample preparations, which can confound automated cross-method comparisons
- Batman requires manual customization of its library (here limited to the 31 metabolites overlapping NMRQNet's candidates), making the comparison sensitive to library curation choices
- Per-cluster evaluation depends on manually annotated cluster boundaries; differences in annotation would change the RMSE/Pearson landscape

## Evidence

- [results] NMRQNet is the best-performing method among NMRQNet, rDolphin, and Batman across the 40 MDS plasma samples, achieving the lowest RMSE and the highest Pearson correlation for most samples.: [evidence span withheld — anchor status: not_found]
- [readme] rDolphin availability of ROI profiles for blood used in targeted profiling.: [evidence span withheld — anchor status: external_doc]
