---
name: ms-imaging-ion-image-extraction
description: Use when input is MS1-only imaging MS data in imzML/ibd format (e.g.,
  MALDI-Orbitrap mouse liver on METASPACE) and the goal is structural annotation via
  pseudo MS/MS built from spatially correlated in-source fragments.
when_to_use_negative:
- The data contain true MS/MS scans (MS2 spectra are available) — use conventional
  MS/MS library search instead of pseudo MS/MS generation
- The input is LC-MS mzML/mzXML data where correlation should be done on XICs in the
  retention-time domain, not on spatial ion images
- The target analytes produce too few in-source fragments above m/z 100 (e.g., guanine
  m/z 150.0421 and glutamic acid m/z 146.0459 failed for lack of sufficient in-source
  fragments)
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3172
tools:
- name: ms1-id
  role: Python package that preprocesses imzML scans, extracts ion images at 10 ppm
    tolerance, spatially correlates them into pseudo MS/MS spectra, and searches them
    against indexed GNPS libraries
  repo: https://github.com/Philipbear/ms1_id
- name: METASPACE
  role: Public repository hosting the imzML/ibd imaging datasets (e.g., mouse liver
    with seven spotted standards, dataset ID 2020-12-07_03h16m14s)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.10.14.618269
    title: 'Structural annotation of full-scan MS data: A unified solution for LC-MS
      and MS imaging analyses'
  source_content_hash: sha256:f075dcbf1447fcf4cbe8761299dd7d718cc986b843f95cc45dbedd2acd790a58
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
  - METASPACE
  - ms1-id
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# ms-imaging-ion-image-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract ion images from MS1-only imaging MS data (imzML/ibd) and spatially correlate them to assemble pseudo MS/MS spectra, enabling MS/MS library-based structural annotation of MALDI-imaging datasets with ms1-id.

## When to use

Use when input is MS1-only imaging MS data in imzML/ibd format (e.g., MALDI-Orbitrap mouse liver on METASPACE) and the goal is structural annotation via pseudo MS/MS built from spatially correlated in-source fragments.

## When NOT to use

- The data contain true MS/MS scans (MS2 spectra are available) — use conventional MS/MS library search instead of pseudo MS/MS generation
- The input is LC-MS mzML/mzXML data where correlation should be done on XICs in the retention-time domain, not on spatial ion images
- The target analytes produce too few in-source fragments above m/z 100 (e.g., guanine m/z 150.0421 and glutamic acid m/z 146.0459 failed for lack of sufficient in-source fragments)

## Inputs

- imzML file with paired ibd file (MS1-only imaging MS data, e.g., MALDI-Orbitrap or MALDI-FTICR acquisitions)
- Indexed GNPS MS/MS library for imaging data (gnps_minmz100.pkl and gnps_minmz100_k10.pkl with peak_scale_k=10, peak_intensity_power=0.5)

## Outputs

- Ion images per m/z feature extracted at 10 ppm mass tolerance
- Pseudo MS/MS (composite) spectra assembled from spatially correlated in-source fragments
- Annotation results table ms1_id_annotations_derep.tsv with library matches per feature

## How to apply

Load the imzML/ibd pair and preprocess each MS scan with moving-average noise reduction and a 100 Da moving-window baseline (5x mean of the lowest 5% ions), centroiding if needed. Extract ion images for each m/z feature using a 10 ppm mass tolerance. Spatially correlate ion image pairs, requiring at least 50 shared pixels with non-zero intensities and a correlation cutoff of 0.85, to cluster co-localized ions (precursor plus in-source fragments) into a pseudo MS/MS spectrum per feature. Search each pseudo MS/MS spectrum against the peak-scaled GNPS library (e.g., gnps_minmz100.pkl with k=10 and square-root intensities) in precursor-tolerant reverse cosine mode with MS imaging cutoffs of minimum score 0.7, minimum matched peaks 3, and minimum spectral usage 0.05. Rationale: almost all MS imaging metabolomics data are MS1 scans, and spatial co-localization of fragments plays the same role as chromatographic co-elution in LC-MS pseudo MS/MS generation; fragments below m/z 100 are typically absent in imaging data, so the imaging-specific library removes them.

## Related tools

- **ms1-id** (Python package that preprocesses imzML scans, extracts ion images at 10 ppm tolerance, spatially correlates them into pseudo MS/MS spectra, and searches them against indexed GNPS libraries) — https://github.com/Philipbear/ms1_id
- **METASPACE** (Public repository hosting the imzML/ibd imaging datasets (e.g., mouse liver with seven spotted standards, dataset ID 2020-12-07_03h16m14s))

## Examples

```
ms1_id msi --input_dir msi --libs db/gnps_minmz100.pkl db/gnps_minmz100_k10.pkl --n_cores 12
```

## Evaluation signals

- On the mouse liver ground-truth dataset with seven spotted standards (ATP, ADP, adenosine, guanine, glutamic acid, glutamine, glutathione), exactly 5 of 7 standards receive annotations; guanine and glutamic acid fail for lack of in-source fragments
- All spatially correlated ion-image pairs share at least 50 non-zero pixels and pass the 0.85 correlation cutoff before joining a pseudo MS/MS spectrum
- All library hits meet the MS imaging cutoffs: reverse cosine score >= 0.7, matched peaks >= 3, spectral usage >= 0.05
- The peak-scaled condition (k=10) yields more matching scores > 0.7 for the ground-truth compounds than the non-scaled condition
- No query peaks below m/z 100 are matched because the imaging library (gnps_minmz100) excludes fragments with m/z < 100

## Limitations

- Standards that do not generate sufficient in-source fragment ions within the collected m/z range (e.g., guanine, glutamic acid) cannot be confidently annotated
- Expected runtime is ~3-20 min per MS imaging dataset, requiring at least 12 cores for the benchmarked performance; longer runs may indicate parameter issues
- Fragment ions below m/z 100 are not matched, as they are not usually included in MS imaging data and are removed from the imaging library

## Evidence

- [readme] Here, `msi` is the input directory consisting of the imzML and ibd files. All the imzML files in the directory will be annotated individually.: [evidence span withheld — anchor status: external_doc]
- [readme] For MS imaging data (fragments with mz < 100 are removed, as they are not usually included in MS imaging data): [evidence span withheld — anchor status: external_doc]
