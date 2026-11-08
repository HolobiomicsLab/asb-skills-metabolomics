---
name: median-spectrum-computation-per-region
description: Use when you have a spatial metabolomics peak table (pixels × m/z ions)
  with manually or lasso-delineated regions and need one representative spectrum per
  region before computing region-pair metrics like Pearson correlation.
when_to_use_negative:
- Input is already aggregated at region or sample level with no per-pixel peak table,
  so no per-pixel median can be computed
- The goal is pixel-level analysis (e.g., clustering or spatial pattern recognition),
  where collapsing pixels to a single median spectrum would destroy spatial resolution
- Regions have not been delineated and no pixel-coordinate or intensity criteria exist
  to define which pixels belong to background vs. tissue
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SMQVP
  role: Web application (R/Shiny) in which lasso-delineated regions are defined and
    quality visualization points (spectra examination, background consistency) are
    run on the region-median spectra
  repo: https://github.com/mzlab-research/SMQVP
- name: R
  role: Environment used to load the peak table, group pixels by region, and compute
    per-ion median intensities for each region's representative spectrum
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo15060354
    title: 'SMQVP: A Web Application for Spatial Metabolomics Quality Visualization
      and Processing'
  source_content_hash: sha256:254a60915ff3c10351d6ae21f8e1ca14a47d87df67d792b4fc44e90179bc1598
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SMQVP
  tools:
  - R
  - SMQVP
  - SMQVP v1.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# median-spectrum-computation-per-region

## Summary

Computes a single representative spectrum for each delineated spatial region (background or tissue) by taking the per-ion median intensity across all pixels in the region, enabling region-level comparisons such as background-consistency Pearson correlations in the SMQVP workflow.

## When to use

Use when you have a spatial metabolomics peak table (pixels × m/z ions) with manually or lasso-delineated regions and need one representative spectrum per region before computing region-pair metrics like Pearson correlation.

## When NOT to use

- Input is already aggregated at region or sample level with no per-pixel peak table, so no per-pixel median can be computed
- The goal is pixel-level analysis (e.g., clustering or spatial pattern recognition), where collapsing pixels to a single median spectrum would destroy spatial resolution
- Regions have not been delineated and no pixel-coordinate or intensity criteria exist to define which pixels belong to background vs. tissue

## Inputs

- Peak table matrix: first two columns X/Y pixel coordinates, remaining columns m/z ion intensities (e.g., 53,812 pixels × 2,654 ion peaks)
- Region definitions: lasso-delineated background/tissue regions (from SMQVP interaction image) or pixel-index groupings for low-intensity pixels outside tissue

## Outputs

- One representative median spectrum per region (vector of per-ion median intensities)
- Downstream artifacts such as a pairwise Pearson correlation matrix between region spectra (e.g., background_correlations.csv)

## How to apply

Load the peak table (e.g., the AFADESI mouse brain table of 53,812 pixels × 2,654 ion peaks from OMIX009541), treating the first two columns as X/Y pixel coordinates and remaining columns as m/z ion intensities. Delineate regions — in SMQVP via the lasso tool on the interaction image, or programmatically by grouping low-intensity pixels outside the tissue for background regions. For each region, compute the median intensity of each ion across all pixels in the region to build a representative spectrum, which is robust to outlier pixels and noise. Feed the resulting region spectra into downstream quality visualization points, e.g., pairwise Pearson correlation between background-region spectra where a coefficient above 0.9 (SMQVP QVP2 criterion) indicates consistent, stable acquisition. Write results such as the pairwise correlation matrix to CSV for inspection against the paper's reported values (all background-region correlations above 0.99).

## Related tools

- **SMQVP** (Web application (R/Shiny) in which lasso-delineated regions are defined and quality visualization points (spectra examination, background consistency) are run on the region-median spectra) — https://github.com/mzlab-research/SMQVP
- **R** (Environment used to load the peak table, group pixels by region, and compute per-ion median intensities for each region's representative spectrum)

## Examples

```
spectrum_bg1 <- apply(peak_table[pixels_bg1, -(1:2)], 2, median); cor(spectrum_bg1, spectrum_bg2, method="pearson")
```

## Evaluation signals

- All pairwise Pearson correlation coefficients between background-region median spectra exceed 0.9 (QVP2 criterion), and in the AFADESI mouse brain dataset they should exceed 0.99
- Each output spectrum has exactly one median value per ion column (2,654 values for the OMIX009541 example table)
- The set of pixels aggregated per region matches the lasso-defined or low-intensity pixel grouping, with no pixels assigned to multiple regions
- The correlation matrix output is symmetric with diagonal values of 1 and matches the values written to background_correlations.csv

## Limitations

- Requires correct region delineation; mis-assigned pixels (e.g., tissue pixels included in a background region) distort the median spectrum and downstream correlations
- Median spectra are region-level summaries and cannot capture spatial heterogeneity within a region
- No software version or license information is stated for SMQVP in the available documentation

## Evidence

- [readme] Use the lasso tool to delineate background and tissue regions on the interaction image: "Use the lasso tool to delineate background and tissue regions on the interaction image"
- [readme] The input for SMQVP is a peak table matrix with the following structure: "The input for SMQVP is a peak table matrix with the following structure:
- First two columns: X and Y coordinates of the pixels
- Subsequent columns: m/z values with corresponding intensities at each"
- [intro] through eight quality visualization points, including background consistency evaluation: "through eight quality visualization points, including background consistency evaluation, noise ion filtering, intensity distribution analysis, and identification of isotopic and adduct ions"
- [results] SMQVP has been demonstrated to effectively improve clustering results by removing noise signals: "SMQVP has been demonstrated to effectively improve clustering results by removing noise signals and enhancing data quality."
