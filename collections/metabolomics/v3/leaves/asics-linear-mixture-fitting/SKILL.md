---
name: asics-linear-mixture-fitting
description: Use when you have one or more 1H NMR plasma spectra (chemical shifts
  plus one intensity column per sample) and need a baseline automated deconvolution
  into reference metabolite spectra with fitted concentrations, e.g. for RMSE-based
  reconstruction benchmarking.
when_to_use_negative:
- The goal is the best possible reconstruction accuracy — the paper shows NMRQNet
  consistently outperformed ASICS on whole-spectrum RMSE and in the 3-4 ppm region
  of EDTA plasma, so ASICS is a baseline rather than the method of choice.
- The input is not 1H NMR spectra of plasma in the csv shift-column-plus-sample-columns
  layout expected by the NMRQNet-style pipeline.
- You need quantification methods that tolerate no manual setup but also need strong
  false-positive control — the paper's broader evaluation covers alternatives (Batman,
  rDolphin, AQuA, Chenomx) with different trade-offs.
tools:
- name: ASICS
  role: 'Linear-regression deconvolution engine: fits each mixture spectrum as a linear
    combination of library reference spectra and returns reconstructed spectra and
    fitted concentrations; run with the tuned max.shift setting for the NMRQNet comparison'
- name: NMRQNet
  role: 'Comparator and data-provider: supplies the EDTA plasma csv input format and
    the tutorial workflow whose reconstructed spectra are used as the RMSE benchmark
    against ASICS'
  repo: github.com/LiuzLab/NMRQNet
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:20fc450716530a57514ee519856d66ca3094e7d19d637091e9a4e8f406895620
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
  - NMRQNet
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# asics-linear-mixture-fitting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply ASICS, a linear-regression-based deconvolution method that models an NMR mixture spectrum as a linear combination of library reference spectra, to obtain per-sample reconstructed spectra and fitted metabolite concentrations for 1H plasma NMR data.

## When to use

Use when you have one or more 1H NMR plasma spectra (chemical shifts plus one intensity column per sample) and need a baseline automated deconvolution into reference metabolite spectra with fitted concentrations, e.g. for RMSE-based reconstruction benchmarking.

## When NOT to use

- The goal is the best possible reconstruction accuracy — the paper shows NMRQNet consistently outperformed ASICS on whole-spectrum RMSE and in the 3-4 ppm region of EDTA plasma, so ASICS is a baseline rather than the method of choice.
- The input is not 1H NMR spectra of plasma in the csv shift-column-plus-sample-columns layout expected by the NMRQNet-style pipeline.
- You need quantification methods that tolerate no manual setup but also need strong false-positive control — the paper's broader evaluation covers alternatives (Batman, rDolphin, AQuA, Chenomx) with different trade-offs.

## Inputs

- 1H NMR spectra csv with chemical shift values in the first column and one column per plasma sample (e.g. the 40 EDTA plasma samples from MDS families)
- ASICS reference metabolite spectral library
- Tuned ASICS run parameters (customized max.shift as used in the NMRQNet comparison)

## Outputs

- Per-sample reconstructed (fitted) NMR spectra
- Per-sample fitted metabolite concentrations
- Per-sample RMSE comparison table (e.g. rmse_comparison_edta.csv with whole-spectrum and 3-4 ppm RMSE per sample)

## How to apply

Load the spectra as a csv whose first column holds chemical shift values and whose remaining columns each hold one plasma sample's 1H NMR spectrum (the format used by the NMRQNet EDTA plasma tutorial data). Run ASICS with the customized settings used in the NMRQNet comparison — notably the tuned max.shift parameter — so that each mixture spectrum is fitted as a linear combination of library reference spectra. Collect the reconstructed (fitted) spectrum and the fitted concentrations for every sample. When used as a benchmark against NMRQNet, compute whole-spectrum RMSE between the original and reconstructed spectra per sample, and additionally restrict to the 3-4 ppm window, since ASICS's reconstruction error concentrates there for EDTA plasma. Rationale: the NMRQNet paper uses ASICS precisely as the representative linear-regression-based method, and its whole-spectrum and regional RMSE comparisons (Fig. 5a pattern) show the linear-combination model's failure region on 40 MDS-family EDTA plasma samples.

## Related tools

- **ASICS** (Linear-regression deconvolution engine: fits each mixture spectrum as a linear combination of library reference spectra and returns reconstructed spectra and fitted concentrations; run with the tuned max.shift setting for the NMRQNet comparison)
- **NMRQNet** (Comparator and data-provider: supplies the EDTA plasma csv input format and the tutorial workflow whose reconstructed spectra are used as the RMSE benchmark against ASICS) — github.com/LiuzLab/NMRQNet

## Evaluation signals

- The output comparison table (e.g. rmse_comparison_edta.csv) contains exactly 40 rows, one per EDTA plasma sample.
- Per-sample whole-spectrum RMSE of the linear-combination reconstruction can be compared to the reference method; in the source study the Fig. 5a pattern held qualitatively, with the comparator's RMSE consistently lower.
- Regional check: RMSE computed on the 3-4 ppm window is markedly larger than elsewhere, reproducing the identified ASICS reconstruction-failure region on EDTA plasma.
- ASICS must be run with the customized settings from the NMRQNet comparison, including the tuned max.shift parameter, otherwise the benchmark is not comparable.
- Each fitted reconstruction plus concentration table should be traceable to one input sample column in the csv.

## Limitations

- ASICS's linear-combination model showed a reconstruction failure region in the 3-4 ppm window for EDTA plasma samples, where its RMSE concentrated.
- As a linear-regression-based method, ASICS was markedly outperformed by NMRQNet in whole-spectrum reconstruction (RMSE) and quantification consistency across samples.
- Benchmark conclusions are tied to the tested plasma type/anticoagulant (EDTA plasma from MDS families); the paper also evaluates heparin and citrate plasma separately.

## Evidence

- [intro] ASICS models mixtures as linear combinations: "ASICS is a method introduced more recently by ref. 17,18, which models the mixture spectrum as a linear combination of the library reference spectra."
- [readme] csv input format: "The input file provided to NMRQNet is a csv file. - The first column is chemical shift values - For the rest of columns, each column is the 1H NMR spectrum for one plasma sample"
- [discussion] NMRQNet outperforms prior linear-regression methods: "NMRQNet accurately reconstructs the whole NMR spectra with very low residuals compared to the original spectra, far more efficient than the traditional approach, and markedly better than previous"
- [methods] EDTA plasma tutorial entry point: "Tutorial for EDTA plasma analysis: at NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb"
