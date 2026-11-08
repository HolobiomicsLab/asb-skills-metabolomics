---
name: mass-spectrometry-imaging-signal-preprocessing
description: Use when raw MALDI-MSI imzML pixel spectra must be made comparable across
  a tissue section (TIC normalization), denoised, and background-corrected before
  clustering or segmentation with MSIght.
when_to_use_negative:
- Input is H&E histology image data — MSIght handles this via a separate H&E image
  pre-processing workflow (Figure S6), not the spectral pre-processing pipeline.
- Input is LC-MS/MS peptide identification data, which follows MSIght's dedicated
  LC-MS/MS workflow (Figure S8) rather than MSI spectral pre-processing.
- Spectra have already been TIC-normalized, smoothed, and baseline-corrected downstream
  of load_and_preprocess_imzml and only need segmentation (e.g. cluster_msi).
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: MSIght
  role: Modular platform whose common-functions module (load_and_preprocess_imzml,
    backed by pyimzML) executes the TIC normalization, Gaussian smoothing, and tophat
    baseline correction pipeline on imzML data
  repo: https://github.com/lingjunli-research/MSIght
- name: Python
  role: Implementation language for the distributable MSIght package and its pre-processing
    pipeline
- name: SciPy
  role: Morphology operations used to perform white tophat baseline subtraction with
    a structuring element of size 10
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.4c01140
    title: 'MSIght: A Modular Platform for Improved Confidence in Global, Untargeted
      Mass Spectrometry Imaging Annotation'
  source_content_hash: sha256:f158861d708c02195c227927c6a0d971d1bb4e5afed824c218b73d9915606c6f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lingjunli-research/MSIght
  tools:
  - MSIght
  - Python
  - SciPy
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# mass-spectrometry-imaging-signal-preprocessing

## Summary

MSIght's spectral pre-processing stage for MALDI-MSI imzML data: per-pixel TIC normalization, Gaussian smoothing with sigma=2, and white tophat baseline correction with a structuring element of size 10, producing cleaned spectra ready for downstream dimensionality reduction and segmentation.

## When to use

Use when raw MALDI-MSI imzML pixel spectra must be made comparable across a tissue section (TIC normalization), denoised, and background-corrected before clustering or segmentation with MSIght.

## When NOT to use

- Input is H&E histology image data — MSIght handles this via a separate H&E image pre-processing workflow (Figure S6), not the spectral pre-processing pipeline.
- Input is LC-MS/MS peptide identification data, which follows MSIght's dedicated LC-MS/MS workflow (Figure S8) rather than MSI spectral pre-processing.
- Spectra have already been TIC-normalized, smoothed, and baseline-corrected downstream of load_and_preprocess_imzml and only need segmentation (e.g. cluster_msi).

## Inputs

- Raw MALDI-MSI imzML file (imzML-backed intensity matrix of spatial pixels, loaded via pyimzML)

## Outputs

- preprocessed_msi_data.imzML — TIC-normalized, Gaussian-smoothed (sigma=2), tophat baseline-corrected imzML-backed spectra ready for downstream analysis or visualization

## How to apply

Load the raw imzML file with MSIght's common-functions module (load_and_preprocess_imzml, backed by pyimzML) to build the intensity matrix of spatial pixels. Apply TIC (total ion current) normalization to each pixel's spectrum so intensities are comparable across the tissue section. Apply a Gaussian filter with sigma=2 to smooth the spectral image data and suppress pixel-level noise. Perform white tophat baseline subtraction with a structuring element of size 10 using SciPy's morphology operations to remove broad chemical background from each spectrum. Write the TIC-normalized, smoothed, baseline-corrected spectra back into the imzML-backed data structure (preprocessed_msi_data.imzML in the designated output directory) so the results object can flow onward to MSIght's segmentation (e.g. cluster_msi from MSIght.refactor_segment).

## Related tools

- **MSIght** (Modular platform whose common-functions module (load_and_preprocess_imzml, backed by pyimzML) executes the TIC normalization, Gaussian smoothing, and tophat baseline correction pipeline on imzML data) — https://github.com/lingjunli-research/MSIght
- **Python** (Implementation language for the distributable MSIght package and its pre-processing pipeline)
- **SciPy** (Morphology operations used to perform white tophat baseline subtraction with a structuring element of size 10)

## Examples

```
pip install msight; from MSIght.common_functions import load_and_preprocess_imzml; data = load_and_preprocess_imzml('raw_data.imzML', output_dir='results/')
```

## Evaluation signals

- The output file preprocessed_msi_data.imzML exists in the designated output directory in imzML-backed format.
- The preprocessed file loads without error through the same pyimzML reader used for the raw input.
- Each pixel's spectrum is TIC-normalized so total ion current is comparable across the tissue section.
- Spectral smoothing used a Gaussian filter with sigma=2 and baseline correction used a white tophat filter with structuring element size 10.
- The returned results object can be passed onward for further analysis or visualization (e.g. into MSIght's cluster_msi segmentation).

## Limitations

- Pre-processing parameters (Gaussian sigma=2, tophat structuring element size 10) are the workflow defaults; the provided context does not describe alternate settings for other tissue or instrument types.
- This skill covers only the spectral pre-processing stage (Figure S1); full MSIght analysis additionally requires H&E pre-processing (Figure S6), linear and non-linear registration (Figure S7), and segmentation steps.

## Evidence
