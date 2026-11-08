---
name: msi-spatial-colocalization-analysis
description: Use when you have MALDI FT-ICR MSI spectra (e.g. HDF5 with ~85,062 m/z
  variables) plus a pixel-level tumor-class prediction map, and you need to find and
  annotate the m/z peaks most co-localized with tumor tissue.
when_to_use_negative:
- Input is already a curated peak/feature table without spatial pixel-level ion images
  — Pearson colocalization with a class-prediction map requires reconstructed spatial
  patterns per peak.
- The classification task is multi-class or involves tissue types where ground-truth
  histology annotation is unreliable (the article demonstrates binary tumor/normal
  classification and notes intra-tumor heterogeneity limits ground-truth quality).
- You need absolute metabolite quantification — this skill yields tentative formula/adduct
  annotations with ppm error, not verified identifications.
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2640
tools:
- name: massNet
  role: Deep learning VAE whose Code layer (5-dim latent variable) and decoder provide
    the latent representation and per-m/z spatial pattern reconstruction used for
    colocalization analysis
  repo: https://github.com/wabdelmoula/massNet
- name: msiPL
  role: Method used to apply Pearson correlation between spatial patterns and VAE-identified
    peaks
- name: HMDB
  role: Database searched for tentative molecular formula/adduct annotation of retained
    m/z peaks (e.g. C9H13N2O9P [M+2K-H]+ at -2.1 ppm)
- name: METLIN
  role: Database searched for tentative annotation of retained m/z peaks (e.g. C10H16N5O13P3
    [M+K]+ at 0.36 ppm)
- name: Python
  role: Implementation environment for correlation computation and table export (Python
    3.6.12 with numpy, scipy, pandas, h5py)
  repo: https://github.com/wabdelmoula/massNet
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:3bf92462d1f29e940d64a3678e9e2f4540ffdc422f514e4c68b9c356d1052a17
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/wabdelmoula/massNet
  repo_url: https://github.com/wabdelmoula/massNet
  tools:
  - HMDB
  - METLIN
  - Python
  - massNet
  - msiPL
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# msi-spatial-colocalization-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies tumor-associated m/z features in mass spectrometry imaging data by reconstructing spatial ion images from a VAE decoder and selecting peaks whose spatial patterns correlate with the predicted tumor-class map (Pearson correlation ≥ 0.7), followed by database annotation.

## When to use

Use when you have MALDI FT-ICR MSI spectra (e.g. HDF5 with ~85,062 m/z variables) plus a pixel-level tumor-class prediction map, and you need to find and annotate the m/z peaks most co-localized with tumor tissue.

## When NOT to use

- Input is already a curated peak/feature table without spatial pixel-level ion images — Pearson colocalization with a class-prediction map requires reconstructed spatial patterns per peak.
- The classification task is multi-class or involves tissue types where ground-truth histology annotation is unreliable (the article demonstrates binary tumor/normal classification and notes intra-tumor heterogeneity limits ground-truth quality).
- You need absolute metabolite quantification — this skill yields tentative formula/adduct annotations with ppm error, not verified identifications.

## Inputs

- HDF5-format MALDI FT-ICR MSI data with ~85,062 m/z variables (converted from imzML exported from SCiLS lab 2020a)
- Trained massNet VAE latent representation (5-dimensional Code layer output) and decoder weights from github.com/wabdelmoula/massNet
- Pixel-level tumor-class probability/prediction map (e.g. from massNet classification, trained on 10,024 normal and 3,644 tumor spectra)

## Outputs

- Ranked table of tumor-correlated m/z peaks (m/z, Pearson correlation, formula, adduct, database, ppm error), e.g. tumor_correlated_features.tsv
- Set of highly co-localized m/z peaks with Pearson correlation ≥ 0.7, annotated against HMDB and METLIN
- Spatial ion images of the top tumor-associated peaks (e.g. m/z 400.9546 ± 0.01)

## How to apply

Load the HDF5-format MALDI FT-ICR MSI data (85,062 m/z variables from 9.4 Tesla SolariX acquisitions at 100 µm spatial resolution) and the 5-dimensional latent Code layer output from the trained massNet VAE (github.com/wabdelmoula/massNet). Reconstruct the spatial image of each m/z peak via the VAE decoder module and compute the Pearson correlation between each peak's spatial pattern and the tumor-class prediction map. Retain only peaks with Pearson correlation coefficient ≥ 0.7 as highly co-localized tumor-associated features; in the demonstrated GBM PDX dataset the top peak was m/z 400.9546 ± 0.01, elevated in the tumor region. Annotate each retained peak with a tentative molecular formula and adduct (e.g. [M+K]+, [M+2K-H]+) by searching HMDB and METLIN, recording the ppm error for each assignment (e.g. C9H13N2O9P [M+2K-H]+ at -2.1 ppm, C10H16N5O13P3 [M+K]+ at 0.36 ppm). Rank features by descending correlation and export the ranked table (m/z, correlation, formula, adduct, database, ppm error) for tumor delineation.

## Related tools

- **massNet** (Deep learning VAE whose Code layer (5-dim latent variable) and decoder provide the latent representation and per-m/z spatial pattern reconstruction used for colocalization analysis) — https://github.com/wabdelmoula/massNet
- **msiPL** (Method used to apply Pearson correlation between spatial patterns and VAE-identified peaks)
- **HMDB** (Database searched for tentative molecular formula/adduct annotation of retained m/z peaks (e.g. C9H13N2O9P [M+2K-H]+ at -2.1 ppm))
- **METLIN** (Database searched for tentative annotation of retained m/z peaks (e.g. C10H16N5O13P3 [M+K]+ at 0.36 ppm))
- **Python** (Implementation environment for correlation computation and table export (Python 3.6.12 with numpy, scipy, pandas, h5py)) — https://github.com/wabdelmoula/massNet

## Examples

```
import h5py; data = h5py.File('msi_data.h5','r')['Data']; r = np.corrcoef(peak_images.reshape(85062,-1), tumor_map.reshape(1,-1))[0]; peaks = mz[r >= 0.7]
```

## Evaluation signals

- Exported table contains only entries with Pearson correlation ≥ 0.7.
- Top-ranked peak matches m/z 400.9546 ± 0.01 and its ion image is elevated in the tumor region.
- Table 2-style assignments reproduced, including HMDB and METLIN annotations with reported ppm errors (C9H13N2O9P [M+2K-H]+ at -2.1 ppm; C10H16N5O13P3 [M+K]+ at 0.36 ppm).
- Output table schema contains m/z, correlation, formula, adduct, database, and ppm error columns, ranked by descending correlation.
- Correlation of each peak's spatial pattern was computed against the tumor-class prediction map, not against raw spectra.

## Limitations

- Annotations are tentative — only molecular formula/adduct assignments with ppm errors from HMDB and METLIN, not confirmed identifications.
- Ground-truth annotation based solely on annotated histology is an approximation and may not be optimal, especially for intra-tumor heterogeneity.
- massNet was demonstrated only for binary class classification; multi-class extension is future work, which constrains the class-prediction map this colocalization step depends on.
- No software version numbers or hyperparameter values for the trained massNet model are reported in the article text, complicating exact reproduction.

## Evidence
