---
name: imzml-msi-data-preprocessing
description: Use when input is an MS imaging dataset in imzML (+ibd) format containing
  only MS1 scans, and the goal is structural annotation of metabolites/lipids (e.g.,
  PC adduct forms, hemin) from ion images.
when_to_use_negative:
- Input is LC-MS data (mzML/mzXML chromatographic runs) — use the ms1_id LC-MS workflow
  with XIC correlation instead of ion-image correlation
- Input is already a pseudo MS/MS mgf file — annotate it directly with 'ms1_id annotate'
  rather than re-deriving spectra from imaging data
- The dataset contains MS/MS scans that should be matched directly (use the unscaled
  gnps.pkl MS/MS library path instead of the k10-scaled MS1 annotation path)
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: ms1-id
  role: 'Python package performing the full imaging-MS pipeline: imzML preprocessing,
    ion image correlation, pseudo MS/MS generation, and reverse spectral matching'
  repo: https://github.com/Philipbear/ms1_id
- name: GNPS library (indexed)
  role: Reference MS/MS spectral library indexed with peak_scale_k=10 and peak_intensity_power=0.5
    (minmz100 variant for MSI); supplied as the --libs argument
  repo: https://github.com/Philipbear/ms1_id
- name: MassCube
  role: Used to flag isotope peaks, which are excluded from pseudo MS/MS generation
    during imaging preprocessing
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.10.14.618269
    title: 'Structural annotation of full-scan MS data: A unified solution for LC-MS
      and MS imaging analyses'
  source_content_hash: sha256:d3fc0ba5e6afe2e0acc7c22acd1d5c000275a75655aa32552593322c6eb71ce7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/Philipbear/ms1_id
  tools:
  - GNPS library (indexed)
  - MassCube
  - ms1-id
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 22
---

# imzml-msi-data-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess and annotate MS1-only mass spectrometry imaging datasets stored in imzML/ibd format using the ms1-id Python package: per-scan noise reduction, centroiding, ion image extraction at 10 ppm, spatial correlation to build pseudo MS/MS spectra, and reverse spectral matching against a peak-scale-adjusted indexed GNPS library.

## When to use

Use when input is an MS imaging dataset in imzML (+ibd) format containing only MS1 scans, and the goal is structural annotation of metabolites/lipids (e.g., PC adduct forms, hemin) from ion images.

## When NOT to use

- Input is LC-MS data (mzML/mzXML chromatographic runs) — use the ms1_id LC-MS workflow with XIC correlation instead of ion-image correlation
- Input is already a pseudo MS/MS mgf file — annotate it directly with 'ms1_id annotate' rather than re-deriving spectra from imaging data
- The dataset contains MS/MS scans that should be matched directly (use the unscaled gnps.pkl MS/MS library path instead of the k10-scaled MS1 annotation path)

## Inputs

- MS imaging dataset in imzML format with accompanying .ibd file (e.g., from MetaboLights MTBLS313)
- Indexed MS/MS reference library built for imaging data (e.g., gnps_minmz100.pkl and gnps_minmz100_k10.pkl from the ms1_id release)

## Outputs

- Annotation table ms1_id_annotations_derep.tsv with metabolite annotations and reverse cosine scores per pixel-clustering pseudo MS/MS spectrum
- Spatially correlated pseudo MS/MS (composite) spectra compiled from ion image pairs
- Ion images for annotated features (e.g., PC(16:0/18:1) [M+H]+ and [M+Na]+, hemin cation and its in-source fragments)

## How to apply

Convert/obtain the imaging dataset as paired imzML/ibd files (e.g., MetaboLights MTBLS313 mouse brain, MALDI-FTICR), download the indexed library built for imaging data (gnps_minmz100, where fragments with m/z < 100 are removed since they are not usually captured in MSI), and run the ms1-id MSI workflow. Preprocess each MS scan with moving-average noise reduction (100 Da window; baseline = 5x the mean of the lowest 5% ions in the window) and centroiding with default sn_factor 3.0. Extract ion images with a 10 ppm mass tolerance and spatially correlate ion image pairs, requiring at least 50 shared non-zero pixels and a spatial correlation cutoff of 0.85, to compile pseudo MS/MS spectra; isotope peaks determined by MassCube are excluded. Search the pseudo spectra against the peak-scaled (k = 10, square-root intensity power) reference library using precursor-tolerant reverse spectral matching with MSI cutoffs: minimum reverse cosine score 0.7, minimum matched peaks 3, minimum spectral usage 0.05. The rationale is that low-energy in-source fragments spatially co-located with the precursor ion provide composite spectra that can match reference MS/MS libraries collected at higher collision energies.

## Related tools

- **ms1-id** (Python package performing the full imaging-MS pipeline: imzML preprocessing, ion image correlation, pseudo MS/MS generation, and reverse spectral matching) — https://github.com/Philipbear/ms1_id
- **GNPS library (indexed)** (Reference MS/MS spectral library indexed with peak_scale_k=10 and peak_intensity_power=0.5 (minmz100 variant for MSI); supplied as the --libs argument) — https://github.com/Philipbear/ms1_id
- **MassCube** (Used to flag isotope peaks, which are excluded from pseudo MS/MS generation during imaging preprocessing)

## Examples

```
ms1_id msi --input_dir msi --libs db/gnps_minmz100.pkl db/gnps_minmz100_k10.pkl --n_cores 12
```

## Evaluation signals

- PC(16:0/18:1) must be annotated in both [M+H]+ and [M+Na]+ forms with reverse cosine scores of 0.85 and 0.77, respectively, on the MTBLS313 mouse brain dataset
- Each spatially correlated ion image pair must share at least 50 non-zero pixels and pass the 0.85 spatial correlation cutoff before entering the pseudo MS/MS spectrum
- Reported annotations must satisfy minimum reverse cosine score 0.7, minimum 3 matched peaks, and minimum spectral usage 0.05
- Output annotation table ms1_id_annotations_derep.tsv must exist and contain the expected annotation columns for every imzML file in the input directory
- The hemin cation ion image must show spatial patterns highly similar to its in-source fragment ion images

## Limitations

- Expected runtime is ~3-20 min per imaging dataset and requires at least 12 cores for parallel processing; fewer cores will slow the run
- Only imzML imaging files are supported; the workflow annotates each imzML file individually rather than jointly
- Reference spectra are largely medium/high collision energy while in-source fragments are low energy, motivating the mass-dependent peak scaling (k = 10); without scaling fewer matches exceed score 0.7

## Evidence

- [readme] MSI preprocessing pipeline: [evidence span withheld — anchor status: external_doc]
- [readme] MSI library variant: [evidence span withheld — anchor status: external_doc]
- [readme] MSI output location: [evidence span withheld — anchor status: external_doc]
- [readme] Runtime/parallelism expectation: [evidence span withheld — anchor status: external_doc]
