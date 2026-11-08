---
name: quality-metric-distribution-visualization
description: Use when a spatial metabolomics peak table matrix (pixels x m/z ions,
  e.g. 53,812 pixels x 2,654 ions from AFADESI mouse brain data) must be assessed
  for missing-value burden prior to noise filtering or clustering.
when_to_use_negative:
- Input is raw mass-spectrometry imaging data (e.g. mzML/imzML) rather than a peak
  table matrix with X/Y coordinates and ion intensity columns
- The goal is to filter noise ions or remove background pixels (use SMQVP's Noise
  Ion Filtering or Background Noise Removal modules instead of the missing-value visualization
  module)
- Data have already been verified to meet the <5% missing-value criterion and the
  next step is clustering or tissue-morphology comparison
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_0091
tools:
- name: SMQVP v1.0
  role: Web application providing the Missing Value Distribution (QVP6) module that
    assesses and visualizes missing values at both pixel and ion levels
  repo: https://github.com/mzlab-research/SMQVP
- name: R
  role: Runtime environment (with Shiny) in which SMQVP is implemented; alternative
    environment for reproducing the missing-ratio computation
  repo: https://github.com/mzlab-research/SMQVP
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo15060354
    title: 'SMQVP: A Web Application for Spatial Metabolomics Quality Visualization
      and Processing'
  source_content_hash: sha256:646b99f834e0908c63d10e469c62f8c0cce925dcdf2d75288b7a0d58bbed5271
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SMQVP
  tools:
  - R
  - SMQVP
  - SMQVP v1.0
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# quality-metric-distribution-visualization

## Summary

Quantifies and visualizes the distribution of missing values in a spatial metabolomics peak table at both pixel and ion levels, as implemented in SMQVP's Missing Value Distribution module (QVP6), to judge data quality before and after preprocessing.

## When to use

Use when a spatial metabolomics peak table matrix (pixels x m/z ions, e.g. 53,812 pixels x 2,654 ions from AFADESI mouse brain data) must be assessed for missing-value burden prior to noise filtering or clustering.

## When NOT to use

- Input is raw mass-spectrometry imaging data (e.g. mzML/imzML) rather than a peak table matrix with X/Y coordinates and ion intensity columns
- The goal is to filter noise ions or remove background pixels (use SMQVP's Noise Ion Filtering or Background Noise Removal modules instead of the missing-value visualization module)
- Data have already been verified to meet the <5% missing-value criterion and the next step is clustering or tissue-morphology comparison

## Inputs

- Peak table matrix CSV/TSV from OMIX009541: 53,812 pixels (rows) x 2,654 ions (columns), first two columns = X/Y pixel coordinates (AFADESI mouse brain, positive ion mode)
- SMQVP-preprocessed peak table (post background-pixel removal and noise-ion filtering)

## Outputs

- missing_ratios.tsv with per-pixel and per-ion missing-value percentage vectors
- Histogram/spatial plot of missing pixel ratios (Figure 2H style, PNG)
- Frequency distribution plot of missing ion ratios (Figure 2I style, PNG)

## How to apply

Load the preprocessed AFADESI mouse brain peak table matrix (53,812 pixels x 2,654 ions; first two columns hold X/Y pixel coordinates) from the OMIX009541 deposit using pandas. Compute, for each pixel, the percentage of ions undetected in that pixel (missing pixel ratio), and for each ion, the percentage of pixels in which that ion was undetected (missing ion ratio). Reproduce SMQVP's QVP6-style Figure 2H,I plots: a histogram/spatial distribution of missing pixel ratios and a frequency distribution of missing ion ratios, using matplotlib, writing both ratio vectors to missing_ratios.tsv. Interpretation follows the paper's benchmark: the majority of pixel-level and ion-level missing ratios should fall below 5%, with only slightly elevated missingness at tissue edges; deviations indicate preprocessing problems or misassigned background/tissue regions (delineated with the SMQVP lasso tool).

## Related tools

- **SMQVP v1.0** (Web application providing the Missing Value Distribution (QVP6) module that assesses and visualizes missing values at both pixel and ion levels) — https://github.com/mzlab-research/SMQVP
- **R** (Runtime environment (with Shiny) in which SMQVP is implemented; alternative environment for reproducing the missing-ratio computation) — https://github.com/mzlab-research/SMQVP

## Examples

```
import pandas as pd; df = pd.read_csv('peak_table.tsv', sep='\t'); ions = df.iloc[:, 2:]; pixel_ratio = ions.isna().mean(axis=1)*100; ion_ratio = ions.isna().mean(axis=0)*100
```

## Evaluation signals

- Output figure exists as PNG files for both the pixel-level (Figure 2H style) and ion-level (Figure 2I style) distributions
- Majority of computed per-pixel and per-ion missing ratios fall below 5%, matching the paper's reported distribution
- missing_ratios.tsv contains two ratio vectors whose lengths equal 53,812 (pixels) and 2,654 (ions) respectively
- Elevated missing ratios, if present, are confined to tissue edges rather than randomly distributed across the tissue section

## Limitations

- No software version specified for SMQVP in the article's discussion, complicating exact reproducibility
- No license information stated for the SMQVP repository
- No changelog available for the repository
- The <5% benchmark was established on the AFADESI mouse brain dataset; other tissue types or acquisition platforms may show different acceptable missingness profiles

## Evidence

- [readme] Missing Value Distribution: Assess missing values across different spatial regions and ions.: "Missing Value Distribution: Assess missing values across different spatial regions and ions."
- [readme] The example data used in this study is spatial metabolomics data of a mouse brain acquired using the AFAD-ESI platform in positive ion mode. The data includes 53,812 pixels and 2,654 ion peaks.: "The example data used in this study is spatial metabolomics data of a mouse brain acquired using the AFAD-ESI platform in positive ion mode. The data includes 53,812 pixels and 2,654 ion peaks."
- [results] Most pixels and ions in the preprocessed dataset exhibited missing values below 5%, with only slightly higher missing proportions at tissue edges.: [evidence span withheld — anchor status: not_found]
- [intro] through eight quality visualization points, including background consistency evaluation, noise ion filtering, intensity distribution analysis, and identification of isotopic and adduct ions: "through eight quality visualization points, including background consistency evaluation, noise ion filtering, intensity distribution analysis, and identification of isotopic and adduct ions"
