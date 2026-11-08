---
name: rmse-correlation-method-benchmarking
description: Use when multiple NMR spectral deconvolution/quantification methods must
  be compared on identical spectra and a manual-annotation reference exists, so reconstruction
  quality can be scored with RMSE and Pearson correlation per metabolite cluster.
when_to_use_negative:
- Input spectra are from biofluids other than plasma — the article states 'Future
  efforts are needed to develop the biofluid-specific analysis pipeline', so this
  plasma-tuned comparison does not transfer directly.
- The goal is candidate biomarker discovery from estimated relative concentrations
  rather than benchmarking reconstruction/quantification accuracy against alternative
  methods.
- No original spectra or manual cluster annotations are available, so RMSE and Pearson
  correlation between reconstructed and original spectra cannot be computed per cluster.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: NMRQNet
  role: Deep-learning (CNN + GRU) method under evaluation; also defines the preprocessing
    tutorial and the EDTA plasma notebook whose outputs (reconstructed spectra, estimated
    concentrations) are scored
  repo: github.com/LiuzLab/NMRQNet
- name: rDolphin
  role: 'Baseline competitor: targeted profiling with the blood ROI library at 0.02
    ppm tolerance run on all 40 spectra'
  repo: danielcanueto/rDolphin
- name: Batman
  role: 'Baseline competitor: Bayesian quantification by Markov chain Monte Carlo
    with a customized 31-metabolite library on the same spectra'
- name: EDTA plasma analysis (NMRQNet tutorial)
  role: Reference workflow notebook (NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb)
    used to run NMRQNet on the plasma spectra
  repo: github.com/LiuzLab/NMRQNet
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:cba9745b0d4a91381a3fce5978f68fe79fe0097ab24181c473eb7593d4afdcd7
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
  - EDTA plasma analysis (NMRQNet tutorial)
  - NMRQNet
  - rDolphin
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# rmse-correlation-method-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark NMR metabolite quantification methods (NMRQNet, rDolphin targeted profiling, Batman MCMC Bayesian fitting) on the same 1H NMR plasma spectra by comparing reconstructed vs. original spectra with RMSE and Pearson correlation, as done for Fig. 4a of the NMRQNet paper on 40 MDS-family plasma samples.

## When to use

Use when multiple NMR spectral deconvolution/quantification methods must be compared on identical spectra and a manual-annotation reference exists, so reconstruction quality can be scored with RMSE and Pearson correlation per metabolite cluster.

## When NOT to use

- Input spectra are from biofluids other than plasma — the article states 'Future efforts are needed to develop the biofluid-specific analysis pipeline', so this plasma-tuned comparison does not transfer directly.
- The goal is candidate biomarker discovery from estimated relative concentrations rather than benchmarking reconstruction/quantification accuracy against alternative methods.
- No original spectra or manual cluster annotations are available, so RMSE and Pearson correlation between reconstructed and original spectra cannot be computed per cluster.

## Inputs

- csv file of 1H NMR plasma spectra: first column chemical shift values, one spectrum per sample column (40 MDS-family plasma samples)
- manually annotated metabolite cluster assignments (three clusters) per spectrum region
- rDolphin blood ROI profile library
- customized Batman library of 31 metabolites overlapping NMRQNet's candidate list

## Outputs

- fig4a_method_comparison.csv: per-method, per-sample RMSE and Pearson correlation within each annotated cluster
- fig4a_method_comparison.png: side-by-side RMSE and Pearson correlation comparison plot
- reconstructed spectra from NMRQNet, rDolphin, and Batman for residual inspection

## How to apply

Load the 40 Mecp2 duplication syndrome (MDS) family plasma spectra from a csv where the first column is chemical shift values and each remaining column is one 1H NMR spectrum (as in NMRQNet/data/tutorial_data/control_human_plasma.csv). Preprocess all spectra identically using the NMRQNet data_preprocessing tutorial so all methods see the same input. Run each competitor on the same preprocessed matrix: rDolphin targeted profiling with the blood ROI library from its extdata folder at 0.02 ppm tolerance; Batman with a customized library of the 31 metabolites overlapping NMRQNet's candidates, fitted by Markov chain Monte Carlo; and NMRQNet following the EDTA plasma tutorial notebook (NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb). For each method, reconstruct the whole spectrum and compute RMSE and Pearson correlation between reconstructed and original spectra within each of the three manually annotated clusters per sample. Assemble a comparison table (fig4a_method_comparison.csv) and plot it; the expected result, per the article, is that NMRQNet achieves the lowest RMSE and highest Pearson correlation for most samples, reflecting its low reconstruction residuals versus Bayesian (Batman) and linear-regression-based (rDolphin, ASICS) approaches.

## Related tools

- **NMRQNet** (Deep-learning (CNN + GRU) method under evaluation; also defines the preprocessing tutorial and the EDTA plasma notebook whose outputs (reconstructed spectra, estimated concentrations) are scored) — github.com/LiuzLab/NMRQNet
- **rDolphin** (Baseline competitor: targeted profiling with the blood ROI library at 0.02 ppm tolerance run on all 40 spectra) — danielcanueto/rDolphin
- **Batman** (Baseline competitor: Bayesian quantification by Markov chain Monte Carlo with a customized 31-metabolite library on the same spectra)
- **EDTA plasma analysis (NMRQNet tutorial)** (Reference workflow notebook (NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb) used to run NMRQNet on the plasma spectra) — github.com/LiuzLab/NMRQNet

## Examples

```
import pandas as pd; spectra = pd.read_csv('mds_plasma_40.csv'); # column 0 = chemical shift values, one 1H NMR spectrum per sample column
```

## Evaluation signals

- fig4a_method_comparison.csv contains, for all 40 samples, RMSE and Pearson correlation values for each of NMRQNet, rDolphin, and Batman within each of the three manually annotated clusters
- NMRQNet should show the lowest RMSE and highest Pearson correlation for most samples, consistent with the reported finding; systematic violations indicate mis-preprocessing or an unfair library setup for a competitor
- All three methods must run on the identical preprocessed spectral matrix (same csv layout: first column chemical shifts, one spectrum per column); divergent preprocessing invalidates the comparison
- Batman must be restricted to the 31 metabolites overlapping NMRQNet's candidate list, not a full library, to keep the comparison matched
- Residuals between reconstructed and original spectra should be visually low for NMRQNet, matching the claim that it 'reconstructs the whole NMR spectra with very low residuals'

## Limitations

- NMRQNet regularizes insufficiently on false-positive metabolite predictions because its loss focuses on eliminating residuals; a final manual visualization check is required to filter false positives before trusting quantification.
- The benchmark is specific to 1H NMR plasma spectra (EDTA, heparin, citrate anticoagulants, MDS-family samples); a biofluid-specific analysis pipeline for other biofluids was not yet developed.
- rDolphin and AQuA require manual library adjustment or dedicated sample preparations, which constrains how the baselines can be configured.

## Evidence

- [readme] rDolphin ROI libraries: [evidence span withheld — anchor status: external_doc]
