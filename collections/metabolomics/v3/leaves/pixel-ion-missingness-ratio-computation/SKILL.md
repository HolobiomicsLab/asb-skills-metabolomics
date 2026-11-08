---
name: pixel-ion-missingness-ratio-computation
description: Use when a preprocessed peak table matrix (X/Y coordinate columns plus
  m/z intensity columns) must be quality-checked for missing values at pixel and ion
  levels before or after SMQVP preprocessing.
when_to_use_negative:
- Input is raw mass-spectrometry imaging data (e.g. mzML/imzML) rather than a preprocessed
  peak table matrix with pixel-coordinate columns
- The analysis goal is background/noise filtering or isotopic/adduct identification
  (other SMQVP QVP modules) rather than missingness quantification
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: SMQVP
  role: Provides the QVP6 missing value distribution module whose pixel- and ion-level
    missingness assessment this computation reproduces on the AFADESI mouse brain
    dataset
  repo: https://github.com/mzlab-research/SMQVP
- name: R
  role: Runtime environment (with Shiny) in which SMQVP's quality visualization, including
    missing value distribution, is implemented
  repo: https://github.com/mzlab-research/SMQVP
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo15060354
    title: 'SMQVP: A Web Application for Spatial Metabolomics Quality Visualization
      and Processing'
  source_content_hash: sha256:132d8b389d19a82c88e9b813600d460f7a3e5e2e61e6ad57e28a059614ff0881
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SMQVP
  tools:
  - R
  - SMQVP
  - SMQVP v1.0
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# pixel-ion-missingness-ratio-computation

## Summary

Computes per-pixel and per-ion missing-value ratios from an SMQVP QVP6 analysis of a spatial metabolomics peak table matrix (e.g. 53,812 pixels x 2,654 ions from AFADESI mouse brain data), quantifying what fraction of ions go undetected in each pixel and what fraction of pixels lack each ion.

## When to use

Use when a preprocessed peak table matrix (X/Y coordinate columns plus m/z intensity columns) must be quality-checked for missing values at pixel and ion levels before or after SMQVP preprocessing.

## When NOT to use

- Input is raw mass-spectrometry imaging data (e.g. mzML/imzML) rather than a preprocessed peak table matrix with pixel-coordinate columns
- The analysis goal is background/noise filtering or isotopic/adduct identification (other SMQVP QVP modules) rather than missingness quantification

## Inputs

- Peak table matrix from OMIX009541 (53,812 pixels x 2,654 ions) with first two columns holding pixel X/Y coordinates and subsequent columns holding m/z intensities (CSV/TSV loaded via pandas)

## Outputs

- Histogram/spatial distribution plot of missing pixel ratios (PNG, Figure 2H style)
- Frequency distribution plot of missing ion ratios (PNG, Figure 2I style)
- missing_ratios.tsv containing the per-pixel and per-ion missing ratio vectors

## How to apply

Load the peak table matrix with pandas, where the first two columns are pixel X/Y coordinates and remaining columns are m/z features. Compute the missing pixel ratio as the percentage of ions undetected in each pixel, and the missing ion ratio as the percentage of pixels in which each ion is undetected. Visualize both ratio vectors as histogram/frequency distributions (Figure 2H,I style) with matplotlib, and optionally map missing pixel ratios spatially via the X/Y coordinates to inspect tissue-edge effects. Rationale: SMQVP's QVP6 missing value distribution module flags noise and unreliable detection, and in the AFADESI mouse brain validation most pixels and ions showed missingness below 5%, with slightly higher proportions at tissue edges.

## Related tools

- **SMQVP** (Provides the QVP6 missing value distribution module whose pixel- and ion-level missingness assessment this computation reproduces on the AFADESI mouse brain dataset) — https://github.com/mzlab-research/SMQVP
- **R** (Runtime environment (with Shiny) in which SMQVP's quality visualization, including missing value distribution, is implemented) — https://github.com/mzlab-research/SMQVP

## Examples

```
import pandas as pd; df = pd.read_csv('peak_table.tsv', sep='\t'); ions = df.columns[2:]; pixel_ratio = df[ions].isna().mean(axis=1)*100; ion_ratio = df[ions].isna().mean()*100
```

## Evaluation signals

- Output PNG figures for both the missing pixel ratio histogram and missing ion ratio frequency distribution exist and are non-empty
- The majority of computed pixel-level and ion-level missing ratios fall below 5%, matching the paper's reported distribution
- The per-pixel ratio vector has exactly 53,812 entries and the per-ion vector exactly 2,654 entries for the OMIX009541 mouse brain table
- Pixels mapped spatially via the X/Y coordinate columns show only slightly elevated missing ratios at tissue edges, consistent with the paper
- missing_ratios.tsv contains two named ratio columns with values in [0, 100]% and no NaNs

## Limitations

- The skill is validated only on AFAD-ESI positive-ion-mode mouse brain data; other acquisition platforms or ion modes may show different missingness profiles
- Repository lacks changelog and explicit version/license information, so reproducing a specific SMQVP release may be non-trivial
- High missingness at tissue edges can be conflated with true biological absence if spatial position is ignored

## Evidence

- [intro] Perform quality visualization points including spectra examination, background consistency, ion expression proportion, noise ion detection, intensity distribution, missing value distribution, isotopic peak ratio, and adduct ion ratio: [evidence span withheld — anchor status: not_found]
- [readme] Missing Value Distribution: Assess and visualize missing values at both pixel and ion levels.: "Missing Value Distribution: Assess and visualize missing values at both pixel and ion levels."
- [readme] The data includes 53,812 pixels and 2,654 ion peaks. This peak table and raw data are available in the public repository https://ngdc.cncb.ac.cn/omix/release/OMIX009541: "The data includes 53,812 pixels and 2,654 ion peaks. This peak table and raw data are available in the public repository https://ngdc.cncb.ac.cn/omix/release/OMIX009541"
