---
name: ppm-error-calculation-for-mz-assignment
description: Use when high-resolution (FT-ICR) MSI peaks already filtered for tumor
  co-localization must be annotated with candidate formulas/adducts from HMDB/METLIN,
  with ppm errors recorded per assignment.
when_to_use_negative:
- When the m/z list has not yet been co-localization filtered — annotating all 85,062
  raw m/z variables would produce mostly spurious assignments; filter at Pearson T
  ≥ 0.7 first
- When the data come from a low-mass-accuracy instrument where ppm-level formula assignment
  is not meaningful (this skill is grounded in 9.4 Tesla FT-ICR data)
- When the goal is peak picking or tissue classification itself — those are handled
  upstream by massNet/msiPL without prior preprocessing or peak picking
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: massNet
  role: Provides the trained VAE whose decoder reconstructs per-m/z spatial images
    and whose 5-dimensional Code-layer latent representation identifies the peaks
    to annotate
  repo: https://github.com/wabdelmoula/massNet
- name: msiPL
  role: Upstream method from which the tumor-correlated m/z peaks emerge via Pearson
    correlation of VAE-decoder spatial patterns (T ≥ 0.7)
  repo: https://github.com/wabdelmoula/massNet
- name: HMDB
  role: Database searched to assign tentative formula/adduct and report ppm error,
    e.g. C9H13N2O9P [M+2K-H]+ at −2.1 ppm
- name: METLIN
  role: Database searched to assign tentative formula/adduct and report ppm error,
    e.g. C10H16N5O13P3 [M+K]+ at 0.36 ppm
- name: h5py
  role: Python package (v2.7.1 in the reference environment) used to read the HDF5-converted
    MSI data containing the m/z variables being annotated
  repo: https://github.com/wabdelmoula/massNet
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:8e1ae0ffdcb35a3835439ff7c45fb8797ddb95f57988b933e11b550d3dfc6998
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
  - h5py
  - massNet
  - msiPL
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# ppm-error-calculation-for-mz-assignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns tentative molecular formulas and adducts ([M+K]+, [M+2K-H]+) to tumor-co-localized m/z peaks by searching the HMDB and METLIN databases, and quantifies assignment quality as a ppm error between observed and theoretical exact masses, as in massNet's Table 2.

## When to use

Use when high-resolution (FT-ICR) MSI peaks already filtered for tumor co-localization must be annotated with candidate formulas/adducts from HMDB/METLIN, with ppm errors recorded per assignment.

## When NOT to use

- When the m/z list has not yet been co-localization filtered — annotating all 85,062 raw m/z variables would produce mostly spurious assignments; filter at Pearson T ≥ 0.7 first
- When the data come from a low-mass-accuracy instrument where ppm-level formula assignment is not meaningful (this skill is grounded in 9.4 Tesla FT-ICR data)
- When the goal is peak picking or tissue classification itself — those are handled upstream by massNet/msiPL without prior preprocessing or peak picking

## Inputs

- HDF5-format MALDI 9.4T SolariX FT-ICR MSI dataset (85,062 m/z variables, 100 µm pixels) from GBM PDX mouse brain
- List of tumor-co-localized m/z peaks (Pearson correlation ≥ 0.7 with the massNet tumor-class prediction map)
- HMDB metabolite database
- METLIN metabolite database

## Outputs

- Ranked annotation table (m/z, Pearson correlation, tentative molecular formula, adduct, database, ppm error), e.g. tumor_correlated_features.tsv
- Per-peak ppm error values (e.g. −2.1 ppm for C9H13N2O9P [M+2K-H]+; 0.36 ppm for C10H16N5O13P3 [M+K]+)

## How to apply

First restrict annotation to highly co-localized m/z peaks, i.e. those whose spatial images reconstructed by the massNet VAE decoder reach Pearson correlation ≥ 0.7 with the tumor-class prediction map (T ≥ 0.7). For each retained peak, search its exact m/z against HMDB and METLIN, allowing common positive-mode MALDI adducts such as [M+K]+ and [M+2K-H]+ that the paper reports for GBM PDX data. Compute the mass error as ppm = (m/z_observed − m/z_theoretical)/m/z_theoretical × 1e6, recording the ppm error next to each formula/adduct/database triple. Rank the annotated features by descending Pearson correlation and export the table (m/z, correlation, formula, adduct, database, ppm error), e.g. as tumor_correlated_features.tsv. Correct application reproduces Table 2-style rows such as C9H13N2O9P [M+2K-H]+ at −2.1 ppm (HMDB) and C10H16N5O13P3 [M+K]+ at 0.36 ppm (Metlin) for the 400.9546 ± 0.01 tumor peak.

## Related tools

- **massNet** (Provides the trained VAE whose decoder reconstructs per-m/z spatial images and whose 5-dimensional Code-layer latent representation identifies the peaks to annotate) — https://github.com/wabdelmoula/massNet
- **msiPL** (Upstream method from which the tumor-correlated m/z peaks emerge via Pearson correlation of VAE-decoder spatial patterns (T ≥ 0.7)) — https://github.com/wabdelmoula/massNet
- **HMDB** (Database searched to assign tentative formula/adduct and report ppm error, e.g. C9H13N2O9P [M+2K-H]+ at −2.1 ppm)
- **METLIN** (Database searched to assign tentative formula/adduct and report ppm error, e.g. C10H16N5O13P3 [M+K]+ at 0.36 ppm)
- **h5py** (Python package (v2.7.1 in the reference environment) used to read the HDF5-converted MSI data containing the m/z variables being annotated) — https://github.com/wabdelmoula/massNet

## Examples

```
ppm = (400.9546 - 400.9554553) / 400.9554553 * 1e6  # -> ~ -2.1 ppm, matching the Table 2 HMDB entry
```

## Evaluation signals

- Every exported row has Pearson correlation ≥ 0.7 against the tumor-class prediction map
- Reproduced Table 2-style assignments include both HMDB and METLIN hits with reported ppm errors (e.g. C9H13N2O9P [M+2K-H]+ at −2.1 ppm; C10H16N5O13P3 [M+K]+ at 0.36 ppm)
- The top-ranked peak is m/z 400.9546 ± 0.01, elevated in the tumor region
- Each ppm error is consistent with recomputing (m/z_obs − m/z_theo)/m/z_theo × 1e6 from the reported theoretical mass (e.g. 400.9546 vs 400.9554553)
- Output table schema contains the required columns: m/z, correlation, formula, adduct, database, ppm error

## Limitations

- Assignments are tentative: annotation of the MSI data relies on the ground truth derived from annotated histology, which the authors note is an approximation and may not capture intra-tumor heterogeneity
- The pipeline was demonstrated only for binary tumor/normal classification of GBM PDX brain tissue; generalization to other tissues and tumor types is untested
- The ppm errors are only meaningful because of the high mass accuracy of the 9.4 Tesla SolariX FT-ICR instrument; lower-resolution instruments will not reproduce sub-ppm/low-ppm assignments

## Evidence
