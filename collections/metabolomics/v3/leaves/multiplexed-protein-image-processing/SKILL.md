---
name: multiplexed-protein-image-processing
description: Use when raw IMC ROI image stacks (multi-channel, metal-isotope antibody
  channels) from the scSpaMet raw data deposit need noise removal and intensity normalization
  before segmentation, registration, or VAE joint embedding.
when_to_use_negative:
- Input is already a single-cell feature/expression table (e.g. post-segmentation
  protein intensities) rather than raw or scaled IMC image pixels
- Processing TOF-SIMS/3D-SMF metabolite image channels — the SIMS modality has its
  own processing track in notebook 01 and should not receive IMC antibody-channel
  denoising parameters
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3383
- http://edamontology.org/topic_0121
tools:
- name: scSpaMet
  role: Provides the image-processing notebook ('01 Processing of IMC (protein) and
    SIMS (metabolite) images') that implements percentile scaling and k-NN denoising
    of IMC ROI stacks
  repo: github.com/coskunlab/ScSpaMet
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-43917-5
    title: Single-cell spatial metabolomics with cell-type specific protein profiling
      for tissue systems biology
  source_content_hash: sha256:9735a503bcc2760dadb2595c61b475801c3e9ec9dafca8c0c329915a210456f6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scSpaMet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3383
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# multiplexed-protein-image-processing

## Summary

Preprocessing of multiplexed IMC (imaging mass cytometry) protein image stacks — percentile scaling to 0–1 followed by channel-wise k-NN spatial denoising — within the scSpaMet pipeline, so that targeted protein imaging at 1 µm/px (up to ~40 markers) is ready for downstream single-cell segmentation and protein-metabolite integration.

## When to use

Use when raw IMC ROI image stacks (multi-channel, metal-isotope antibody channels) from the scSpaMet raw data deposit need noise removal and intensity normalization before segmentation, registration, or VAE joint embedding.

## When NOT to use

- Input is already a single-cell feature/expression table (e.g. post-segmentation protein intensities) rather than raw or scaled IMC image pixels
- Processing TOF-SIMS/3D-SMF metabolite image channels — the SIMS modality has its own processing track in notebook 01 and should not receive IMC antibody-channel denoising parameters

## Inputs

- Raw IMC ROI protein image stacks from Zenodo 10.5281/zenodo.6784251 (multi-channel TIFF, up to 40 metal-isotope antibody markers at 1 µm/pixel)
- Percentile-scaled ROI channel images (0–1 normalized) prior to filtering

## Outputs

- imc_denoised_stack.tif — multi-channel denoised image stack per ROI with the same number of channels and ROI dimensions as the input, pixel values within the 0–1 percentile-scaled range

## How to apply

Load the raw IMC protein image stack for each region of interest from the Zenodo deposit (10.5281/zenodo.6784251) using the preprocessing routine in the scSpaMet notebook '01 Processing of IMC (protein) and SIMS (metabolite) images'. Apply percentile scaling channel-wise so each IMC channel's intensities are normalized to the 0–1 range prior to filtering. Then apply the k-NN spatial denoising filter channel-wise over each percentile-scaled ROI image to smooth pixel noise while retaining cell boundary contrast — this matters because these stacks encode cancer/epithelial, stroma, and immune cell types across up to ~40 markers at 1 µm/pixel. Stack the denoised channel images per ROI and export as imc_denoised_stack.tif for the subsequent registration and segmentation steps of the pipeline.

## Related tools

- **scSpaMet** (Provides the image-processing notebook ('01 Processing of IMC (protein) and SIMS (metabolite) images') that implements percentile scaling and k-NN denoising of IMC ROI stacks) — github.com/coskunlab/ScSpaMet

## Examples

```
jupyter notebook "notebooks/01 Processing of IMC (protein) and SIMS (metabolite) images.ipynb"  # percentile-scale channels then k-NN denoise ROI stacks -> imc_denoised_stack.tif
```

## Evaluation signals

- imc_denoised_stack.tif exists and loads as a multi-channel image array
- Output stack has the same number of channels and ROI dimensions as the input stack
- All pixel values in the denoised stack fall within the 0–1 percentile-scaled range
- Pixel noise is visibly smoothed while cell boundary contrast is retained (denoising should not blur cell segmentation cues)
- Downstream registration and segmentation steps (notebooks 02–03) accept the denoised stack without channel/dimension mismatches

## Limitations

- Applies only to the IMC protein modality; the metabolite (TOF-SIMS/3D-SMF) images are processed separately in the pipeline
- IMC images used are percentile-scaled ROI stacks; raw unnormalized stacks must be scaled first or the k-NN filter behavior is not the one validated here
- TOF-SIMS imaging trades off imaging speed against spatial resolution, yielding relatively small imaging regions — ROI size and coverage are bounded by the upstream acquisition

## Evidence

- [readme] README notebook list: "- 01 Processing of IMC (protein) and SIMS (metabolite) images"
- [readme] raw data deposit: "You can find the raw data here: https://doi.org/10.5281/zenodo.6784251."
- [results] IMC role in pipeline: "Tissue samples on glass slides are labeled with metal-isotope conjugated antibodies followed by metabolic profiling with 3D-SMF and finally proteomic profiling using IMC"
- [intro] multiplex protein imaging scope: "scSpaMet enables the correlation of more than 200 metabolic markers and 25 protein markers in individual cells within native tissues."
