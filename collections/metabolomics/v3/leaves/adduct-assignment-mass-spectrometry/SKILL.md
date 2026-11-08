---
name: adduct-assignment-mass-spectrometry
description: Use when a set of tumor-correlated m/z peaks (Pearson correlation ≥ 0.7
  with the tumor-class prediction map) from MALDI FT-ICR MSI data needs tentative
  molecular formula and adduct annotation before downstream interpretation or table
  export.
when_to_use_negative:
- Peaks whose spatial pattern has Pearson correlation < 0.7 with the tumor-class prediction
  map — these are not considered highly co-localized tumor-associated features and
  would dilute the annotation table.
- Input is already an annotated feature table with formula/adduct assignments (e.g.
  from a prior HMDB/METLIN search) — re-annotation would duplicate effort.
- Data acquired without prior peak identification from the VAE module (e.g. raw spectra
  before msiPL peak identification) — annotation is applied downstream of peak selection,
  not to raw spectral complexity.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: massNet
  role: VAE whose decoder identifies the m/z peaks and produces the tumor-class prediction
    map used as the selection basis for annotation
  repo: github.com/wabdelmoula/massNet
- name: msiPL
  role: Method by which peaks are identified from the VAE latent representation before
    adduct assignment
  repo: github.com/wabdelmoula/massNet
- name: HMDB
  role: Database searched for tentative formula/adduct assignment (e.g. C9H13N2O9P
    [M+2K-H]+ at -2.1 ppm)
- name: METLIN
  role: Database searched for tentative formula/adduct assignment (e.g. C10H16N5O13P3
    [M+K]+ at 0.36 ppm)
- name: Python
  role: Implemented in python (Keras/tensorflow backend) for computing correlations
    and exporting the ranked annotation table
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:a6dcb743b51bfcb51315bbe784c930ca28a576b87e548e859db4dc406ea7d4dd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/wabdelmoula/massNet
  tools:
  - HMDB
  - METLIN
  - Python
  - massNet
  - msiPL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# adduct-assignment-mass-spectrometry

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates m/z peaks selected from the massNet/msiPL VAE decoder by tentative molecular formula and adduct state (e.g. [M+K]+, [M+2K-H]+) via HMDB and METLIN searches, recording the ppm error for each assignment. This links tumor-correlated spatial features to putative molecular identities for biological interpretation.

## When to use

Use when a set of tumor-correlated m/z peaks (Pearson correlation ≥ 0.7 with the tumor-class prediction map) from MALDI FT-ICR MSI data needs tentative molecular formula and adduct annotation before downstream interpretation or table export.

## When NOT to use

- Peaks whose spatial pattern has Pearson correlation < 0.7 with the tumor-class prediction map — these are not considered highly co-localized tumor-associated features and would dilute the annotation table.
- Input is already an annotated feature table with formula/adduct assignments (e.g. from a prior HMDB/METLIN search) — re-annotation would duplicate effort.
- Data acquired without prior peak identification from the VAE module (e.g. raw spectra before msiPL peak identification) — annotation is applied downstream of peak selection, not to raw spectral complexity.

## Inputs

- Ranked list of tumor-correlated m/z peaks (Pearson correlation ≥ 0.7) from the massNet VAE decoder on the 85,062 m/z-variable MALDI FT-ICR MSI dataset
- Tumor-class prediction map derived from the 5-dimensional massNet Code layer latent variable
- HMDB and METLIN database search access
- Measured exact m/z values from positive-ion MALDI FT-ICR MSI (HDF5-format data)

## Outputs

- tumor_correlated_features.tsv with columns (m/z, correlation, formula, adduct, database, ppm error)
- Tentative molecular formula and adduct annotations (e.g. [M+K]+, [M+2K-H]+) per m/z peak
- Table 2-style assignment list reporting database source and ppm error

## How to apply

Start from the m/z peaks identified by the msiPL VAE decoder that achieved Pearson correlation coefficient ≥ 0.7 against the tumor-class prediction map computed from the 5-dimensional Code layer. For each retained peak, search the exact m/z against the HMDB and METLIN databases in positive ion mode, testing potassium-related adducts such as [M+K]+ and [M+2K-H]+ typical of the 9.4 Tesla SolariX FT-ICR acquisition. Record the tentative formula, adduct, database source, and the ppm error for each assignment (e.g. C9H13N2O9P [M+2K-H]+ at 400.9554553 via HMDB with -2.1 ppm; C10H16N5O13P3 [M+K]+ at 545.9589 via Metlin with 0.36 ppm). Rank annotated features by descending Pearson correlation and export the table (m/z, correlation, formula, adduct, database, ppm error) as tumor_correlated_features.tsv. Assignments should be labeled tentative since database search alone does not confirm identity.

## Related tools

- **massNet** (VAE whose decoder identifies the m/z peaks and produces the tumor-class prediction map used as the selection basis for annotation) — github.com/wabdelmoula/massNet
- **msiPL** (Method by which peaks are identified from the VAE latent representation before adduct assignment) — github.com/wabdelmoula/massNet
- **HMDB** (Database searched for tentative formula/adduct assignment (e.g. C9H13N2O9P [M+2K-H]+ at -2.1 ppm))
- **METLIN** (Database searched for tentative formula/adduct assignment (e.g. C10H16N5O13P3 [M+K]+ at 0.36 ppm))
- **Python** (Implemented in python (Keras/tensorflow backend) for computing correlations and exporting the ranked annotation table)

## Examples

```
df[df.correlation >= 0.7].sort_values('correlation', ascending=False).to_csv('tumor_correlated_features.tsv', sep='\t')
```

## Evaluation signals

- Every exported entry in tumor_correlated_features.tsv has Pearson correlation ≥ 0.7.
- Assignments carry reported ppm errors within small tolerance — reproduce Table 2-style values such as -2.1 ppm (C9H13N2O9P [M+2K-H]+, HMDB) and 0.36 ppm (C10H16N5O13P3 [M+K]+, Metlin).
- The top-ranked peak is m/z 400.9546 ± 0.01, elevated in the tumor region.
- Each row records the database source (HMDB or Metlin) and adduct state; rows missing adduct or ppm columns indicate incomplete annotation.

## Limitations

- Assignments are tentative database matches only; the paper reports these as tentative formula/adduct labels without confirmatory validation.
- The workflow depends on ground truth annotation based solely on annotated histology, which is an approximation approach that might not be optimal especially in case of intra-tumor heterogeneity.
- The demonstrated scope is binary normal-vs-tumor classification on GBM PDX MALDI FT-ICR MSI; the paper leaves multi-class classification and other tissue/tumor types as future work, limiting transferability of the annotated feature set.

## Evidence

- [methods] co-localized peak threshold: "The highly co-localized m/z peaks are those that achieved the highest Pearson correlation coefficient (T ≥ 0.7)"
- [methods] msiPL peak identification: "Pearson correlation was then applied on each of these spatial patterns and the identified peaks from the VAE module using the msiPL method (Abdelmoula et al., 2020)"
- [discussion] HMDB assignment example: "C9H13N2O9P [M+2K-H]+ 400.9554553 HMDB -2.1"
- [discussion] METLIN assignment example: "C10H16N5O13P3 [M+K]+ 545.9589 Metlin 0.36"
- [readme] implementation environment: "Packages: numpy(1.15.4), sklearn(0.23.2), scipy(1.0.0), seaborn (0.9.0), Pandas(1.1.1.), and h5py(2.7.1)"
