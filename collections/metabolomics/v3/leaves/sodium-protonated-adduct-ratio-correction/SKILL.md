---
name: sodium-protonated-adduct-ratio-correction
description: Use when positive-mode qMSI lipid images contain [M+H]+ adducts isobarically
  overlapped by [M+Na]+ species of the same lipid class that cannot be mass-resolved
  (Δm/z 0.0025), especially in datasets with prevalent M+Na adducts.
when_to_use_negative:
- Datasets acquired with sufficient resolving power to mass-resolve the [M+H]+/[M+Na]+
  pair directly — the Δm/z of 0.0025 requires a minimum resolution of 600,000, so
  if the instrument achieves this the signals need no ratio-based deconvolution
- Lipid classes flagged as unsupported by the H/Na overlap correction — the images
  must be passed through unchanged rather than corrected
- Datasets with no prevalent [M+Na]+ adducts, where the optional Na/H overlap correction
  adds no value over raw or Type II-corrected images
tools:
- name: LipidQMap
  role: 'Implements the H/Na overlap correction: derives the sodiated-to-protonated
    ratio from class-specific standards and applies it pixel-wise to [M+H]+ images
    during the isobaric-correction stage of the qMSI workflow'
  repo: github.com/swinnenteam/LipidQMap
- name: NumPy
  role: 'Array engine for the pixel-wise arithmetic: the per-sample image dictionaries
    are dict[str, numpy.ndarray], NaN masking and subtraction are NumPy operations'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.10.15.682422
    title: LipidQMap - An Open-Source Tool for Quantitative Mass Spectrometry Imaging
      of Lipids
  source_content_hash: sha256:8cb32b3d7c62b84f56a7ff8dd6d9d3302b6623157970a02c9c08e55e3c1e1ce4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - LipidQMap
  - NumPy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# sodium-protonated-adduct-ratio-correction

## Summary

LipidQMap's pixel-wise correction that removes the isobaric contribution of [M+Na]+ lipid ions (Δm/z 0.0025 from [M+H]+, requiring ~600,000 resolution to separate) from protonated-channel ion images using sodiated-to-protonated adduct ratios measured from co-sprayed lipid class-specific standards.

## When to use

Use when positive-mode qMSI lipid images contain [M+H]+ adducts isobarically overlapped by [M+Na]+ species of the same lipid class that cannot be mass-resolved (Δm/z 0.0025), especially in datasets with prevalent M+Na adducts.

## When NOT to use

- Datasets acquired with sufficient resolving power to mass-resolve the [M+H]+/[M+Na]+ pair directly — the Δm/z of 0.0025 requires a minimum resolution of 600,000, so if the instrument achieves this the signals need no ratio-based deconvolution
- Lipid classes flagged as unsupported by the H/Na overlap correction — the images must be passed through unchanged rather than corrected
- Datasets with no prevalent [M+Na]+ adducts, where the optional Na/H overlap correction adds no value over raw or Type II-corrected images

## Inputs

- Per-sample image dictionary (dict[str, numpy.ndarray]) of quantitative ion images and summed-adduct images keyed by species ID (e.g. from LipidQMap's .imzML or .h5ad/.h5mu import pipeline)
- Co-sprayed lipid class-specific internal standard signals (used to derive the sodiated-to-protonated adduct ratio per class)
- Lipid annotation database (.csv with nine required columns) indicating species, adduct ([M+H]+ vs [M+Na]+), and lipid class
- Per-lipid-class flags indicating whether the class is supported by the H/Na overlap correction

## Outputs

- Corrected quantitative [M+H]+ ion images with the sodium-adduct contribution subtracted, written back as one pickle file per sample (dict[str, numpy.ndarray])
- Unmodified (bit-identical) images for unsupported lipid classes

## How to apply

For each lipid species annotated as [M+H]+, retrieve the corresponding [M+Na]+ summed-adduct image from the per-sample image dictionary (dict[str, numpy.ndarray] keyed by species ID) and compute the sodiated-to-protonated adduct ratio (RA_Na) from the co-sprayed, class-specific internal standard signals. Apply this ratio pixel-wise to estimate the sodium-adduct (including isotopic) contribution to the protonated channel and subtract it from the quantitative [M+H]+ image, using only the selected adducts in the arithmetic and treating NaN pixels in summed-adduct images as absent. Skip lipid classes flagged as unsupported by the H/Na overlap correction and pass their images through unchanged. The correction is applied sequentially (before/after Type II isotope correction per the import dialog's correction choice), and corrected images can be exported individually or as panel collections.

## Related tools

- **LipidQMap** (Implements the H/Na overlap correction: derives the sodiated-to-protonated ratio from class-specific standards and applies it pixel-wise to [M+H]+ images during the isobaric-correction stage of the qMSI workflow) — github.com/swinnenteam/LipidQMap
- **NumPy** (Array engine for the pixel-wise arithmetic: the per-sample image dictionaries are dict[str, numpy.ndarray], NaN masking and subtraction are NumPy operations)

## Evaluation signals

- Images of unsupported lipid classes in the output pickle are bit-identical to their input images (correction skipped, not zeroed)
- Corrected protonated images contain no values propagated from NaN entries of the summed-adduct images (NaN pixels treated as absent)
- Each output pickle loads as a dict[str, numpy.ndarray] with keys matching the input species IDs
- Corrected [M+H]+ images should show reduced intensity specifically where the corresponding [M+Na]+ image has signal, with the reduction proportional to the class-derived RA_Na
- Toggling LipidQMap's view between Raw, Isotope corrected, and Quantitative data (R/I/Q) shows the corrected protonated image differing from raw while non-overlapped species remain consistent

## Limitations

- The correction performs optimally with protonating matrices; further research and optimization are needed to leverage the correction in qMSI with various MALDI matrices
- Not all lipid classes are supported — the correction skips flagged classes, leaving their [M+H]+ images uncorrected
- The isobaric Δm/z of 0.0025 cannot be resolved instrumentally below ~600,000 resolution, so correctness depends entirely on the standard-derived ratio rather than spectral separation

## Evidence
