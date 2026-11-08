---
name: pearson-correlation-analysis-of-ion-intensities
description: Use when you need to verify background-region signal stability in a spatial
  metabolomics peak table, e.g. before background noise removal, by checking that
  pairwise Pearson correlations of region spectra exceed 0.9.
when_to_use_negative:
- The input is not a pixel-coordinate-indexed peak table but already aggregated spectra
  or a feature table without X/Y coordinates
- The goal is quality visualization of tissue regions (e.g. intensity distribution,
  isotopic/adduct identification) rather than background consistency evaluation
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3292
tools:
- name: SMQVP
  role: Web application implementing background consistency evaluation (QVP2) with
    lasso-based region delineation on the interaction image; Pearson correlation above
    0.9 flags consistent acquisition
  repo: https://github.com/mzlab-research/SMQVP
- name: R
  role: R environment (with Shiny/SMQVP) used to load the peak table and compute pairwise
    Pearson correlations of background-region spectra
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo15060354
    title: 'SMQVP: A Web Application for Spatial Metabolomics Quality Visualization
      and Processing'
  source_content_hash: sha256:e0ea6ea023246f14b1aa83907a6f5f752b016f18abb349b907fc8751a4bffe3f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SMQVP
  tools:
  - R
  - SMQVP
  - SMQVP v1.0
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# pearson-correlation-analysis-of-ion-intensities

## Summary

Quantifies the consistency of spatial-metabolomics background regions by computing pairwise Pearson correlation coefficients between representative spectra (median ion intensities per region), applying the SMQVP QVP2 criterion that coefficients above 0.9 indicate stable signal acquisition.

## When to use

Use when you need to verify background-region signal stability in a spatial metabolomics peak table, e.g. before background noise removal, by checking that pairwise Pearson correlations of region spectra exceed 0.9.

## When NOT to use

- The input is not a pixel-coordinate-indexed peak table but already aggregated spectra or a feature table without X/Y coordinates
- The goal is quality visualization of tissue regions (e.g. intensity distribution, isotopic/adduct identification) rather than background consistency evaluation

## Inputs

- Peak table matrix (pixels x m/z ion intensities) with first two columns = X/Y pixel coordinates, e.g. CSV from OMIX009541
- Definition of background regions (groupings of low-intensity pixels outside the tissue)

## Outputs

- Pairwise Pearson correlation matrix between background-region representative spectra (background_correlations.csv)
- Per-region representative spectra (median ion intensities across pixels)

## How to apply

Load the peak table (e.g. 53,812 pixels x 2,654 ion peaks from OMIX009541) with the first two columns as X/Y pixel coordinates and remaining columns as m/z ion intensities. Delineate background regions by grouping low-intensity pixels outside the tissue, mirroring the SMQVP lasso-based region selection. For each background region, compute the median intensity of each ion across its pixels to build a representative spectrum. Compute pairwise Pearson correlation coefficients of ion peak intensities between all background-region spectra; per the QVP2 criterion, a coefficient above 0.9 indicates consistent, stable acquisition. Write the resulting pairwise correlation matrix to a file such as background_correlations.csv and inspect it against the validation threshold (all pairs above 0.99 in the paper's AFADESI mouse brain result).

## Related tools

- **SMQVP** (Web application implementing background consistency evaluation (QVP2) with lasso-based region delineation on the interaction image; Pearson correlation above 0.9 flags consistent acquisition) — https://github.com/mzlab-research/SMQVP
- **R** (R environment (with Shiny/SMQVP) used to load the peak table and compute pairwise Pearson correlations of background-region spectra)

## Examples

```
cor(med <- sapply(bg_regions, function(idx) apply(peaktab[idx, -(1:2)], 2, median)))  # pairwise Pearson correlations of background-region median spectra
```

## Evaluation signals

- All pairwise background-region Pearson correlations exceed 0.9 (QVP2 consistency criterion)
- In the AFADESI mouse brain benchmark, all reported background-region correlations exceed 0.99, matching the paper's reported result
- Correlation matrix is complete and symmetric over all background-region pairs with unit diagonal
- Representative spectra are built from median ion intensities across each region's pixels, not means of raw pixels

## Limitations

- Correlation reflects acquisition stability only; high correlation between background regions does not itself validate tissue-region signal quality
- Region delineation depends on manual/visual grouping of low-intensity pixels (lasso tool in SMQVP), so results can vary with region choice

## Evidence

- [results] All selected background regions in the AFADESI mouse brain dataset showed pairwise Pearson correlation coefficients consistently above 0.99, indicating stable signal acquisition during data collection.: [evidence span withheld — anchor status: not_found]
- [full_text] a coefficient exceeding 0.9 indicates consistent, stable acquisition: [evidence span withheld — anchor status: not_found]
- [readme] Use the lasso tool to delineate background and tissue regions on the interaction image: "Use the lasso tool to delineate background and tissue regions on the interaction image"
- [readme] First two columns: X and Y coordinates of the pixels; Subsequent columns: m/z values with corresponding intensities at each pixel: "First two columns: X and Y coordinates of the pixels"
