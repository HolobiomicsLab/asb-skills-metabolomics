---
name: matrix-region-detection-in-maldi-msi
description: Use when pre-processing MALDI MSI (e.g. t-MALDI-2) imzML data and a binary
  mask separating matrix/off-tissue pixels from tissue-covered pixels must be generated
  automatically, without manually defined matrix peak lists.
when_to_use_negative:
- Input is already a peak-picked or feature table rather than raw pixel-level spectra,
  since the method depends on spectral similarity of full pixel spectra
- The sample covers the entire measured area with no matrix at the borders, so no
  border-connected matrix cluster exists
- The goal is lipid annotation or interpretation — msiFlow explicitly does not include
  an annotation or interpretation step
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: msiFlow
  role: Wraps the UMAP + HDBSCAN matrix detection and mask post-processing into an
    automated Snakemake workflow
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: imzy
  role: Reader interface to load raw imzML pixel spectra before dimensionality reduction
  repo: https://github.com/vandeplaslab/imzy
- name: Snakemake
  role: Workflow engine automating and parallelizing the matrix detection steps across
    samples
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:3518727676fc2c4f5491f8e24aa47970afcfcc71afbc4ea62a2fe28e11ca0d67
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Immunodynamics-Engel-Lab/msiflow
  repo_url: https://github.com/Immunodynamics-Engel-Lab/msiflow
  tools:
  - Snakemake
  - imzy
  - msiFlow
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# matrix-region-detection-in-maldi-msi

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Unsupervised detection of matrix/off-tissue pixels in MALDI MSI samples by embedding pixel spectra with UMAP, clustering with HDBSCAN, and selecting the border-connected cluster with the most pixels as the matrix cluster — no prior definition of known matrix peaks is required.

## When to use

Use when pre-processing MALDI MSI (e.g. t-MALDI-2) imzML data and a binary mask separating matrix/off-tissue pixels from tissue-covered pixels must be generated automatically, without manually defined matrix peak lists.

## When NOT to use

- Input is already a peak-picked or feature table rather than raw pixel-level spectra, since the method depends on spectral similarity of full pixel spectra
- The sample covers the entire measured area with no matrix at the borders, so no border-connected matrix cluster exists
- The goal is lipid annotation or interpretation — msiFlow explicitly does not include an annotation or interpretation step

## Inputs

- Raw MALDI MSI pixel spectra in imzML format (e.g. t-MALDI-2 acquisitions)
- Measured-area geometry (pixel grid) for determining border pixels

## Outputs

- Per-sample binary matrix mask saved as matrix_mask.tif
- HDBSCAN cluster labels over 2D UMAP-embedded pixels

## How to apply

Load raw imzML pixel spectra with the imzy reader interface, then reduce each sample's pixel spectra to a 2D UMAP embedding so that spectrally similar pixels (matrix vs. tissue) group together. Cluster the 2D-embedded pixels with HDBSCAN density-based clustering. Identify the HDBSCAN cluster connected to the largest number of border pixels of the measured area and designate it as the matrix cluster; the rationale is that matrix surrounds the tissue, so the matrix signal always touches the measurement border, letting the method work on whole-sample measurements. Extend the selected cluster by merging remaining clusters whose mean spectra correlate with it, then apply morphological post-processing (closing/smoothing) to obtain a coherent binary mask. Save the mask per sample as matrix_mask.tif and visually verify it covers the matrix shell around the tissue and excludes tissue pixels.

## Related tools

- **msiFlow** (Wraps the UMAP + HDBSCAN matrix detection and mask post-processing into an automated Snakemake workflow) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **imzy** (Reader interface to load raw imzML pixel spectra before dimensionality reduction) — https://github.com/vandeplaslab/imzy
- **Snakemake** (Workflow engine automating and parallelizing the matrix detection steps across samples)

## Examples

```
from imzy import get_reader; reader = get_reader("sample.imzML"); mzs, ints = reader.get_spectrum(0)
```

## Evaluation signals

- The selected matrix cluster is the border-connected cluster containing the most pixels
- The post-processed binary mask forms a coherent matrix shell around the measured tissue area
- Visual inspection confirms the mask covers matrix regions surrounding the tissue and excludes tissue-covered pixels
- Merging by mean-spectrum correlation does not absorb tissue clusters into the matrix mask after morphological closing/smoothing

## Limitations

- The segmentation example data requires at least 120 GB RAM, so large t-MALDI-2 datasets may be impractical to process locally (a smaller demo sample can be substituted)
- Effectiveness relies on tissue architecture and borders, so it is limited to whole-sample measurements where matrix touches the measurement border

## Evidence

- [discussion] Automated matrix pixel detection via UMAP and HDBSCAN using tissue architecture and borders: [evidence span withheld — anchor status: not_found]
- [readme] imzy provides a common reader interface for imzML, Bruker, and Waters imaging mass spectrometry data.: [evidence span withheld — anchor status: external_doc]
