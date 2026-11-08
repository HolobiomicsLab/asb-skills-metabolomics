---
name: spatial-ion-image-correlation
description: Use when input is MS1-only imaging data (imzML/ibd) and structural annotation
  is needed by grouping spatially correlated ion images into pseudo MS/MS spectra.
when_to_use_negative:
- Input is LC-MS chromatographic data with retention time profiles rather than ion
  images — use the XIC-domain correlation variant instead of spatial correlation
- Data already contains true MS/MS scans for the features of interest — direct MS/MS
  library search (e.g. with gnps.pkl) is more appropriate than reconstructing pseudo
  MS/MS from MS1 co-localization
- Ion images are too sparse for the minimum 50 shared non-zero pixel requirement (e.g.
  low-abundance features or very small regions of interest)
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: ms1-id
  role: Python package performing noise reduction, ion image extraction, spatial correlation
    clustering into pseudo MS/MS spectra, and reverse spectral matching against indexed
    libraries for imzML inputs
  repo: https://github.com/Philipbear/ms1_id
- name: MassCube
  role: backend feature detection used to extract metabolic features and flag isotope
    peaks that are excluded from pseudo MS/MS generation
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1101/2024.10.14.618269
    title: 'Structural annotation of full-scan MS data: A unified solution for LC-MS
      and MS imaging analyses'
  source_content_hash: sha256:b0a7894822b4fd2bf4942ba727db0c13cc4335043f20ee693c116fc82137b607
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
  - MassCube
  - Numba
  - ms1-id
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# spatial-ion-image-correlation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Clusters ions in MS imaging data by correlating their extracted ion images, so that in/post-source fragments co-localized with a precursor feature can be merged into a pseudo MS/MS spectrum for structural annotation with ms1-id.

## When to use

Use when input is MS1-only imaging data (imzML/ibd) and structural annotation is needed by grouping spatially correlated ion images into pseudo MS/MS spectra.

## When NOT to use

- Input is LC-MS chromatographic data with retention time profiles rather than ion images — use the XIC-domain correlation variant instead of spatial correlation
- Data already contains true MS/MS scans for the features of interest — direct MS/MS library search (e.g. with gnps.pkl) is more appropriate than reconstructing pseudo MS/MS from MS1 co-localization
- Ion images are too sparse for the minimum 50 shared non-zero pixel requirement (e.g. low-abundance features or very small regions of interest)

## Inputs

- MS imaging dataset in imzML/ibd format (MS1-only full scans)
- Indexed MS/MS library pickles with mass-dependent peak scaling (e.g. gnps_minmz100.pkl, gnps_minm100_k10.pkl built with peak_scale_k=10, peak_intensity_power=0.5)
- Preprocessing parameters: 100 Da moving-average window, baseline factor 5, lowest 5% ions, 10 ppm mass tolerance

## Outputs

- pseudo_msms.mgf containing clustered pseudo MS/MS spectra with pixel/spatial metadata and precursor feature m/z
- Annotation TSV (ms1_id_annotations_derep.tsv) with structural assignments and reverse cosine scores
- Per-ion image visualizations of annotated features

## How to apply

Preprocess each MS scan with ms1-id: apply moving-average noise reduction, then set the baseline within a moving 100 Da window as 5 times the mean intensity of the lowest 5% ions, and centroid peaks (sn_factor default 3.0) if needed. Extract ion images for each metabolic feature at ~10 ppm mass tolerance, then for every pair of ion images sharing at least 50 pixels with non-zero intensities, compute spatial correlation and cluster ions whose correlation coefficient is >= 0.85 with the target feature into a pseudo MS/MS spectrum, using MS1 peak heights as fragment intensities and excluding isotope peaks flagged by MassCube. Write the spectra to pseudo_msms.mgf, then annotate them with ms1-id against a peak-scaled indexed GNPS library (e.g. gnps_minmz100.pkl and gnps_minmz100_k10.pkl, with fragments < 100 m/z removed) using MS imaging cutoffs: minimum reverse cosine score 0.7, minimum matched peaks 3, minimum spectral usage 0.05. The rationale is that in/post-source fragments detected in MS1-only MALDI data show spatial patterns matching their precursor, so co-localization provides a fragment fingerprint analogous to chromatographic XIC correlation, enabling the first MS/MS library-based structure annotation for MS imaging data.

## Related tools

- **ms1-id** (Python package performing noise reduction, ion image extraction, spatial correlation clustering into pseudo MS/MS spectra, and reverse spectral matching against indexed libraries for imzML inputs) — https://github.com/Philipbear/ms1_id
- **MassCube** (backend feature detection used to extract metabolic features and flag isotope peaks that are excluded from pseudo MS/MS generation)

## Examples

```
ms1_id msi --input_dir msi --libs db/gnps_minmz100.pkl db/gnps_minmz100_k10.pkl --n_cores 12
```

## Evaluation signals

- pseudo_msms.mgf parses correctly and every entry contains a precursor feature m/z plus fragment peaks whose intensities derive from MS1 peak heights
- Clustering cutoffs recorded in run parameters match the paper values: spatial correlation >= 0.85, >= 50 shared non-zero pixels, ~10 ppm mass tolerance
- Annotation table (ms1_id_annotations_derep.tsv) shows only hits with reverse cosine score >= 0.7, matched peaks >= 3, and spectral usage >= 0.05
- Ground-truth recovery: PC(16:0/18:1) annotated in both [M+H]+ (score 0.85) and [M+Na]+ (score 0.77) forms on MTBLS313 mouse brain data; hemin cation ion image spatially matches its in-source fragments
- Runtime stays within ~3-20 min per MS imaging dataset when at least 12 cores are used

## Limitations

- Requires a minimum of 50 shared pixels with non-zero intensities between two ion images; features below this threshold cannot be clustered
- Fragments with m/z < 100 are excluded from the MS imaging library (gnps_minmz100) because they are not usually recorded in MS imaging data
- Spatial correlation treats chromatographic and spatial domains uniformly, so no chromatographic peak-shape validation is available for imaging runs
- ms1-id itself does not report a version number; only MassCube ver 1.0.11 is documented
- Reverse spectral search score equation and the mass-dependent peak scaling equation are not fully rendered in the article text

## Evidence

- [readme] ms1_id msi CLI invocation: [evidence span withheld — anchor status: external_doc]
