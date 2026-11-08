---
name: pearson-correlation-feature-ranking
description: Use when you have a trained VAE/latent-space MSI classifier (e.g. massNet
  Code layer output) plus decoded per-m/z spatial images and a pixel-level class-probability
  map, and need to identify which m/z peaks drive tumor delineation.
when_to_use_negative:
- You need de novo peak picking or spectral preprocessing — massNet applies Pearson
  correlation on peaks identified by the VAE decoder without prior peak picking, so
  a classical peak-picking pipeline would duplicate/conflict with that step
- Your classification map comes from a multi-class model or a different tissue/tumor
  type — the demonstrated ground truth is binary (tumor vs. normal) histology annotation,
  and the paper notes multi-class classification and ground-truth establishment for
  other tissues are future work
- You lack spatially resolved pixel-level data (e.g. only bulk spectra or a precomputed
  feature table) — the correlation is computed between spatial patterns, not summary
  intensities
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: massNet
  role: Supplies the trained VAE (5-dim Code layer) whose decoder identifies m/z peaks
    and whose classifier produces the tumor-class prediction map that spatial patterns
    are correlated against
  repo: https://github.com/wabdelmoula/massNet
- name: msiPL
  role: Applies Pearson correlation on the spatial patterns and the peaks identified
    from the VAE module to select co-localized tumor-associated peaks
- name: HMDB
  role: Database searched for tentative molecular formula/adduct annotation of retained
    peaks, with reported ppm errors
- name: METLIN
  role: Database searched for tentative molecular formula/adduct annotation of retained
    peaks, with reported ppm errors
- name: Python
  role: Implementation environment (Python 3.6.12 with Keras 2.2.0/Tensorflow 1.8.0
    backend, h5py for HDF5 I/O) for loading data and computing correlations
  repo: https://github.com/wabdelmoula/massNet
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:08454c22d649a378827fc99a62776396be3d13aa8cc8fedad6c71bd996eccb1d
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# pearson-correlation-feature-ranking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Ranks m/z peaks by the Pearson correlation between each peak's reconstructed spatial ion image and the tumor-class prediction map from the massNet VAE, retaining co-localized (correlation ≥ 0.7) tumor-associated features for molecular annotation.

## When to use

Use when you have a trained VAE/latent-space MSI classifier (e.g. massNet Code layer output) plus decoded per-m/z spatial images and a pixel-level class-probability map, and need to identify which m/z peaks drive tumor delineation.

## When NOT to use

- You need de novo peak picking or spectral preprocessing — massNet applies Pearson correlation on peaks identified by the VAE decoder without prior peak picking, so a classical peak-picking pipeline would duplicate/conflict with that step
- Your classification map comes from a multi-class model or a different tissue/tumor type — the demonstrated ground truth is binary (tumor vs. normal) histology annotation, and the paper notes multi-class classification and ground-truth establishment for other tissues are future work
- You lack spatially resolved pixel-level data (e.g. only bulk spectra or a precomputed feature table) — the correlation is computed between spatial patterns, not summary intensities

## Inputs

- HDF5-format MALDI FT-ICR MSI dataset (85,062 m/z variables, e.g. GBM PDX mouse brain tumor sections acquired at 100 µm spatial resolution on a 9.4 Tesla SolariX)
- massNet VAE latent variable (5-dimensional Code layer output) and VAE-decoded spatial ion images for identified m/z peaks
- Tumor-class prediction (probability) map from massNet pixel-based classification
- Pathologist-annotated tumor regions transferred from H&E images (ground truth for the class map)

## Outputs

- Ranked table of tumor-correlated m/z features (m/z, Pearson correlation, formula, adduct, database, ppm error), e.g. tumor_correlated_features.tsv
- Set of highly co-localized tumor-associated peaks with Pearson correlation ≥ 0.7
- HMDB and METLIN database annotations with reported ppm errors for each retained peak

## How to apply

Load the HDF5-format MALDI FT-ICR MSI data (85,062 m/z variables) and the 5-dimensional Code-layer latent variable from the massNet repository, and reconstruct the spatial image of each m/z peak identified by the VAE decoder. Compute the Pearson correlation between each peak's spatial pattern and the tumor-class prediction map, retaining only peaks with Pearson correlation coefficient T ≥ 0.7 as highly co-localized tumor-associated features. Optionally refine candidate peaks via the msiPL method, which applies Pearson correlation on the spatial patterns and the VAE-identified peaks. Annotate each retained peak with tentative molecular formula and adduct (e.g. [M+K]+, [M+2K-H]+) by searching HMDB and METLIN, recording the ppm error for each assignment (e.g. C9H13N2O9P [M+2K-H]+ at -2.1 ppm, C10H16N5O13P3 [M+K]+ at 0.36 ppm). Rank the annotated features by descending correlation and export a table of m/z, correlation, formula, adduct, database, and ppm error (e.g. tumor_correlated_features.tsv).

## Related tools

- **massNet** (Supplies the trained VAE (5-dim Code layer) whose decoder identifies m/z peaks and whose classifier produces the tumor-class prediction map that spatial patterns are correlated against) — https://github.com/wabdelmoula/massNet
- **msiPL** (Applies Pearson correlation on the spatial patterns and the peaks identified from the VAE module to select co-localized tumor-associated peaks)
- **HMDB** (Database searched for tentative molecular formula/adduct annotation of retained peaks, with reported ppm errors)
- **METLIN** (Database searched for tentative molecular formula/adduct annotation of retained peaks, with reported ppm errors)
- **Python** (Implementation environment (Python 3.6.12 with Keras 2.2.0/Tensorflow 1.8.0 backend, h5py for HDF5 I/O) for loading data and computing correlations) — https://github.com/wabdelmoula/massNet

## Examples

```
import h5py, scipy.stats; corr = scipy.stats.pearsonr(peak_image.flatten(), tumor_class_map.flatten())[0]
```

## Evaluation signals

- Every exported feature row has Pearson correlation ≥ 0.7; features below the threshold are excluded
- Top-ranked features reproduce Table 2-style assignments, including the m/z 400.9546 ± 0.01 peak most highly correlated with the tumor class and elevated in the tumor region
- Each retained peak carries a formula/adduct annotation from HMDB or METLIN with a recorded ppm error (e.g. -2.1 ppm for C9H13N2O9P [M+2K-H]+; 0.36 ppm for C10H16N5O13P3 [M+K]+)
- Exported table contains the required columns: m/z, correlation, formula, adduct, database, ppm error
- The m/z variables analyzed match the HDF5 dataset dimensionality (85,062 variables), and spatial images align with the 100 µm-resolution pixel grid of the class map

## Limitations

- Only binary (tumor vs. normal) classification is demonstrated; the correlation ranking is against a two-class prediction map, and multi-class extension is left to future development
- Ground truth is based solely on pathologist-annotated H&E histology, an approximation that may not capture intra-tumor heterogeneity or tumor margins
- No software version numbers or hyperparameter values for the trained massNet model are reported in the article text (README lists Python 3.6.12, Keras 2.2.0, Tensorflow 1.8.0, h5py 2.7.1), which can hinder exact reproduction

## Evidence
