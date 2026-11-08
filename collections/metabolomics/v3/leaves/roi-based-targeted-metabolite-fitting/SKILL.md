---
name: roi-based-targeted-metabolite-fitting
description: Use when you have 1H NMR spectra of plasma (or other biofluid) samples
  plus a curated metabolite reference library, and need per-metabolite concentration
  estimates to benchmark against a new quantification method.
when_to_use_negative:
- When the goal is automatic quantification without a manually curated ROI library
  or prior metabolite selection — the article notes rDolphin requires manual library
  adjustment and Batman requires customized libraries built from specialized knowledge,
  whereas NMRQNet automates candidate selection.
- When you need to quantify metabolites absent from the predefined ROI/reference library
  — targeted fitting cannot recover signal outside the curated metabolite set.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3336
tools:
- name: rDolphin
  role: Performs ROI-based targeted profiling of the 40 MDS plasma spectra using the
    blood ROI library at 0.02 ppm tolerance, producing baseline concentration estimates
    compared against NMRQNet.
  repo: https://github.com/danielcanueto/rDolphin
- name: Batman
  role: Bayesian quantification of the same spectra via Markov chain Monte Carlo,
    configured with a customized library of the 31 metabolites overlapping NMRQNet's
    candidates.
- name: NMRQNet
  role: Reference/deep-learning baseline that the rDolphin and Batman targeted-fitting
    results are benchmarked against using RMSE and Pearson correlation (Fig. 4a);
    run following the EDTA plasma tutorial notebook.
  repo: https://github.com/LiuzLab/NMRQNet
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:5982b45173c3ae81b8cf5c19b66bb9dd5e446171a744b006a44246b5ffb18e53
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LiuzLab/NMRQNet
  repo_url: https://github.com/danielcanueto/rDolphin
  tools:
  - Batman
  - NMRQNet
  - rDolphin
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# roi-based-targeted-metabolite-fitting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Targeted profiling of 1H NMR plasma spectra by fitting predefined regions-of-interest (ROIs) from a curated metabolite library to each sample spectrum, as used for rDolphin and Batman baselines in the NMRQNet comparison on 40 MDS-family plasma samples.

## When to use

Use when you have 1H NMR spectra of plasma (or other biofluid) samples plus a curated metabolite reference library, and need per-metabolite concentration estimates to benchmark against a new quantification method.

## When NOT to use

- When the goal is automatic quantification without a manually curated ROI library or prior metabolite selection — the article notes rDolphin requires manual library adjustment and Batman requires customized libraries built from specialized knowledge, whereas NMRQNet automates candidate selection.
- When you need to quantify metabolites absent from the predefined ROI/reference library — targeted fitting cannot recover signal outside the curated metabolite set.

## Inputs

- csv file of 1H NMR spectra (first column: chemical shift values; one column per plasma sample), e.g. control_human_plasma.csv
- 40 plasma samples from Mecp2 duplication syndrome (MDS) families
- curated metabolite reference/ROI library (e.g. rDolphin blood ROI library; Batman library of 31 metabolites overlapping NMRQNet candidates)

## Outputs

- per-sample, per-metabolite concentration/fitting reports from rDolphin targeted profiling and Batman MCMC quantification
- reconstructed spectra used for residual computation
- RMSE and Pearson correlation values per method per manually annotated cluster (e.g. fig4a_method_comparison.csv)

## How to apply

Load the sample spectra from a csv file whose first column holds chemical shift values and whose remaining columns each contain one 1H NMR spectrum per plasma sample, as in NMRQNet/data/tutorial_data/control_human_plasma.csv. For rDolphin, run targeted profiling with a blood ROI library at 0.02 ppm tolerance on all 40 Mecp2 duplication syndrome (MDS) family plasma samples; ROI profiles for blood, urine and fecal extract are available in the rDolphin package's extdata folder. For Batman, customize the metabolite library first based on specialized knowledge — here the 31 metabolites overlapping with NMRQNet's candidates — and quantify via Markov chain Monte Carlo fitting. Compute RMSE and Pearson correlation between reconstructed and original spectra within manually annotated clusters (e.g. valine, leucine, isoleucine) to compare methods; in the NMRQNet study this comparison produced the Fig. 4a result that NMRQNet achieved the lowest RMSE and highest Pearson correlation across the 40 MDS plasma samples.

## Related tools

- **rDolphin** (Performs ROI-based targeted profiling of the 40 MDS plasma spectra using the blood ROI library at 0.02 ppm tolerance, producing baseline concentration estimates compared against NMRQNet.) — https://github.com/danielcanueto/rDolphin
- **Batman** (Bayesian quantification of the same spectra via Markov chain Monte Carlo, configured with a customized library of the 31 metabolites overlapping NMRQNet's candidates.)
- **NMRQNet** (Reference/deep-learning baseline that the rDolphin and Batman targeted-fitting results are benchmarked against using RMSE and Pearson correlation (Fig. 4a); run following the EDTA plasma tutorial notebook.) — https://github.com/LiuzLab/NMRQNet

## Examples

```
library(rDolphin); profiling <- targeted_profiling(roi_library_blood, spectra_40_MDS_plasma, tol_ppm = 0.02)
```

## Evaluation signals

- RMSE between reconstructed and original spectra computed within each manually annotated cluster (valine, leucine, isoleucine); lower RMSE indicates a better fit
- Pearson correlation between reconstructed and original spectra per cluster; in the NMRQNet study NMRQNet achieved the highest Pearson correlation across the 40 MDS plasma samples
- Consistency of the fitting across samples — the article notes NMRQNet 'has better consistency among samples' than prior computational methods
- Reproduce Fig. 4a: outputs saved as fig4a_method_comparison.csv and fig4a_method_comparison.png should show NMRQNet as best-performing (lowest RMSE, highest Pearson correlation) against rDolphin and Batman

## Limitations

- rDolphin requires manual library adjustment and Batman requires library customization from specialized knowledge, so neither is fully automatic
- Targeted fitting is restricted to metabolites present in the curated library (here 31 overlapping metabolites for Batman; blood ROI library for rDolphin)
- NMRQNet regularizes less on false-positive predictions, so final manual visualization checks are still needed to filter false positives — a caveat that also applies to judging targeted-fitting results
- rDolphin installation can fail on missing dependencies (e.g. impute, MassSpecWavelet) and requires bioconductor packages installed via biocLite

## Evidence

- [results] rDolphin targeted profiling with blood ROI library at 0.02 ppm tolerance: [evidence span withheld — anchor status: not_found]
- [results] Batman customized library of 31 overlapping metabolites, MCMC quantification: [evidence span withheld — anchor status: not_found]
- [results] NMRQNet outperformed rDolphin and Batman on 40 MDS plasma samples: [evidence span withheld — anchor status: not_found]
- [readme] ROI profiles for blood available in rDolphin package extdata: [evidence span withheld — anchor status: external_doc]
