---
name: noise-ion-score-filtering
description: Use when preprocessing a spatial metabolomics peak table matrix (pixels
  x ions with X/Y coordinates) to remove randomly distributed noise ions that would
  interfere with spatial pattern recognition or downstream clustering.
when_to_use_negative:
- The input is already a filtered or feature-selected table where noise ions have
  been removed by another QC step
- The goal is to remove background pixels or evaluate background consistency rather
  than filter spatially random ions — SMQVP provides separate background-consistency
  and background-noise-removal modules
- The data are not spatially resolved (no per-pixel X/Y coordinates), so a quadrat
  test on spatial intensity patterns is undefined
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SMQVP
  role: Web application providing the Noise Ion Detection module that identifies and
    filters noise ions using the noise-score approach on uploaded peak table matrices
  repo: https://github.com/mzlab-research/SMQVP
- name: R
  role: Environment for running the spatstat quadrat test per ion and computing -log10(p)
    noise scores on the peak table
- name: spatstat
  role: R package supplying the quadrat test for Complete Spatial Randomness applied
    to each ion's spatial intensity pattern across pixels
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo15060354
    title: 'SMQVP: A Web Application for Spatial Metabolomics Quality Visualization
      and Processing'
  source_content_hash: sha256:2d1511ee0340e73e59243259632de1b8d01b45e9b1160614733dc343479c233e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SMQVP
  tools:
  - R
  - SMQVP
  - spatstat
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# noise-ion-score-filtering

## Summary

Score each spatial metabolomics ion by how strongly it deviates from Complete Spatial Randomness (noise score = -log10 of the spatstat quadrat test p-value) and filter ions with scores below a chosen threshold (e.g. 90), as implemented in SMQVP for AFADESI mouse brain data.

## When to use

Use when preprocessing a spatial metabolomics peak table matrix (pixels x ions with X/Y coordinates) to remove randomly distributed noise ions that would interfere with spatial pattern recognition or downstream clustering.

## When NOT to use

- The input is already a filtered or feature-selected table where noise ions have been removed by another QC step
- The goal is to remove background pixels or evaluate background consistency rather than filter spatially random ions — SMQVP provides separate background-consistency and background-noise-removal modules
- The data are not spatially resolved (no per-pixel X/Y coordinates), so a quadrat test on spatial intensity patterns is undefined

## Inputs

- Peak table matrix with first two columns X/Y pixel coordinates and subsequent columns of ion m/z values with intensities (e.g. 53,812 pixels x 2,654 ions, background-removed AFADESI mouse brain data)
- Noise-score threshold (e.g. 90)

## Outputs

- noise_scores.csv: all ion m/z values with their CSR-based noise scores (-log10 quadrat test p-value)
- noise_filtered_peak_table.csv: peak intensity table restricted to ions with noise score >= threshold

## How to apply

Load the background-removed peak table matrix (e.g. the AFADESI mouse brain table of 53,812 pixels x 2,654 ions with X/Y coordinates in the first two columns, from deposit OMIX009541). For each ion, run the quadrat test from the R spatstat package against Complete Spatial Randomness (CSR) on its spatial intensity pattern across pixels, then compute the ion's noise score as the negative base-10 logarithm of the quadrat test p-value and export all m/z values with their scores (noise_scores.csv). Choose a noise-score threshold (SMQVP's demonstrated value is 90), remove ions scoring below it, and save the retained ions' intensity table (noise_filtered_peak_table.csv). The rationale is that spatially informative ions are non-randomly distributed across tissue, whereas randomly distributed ions reflect noise; at threshold 90 this filtering retained 37% of the original 2,654 ions while improving clustering agreement with tissue morphology.

## Related tools

- **SMQVP** (Web application providing the Noise Ion Detection module that identifies and filters noise ions using the noise-score approach on uploaded peak table matrices) — https://github.com/mzlab-research/SMQVP
- **R** (Environment for running the spatstat quadrat test per ion and computing -log10(p) noise scores on the peak table)
- **spatstat** (R package supplying the quadrat test for Complete Spatial Randomness applied to each ion's spatial intensity pattern across pixels)

## Examples

```
Rscript -e 'library(spatstat); p <- quadrat.test(ppp(X, Y, marks=ion_int)); score <- -log10(p$p.value)'
```

## Evaluation signals

- Retained-ion fraction at threshold 90 should be ~37% of the original 2,654 ions (accept ~35-39%)
- Every removed ion must have a noise score strictly below the chosen threshold; every retained ion must have score >= threshold
- noise_scores.csv must contain an entry for all 2,654 original ion m/z values
- noise_filtered_peak_table.csv row count must equal the original 53,812 pixels with X/Y columns intact, column count equal to retained ions
- Downstream clustering on filtered data should align better with tissue morphology than on unprocessed data

## Limitations

- The demonstrated threshold of 90 and 37% retention are specific to the AFADESI mouse brain dataset; other datasets and thresholds require re-validation
- No software version or changelog is specified for SMQVP, and no license information is stated beyond citation guidance
- Quadrat-test scores depend on the pixel grid; ions randomly distributed only outside the scanned region or sparse sparse ions may be mis-scored

## Evidence

- [readme] Noise Ion Filtering: Detect and filter out noise ions that may interfere with spatial pattern recognition.: "Noise Ion Filtering: Detect and filter out noise ions that may interfere with spatial pattern recognition."
- [results] noise score threshold 90 retains 37% of ions: [evidence span withheld — anchor status: not_found]
- [readme] input data format: "The input for SMQVP is a peak table matrix with the following structure: First two columns: X and Y coordinates of the pixels"
- [abstract] eight quality visualization points: "SMQVP incorporates eight comprehensive quality visualization and evaluation modules, including background consistency assessments, noise ion filtering, intensity distribution analyses, and the"
- [results] clustering results: "SMQVP has been demonstrated to effectively improve clustering results by removing noise signals and enhancing data quality."
