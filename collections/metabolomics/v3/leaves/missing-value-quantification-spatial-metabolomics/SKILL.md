---
name: missing-value-quantification-spatial-metabolomics
description: Use when a peak table matrix (pixels x m/z ion columns with X/Y coordinates)
  must be checked for missing/undetected signals before noise ion filtering or clustering,
  e.g. to validate a preprocessed AFADESI dataset's quality.
when_to_use_negative:
- Input is raw MS imaging data (e.g. imzML) rather than an already-built peak table
  matrix — SMQVP expects a pixel x ion intensity matrix with X/Y coordinate columns
- The analysis goal is to impute or fill missing values rather than quantify and visualize
  their distribution
- The question concerns background consistency, noise ion filtering, isotopic/adduct
  identification, or other SMQVP quality visualization points unrelated to missingness
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SMQVP
  role: Web application whose Missing Value Distribution quality visualization point
    assesses missing values at both pixel and ion levels; v1.0, also runnable from
    the GitHub source
  repo: github.com/mzlab-research/SMQVP
- name: R
  role: Runtime environment (with Shiny package) in which SMQVP is implemented and
    locally deployed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo15060354
    title: 'SMQVP: A Web Application for Spatial Metabolomics Quality Visualization
      and Processing'
  source_content_hash: sha256:53b8fc1a7d3eef913a2c8d899dbbf18137facade05f7b4383ea4d7588f7d261a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SMQVP
  - SMQVP v1.0
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# missing-value-quantification-spatial-metabolomics

## Summary

Quantify and visualize missing-value distributions at both pixel and ion levels in a spatial metabolomics peak table matrix, using SMQVP's Missing Value Distribution quality visualization point on AFADESI-acquired mouse brain data.

## When to use

Use when a peak table matrix (pixels x m/z ion columns with X/Y coordinates) must be checked for missing/undetected signals before noise ion filtering or clustering, e.g. to validate a preprocessed AFADESI dataset's quality.

## When NOT to use

- Input is raw MS imaging data (e.g. imzML) rather than an already-built peak table matrix — SMQVP expects a pixel x ion intensity matrix with X/Y coordinate columns
- The analysis goal is to impute or fill missing values rather than quantify and visualize their distribution
- The question concerns background consistency, noise ion filtering, isotopic/adduct identification, or other SMQVP quality visualization points unrelated to missingness

## Inputs

- SMQVP-format peak table matrix (pixels x ions) with first two columns as X/Y pixel coordinates, e.g. AFADESI mouse brain data from OMIX009541 (53,812 pixels x 2,654 ions)

## Outputs

- Histogram/spatial distribution plot of missing pixel ratios (PNG, Figure 2H style)
- Frequency distribution plot of missing ion ratios (PNG, Figure 2I style)
- missing_ratios.tsv containing per-pixel and per-ion missing ratio vectors

## How to apply

Load the SMQVP-formatted peak table matrix (first two columns X/Y coordinates, remaining columns ion m/z intensities) — e.g. the 53,812-pixel x 2,654-ion AFADESI mouse brain table from OMIX009541 — with pandas. Compute the missing pixel ratio for each pixel as the percentage of ions undetected in that pixel, and the missing ion ratio for each ion as the percentage of pixels in which that ion was undetected. Plot the histogram/spatial distribution of missing pixel ratios and the frequency distribution of missing ion ratios (Figure 2H,I style) with matplotlib, and write both ratio vectors to missing_ratios.tsv. Interpretation threshold from the paper: the majority of pixel-level and ion-level missing ratios should fall below 5%, with only slightly higher missing proportions expected at tissue edges.

## Related tools

- **SMQVP** (Web application whose Missing Value Distribution quality visualization point assesses missing values at both pixel and ion levels; v1.0, also runnable from the GitHub source) — github.com/mzlab-research/SMQVP
- **R** (Runtime environment (with Shiny package) in which SMQVP is implemented and locally deployed)

## Examples

```
import pandas as pd; df = pd.read_csv('OMIX009541_peak_table.tsv', sep='\t'); pixel_ratio = df.iloc[:,2:].isna().mean(axis=1)*100; ion_ratio = df.iloc[:,2:].isna().mean(axis=0)*100
```

## Evaluation signals

- Output PNG figures exist and show the missing pixel ratio distribution and missing ion ratio frequency distribution
- The majority of computed pixel-level missing ratios fall below 5%
- The majority of computed ion-level missing ratios fall below 5%
- Only slightly elevated missing proportions appear at tissue edges, matching the paper's reported distribution
- missing_ratios.tsv row/column counts match the input matrix dimensions (53,812 pixels; 2,654 ions for the example dataset)

## Limitations

- Requires a preprocessed peak table matrix in SMQVP input format; raw MS imaging files are not accepted directly
- Slightly higher missing proportions at tissue edges are expected and should not be mistaken for noise
- No software version or changelog details are available beyond 'SMQVP v1.0'; no license information is stated in the repository README

## Evidence

- [readme] Missing Value Distribution: Assess and visualize missing values at both pixel and ion levels.: "Missing Value Distribution: Assess and visualize missing values at both pixel and ion levels."
- [readme] The input for SMQVP is a peak table matrix with the following structure: First two columns: X and Y coordinates of the pixels; Subsequent columns: m/z values with corresponding intensities at each pixel: "The input for SMQVP is a peak table matrix with the following structure:
- First two columns: X and Y coordinates of the pixels
- Subsequent columns: m/z values with corresponding intensities at each"
- [readme] The example data used in this study is spatial metabolomics data of a mouse brain acquired using the AFAD-ESI platform in positive ion mode. The data includes 53,812 pixels and 2,654 ion peaks.: "The example data used in this study is spatial metabolomics data of a mouse brain acquired using the AFAD-ESI platform in positive ion mode. The data includes 53,812 pixels and 2,654 ion peaks."
- [intro] through eight quality visualization points, including background consistency evaluation, noise ion filtering, intensity distribution analysis, and identification of isotopic and adduct ions: "through eight quality visualization points, including background consistency evaluation, noise ion filtering, intensity distribution analysis, and identification of isotopic and adduct ions"
- [results] Most pixels and ions in the preprocessed dataset exhibited missing values below 5%, with only slightly higher missing proportions at tissue edges.: [evidence span withheld — anchor status: not_found]
