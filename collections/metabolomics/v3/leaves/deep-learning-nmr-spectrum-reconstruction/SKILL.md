---
name: deep-learning-nmr-spectrum-reconstruction
description: Use when you have 1H NMR spectra of human/mouse plasma (e.g., EDTA plasma
  csv with chemical shifts in column 1 and one spectrum per sample column) and need
  automatic metabolite identification/quantification with spectrum-level reconstruction
  metrics.
when_to_use_negative:
- Input spectra are not plasma 1H NMR data (e.g. other biofluids) — the authors state
  a biofluid-specific analysis pipeline for other biofluid samples is not yet developed.
- You need fully automated removal of false-positive metabolite predictions — NMRQNet
  lacks automated pre-/post-filtering and requires a final manual visualization check.
- Samples require dedicated library adjustment or special sample preparation workflows
  incompatible with rDolphin/Chenomx-style targeted profiling assumptions.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
tools:
- name: NMRQNet
  role: Deep-learning (CNN + GRU) pipeline that preprocesses, reconstructs the original
    spectra, and identifies/quantifies dominant plasma metabolites
  repo: github.com/LiuzLab/NMRQNet
- name: rDolphin
  role: Baseline targeted-profiling quantification using the blood ROI library at
    0.02 ppm tolerance, benchmarked against NMRQNet
  repo: danielcanueto/rDolphin
- name: Batman
  role: Baseline Bayesian (MCMC) quantification with a customized metabolite library,
    benchmarked against NMRQNet
- name: Chenomx
  role: Manual expert mapping of library metabolite candidates used as the reference
    manual-annotation context
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:8a86b785f508c06326e4b60675b7564447cf029cd073066fb3e2bd2eae61d621
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
  - Chenomx
  - NMRQNet
  - rDolphin
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.95
n_claims: 20
---

# deep-learning-nmr-spectrum-reconstruction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct 1H NMR plasma spectra and quantify dominant metabolites using NMRQNet, a deep-learning (CNN + GRU) pipeline, and benchmark reconstruction quality (RMSE, Pearson correlation) against rDolphin targeted profiling and Batman MCMC Bayesian quantification.

## When to use

Use when you have 1H NMR spectra of human/mouse plasma (e.g., EDTA plasma csv with chemical shifts in column 1 and one spectrum per sample column) and need automatic metabolite identification/quantification with spectrum-level reconstruction metrics.

## When NOT to use

- Input spectra are not plasma 1H NMR data (e.g. other biofluids) — the authors state a biofluid-specific analysis pipeline for other biofluid samples is not yet developed.
- You need fully automated removal of false-positive metabolite predictions — NMRQNet lacks automated pre-/post-filtering and requires a final manual visualization check.
- Samples require dedicated library adjustment or special sample preparation workflows incompatible with rDolphin/Chenomx-style targeted profiling assumptions.

## Inputs

- csv file of 1H NMR spectra (first column = chemical shift values; one column per plasma sample), e.g. NMRQNet/data/tutorial_data/control_human_plasma.csv
- Plasma sample set (e.g. 40 Mecp2 duplication syndrome (MDS) family plasma samples; EDTA, heparin, or citrate anticoagulant)
- rDolphin blood ROI profile library (extdata folder)
- Batman customized metabolite library (metabolites overlapping NMRQNet candidates)

## Outputs

- Reconstructed 1H NMR spectra per sample from each method (NMRQNet, rDolphin, Batman)
- Estimated relative concentrations of dominant metabolite candidates per sample
- Evaluation scores: RMSE, Pearson correlation, and residuals per sample/method
- Benchmark tables/figures (e.g. fig4a_method_comparison.csv, fig4a_method_comparison.png)

## How to apply

Load spectra from a csv whose first column is chemical shift values and whose remaining columns are one 1H NMR spectrum per plasma sample (see NMRQNet/data/tutorial_data/control_human_plasma.csv as the format example). Preprocess the spectra following the NMRQNet data_preprocessing tutorial, then run the EDTA plasma notebook (NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb; use tutorial_for_other_plasma.ipynb for heparin/citrate plasma). NMRQNet fits dominant metabolite candidates against the IVDr (Bruker) list of 39 commonly measurable metabolites by combining CNN and GRU architectures with an optimization pipeline that minimizes residuals between original and reconstructed spectra. For benchmarking, run rDolphin targeted profiling with the blood ROI library (0.02 ppm tolerance) and Batman MCMC quantification with a customized library of metabolites overlapping NMRQNet's candidates, then compute RMSE and Pearson correlation between reconstructed and original spectra per method and per sample/cluster. Since NMRQNet regularizes weakly against false positives, always finish with a manual visualization check of predicted metabolites.

## Related tools

- **NMRQNet** (Deep-learning (CNN + GRU) pipeline that preprocesses, reconstructs the original spectra, and identifies/quantifies dominant plasma metabolites) — github.com/LiuzLab/NMRQNet
- **rDolphin** (Baseline targeted-profiling quantification using the blood ROI library at 0.02 ppm tolerance, benchmarked against NMRQNet) — danielcanueto/rDolphin
- **Batman** (Baseline Bayesian (MCMC) quantification with a customized metabolite library, benchmarked against NMRQNet)
- **Chenomx** (Manual expert mapping of library metabolite candidates used as the reference manual-annotation context)

## Examples

```
R: rDolphin_obj <- load_data('ROIprofiles_blood', 'NMRSpectra.csv'); profiling <- profiling_PI(rDolphin_obj, frequency_threshold = 0.02)   # NMRQNet: follow NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb on the preprocessed csv
```

## Evaluation signals

- NMRQNet should achieve the lowest RMSE and highest Pearson correlation between reconstructed and original spectra compared with rDolphin and Batman across the 40 MDS plasma samples (Fig. 4a comparison).
- Reconstruction residuals of the whole NMR spectra should be very low relative to the original spectra, and evaluation scores should show better consistency (lower spread) among samples than prior Bayesian or linear-regression methods.
- Output concentrations feed into statistical analysis (e.g. biomarker extraction for the MDS condition) without format errors.
- Manually flagged false-positive metabolite predictions are removed after the final visualization check before reporting concentrations.

## Limitations

- NMRQNet regularizes insufficiently on false-positive predictions because the loss focuses on eliminating residuals; a final manual visualization check is required to filter false positives.
- Further experiments are needed to confirm the proposed potential biomarkers.
- No biofluid-specific analysis pipeline yet for biofluids other than plasma.
- rDolphin and AQuA require manual library adjustment or dedicated sample preparations; Batman requires a customized library based on specialized knowledge.

## Evidence
