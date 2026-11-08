---
name: pixel-coordinate-spatial-statistics
description: Use when you have a peak table matrix with X/Y pixel coordinates (e.g.,
  53,812 pixels x 2,654 ions from AFADESI imaging) and need to detect and remove randomly
  distributed (noise) ions before spatial pattern recognition or clustering.
when_to_use_negative:
- Input data lacks pixel X/Y coordinates (e.g., bulk LC-MS feature tables) — the quadrat
  test on spatial intensity patterns cannot be computed
- The goal is removing background pixels rather than noise ions — SMQVP's Background
  Noise Removal module is the appropriate step
- Very few pixels per ion or already cluster-validated ion lists — the CSR test is
  designed to flag randomly distributed signals and may not apply
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: spatstat
  role: R package used to run the quadrat test for Complete Spatial Randomness on
    each ion's spatial intensity pattern across pixels
- name: SMQVP
  role: Web application whose Noise Ion Filtering module implements this detection/filtering
    workflow; source code available on GitHub
  repo: github.com/mzlab-research/SMQVP
- name: R
  role: Environment in which the peak table is loaded and the per-ion quadrat test
    and noise-score computation are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo15060354
    title: 'SMQVP: A Web Application for Spatial Metabolomics Quality Visualization
      and Processing'
  source_content_hash: sha256:986c351232f21d5a63bf129fce40da37841ab5f277ffc06af437c00a380de19f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SMQVP
  - spatstat
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# pixel-coordinate-spatial-statistics

## Summary

A noise-ion filtering method for spatial metabolomics that scores each ion's spatial pattern across pixels using the spatstat quadrat test for Complete Spatial Randomness (CSR), converts the p-value to a noise score (-log10), and removes ions with scores below a threshold (90 in the SMQVP mouse-brain example).

## When to use

Use when you have a peak table matrix with X/Y pixel coordinates (e.g., 53,812 pixels x 2,654 ions from AFADESI imaging) and need to detect and remove randomly distributed (noise) ions before spatial pattern recognition or clustering.

## When NOT to use

- Input data lacks pixel X/Y coordinates (e.g., bulk LC-MS feature tables) — the quadrat test on spatial intensity patterns cannot be computed
- The goal is removing background pixels rather than noise ions — SMQVP's Background Noise Removal module is the appropriate step
- Very few pixels per ion or already cluster-validated ion lists — the CSR test is designed to flag randomly distributed signals and may not apply

## Inputs

- AFADESI-based mouse brain peak table matrix from deposit OMIX009541 (53,812 pixels x 2,654 ions, background-removed)
- Peak table format with X and Y pixel coordinates in the first two columns, followed by m/z intensity columns

## Outputs

- noise_scores.csv: all ion m/z values with their noise scores (-log10 of the quadrat test p-value)
- noise_filtered_peak_table.csv: peak intensity table containing only ions with noise scores >= 90
- Retained-ion proportion relative to the original 2,654 ions (expected ~37%)

## How to apply

Load the background-removed peak table matrix from the AFADESI mouse brain deposit OMIX009541 (53,812 pixels x 2,654 ions, with X/Y coordinates in the first two columns) in R. For each ion, run the spatstat quadrat test for Complete Spatial Randomness (CSR) on its spatial intensity pattern across pixels. Compute each ion's noise score as the negative base-10 logarithm of the quadrat test p-value and write all ion m/z values with their noise scores to noise_scores.csv. Filter ions using the reported threshold of 90 — remove ions with noise scores below 90 — and report the retained proportion relative to the original 2,654 ions. Save the retained ions' peak intensity table to noise_filtered_peak_table.csv. Rationale: ions failing CSR (random spatial distribution) are noise that interferes with spatial pattern recognition, as implemented in the SMQVP noise-ion-filtering module.

## Related tools

- **spatstat** (R package used to run the quadrat test for Complete Spatial Randomness on each ion's spatial intensity pattern across pixels)
- **SMQVP** (Web application whose Noise Ion Filtering module implements this detection/filtering workflow; source code available on GitHub) — github.com/mzlab-research/SMQVP
- **R** (Environment in which the peak table is loaded and the per-ion quadrat test and noise-score computation are executed)

## Examples

```
library(spatstat); q <- quadrat.test(as.ppp(xy, W)); noise_score <- -log10(q$p.value)
```

## Evaluation signals

- Retained fraction equals the reported 37% of the original 2,654 ions (accepting a small tolerance, e.g., 35-39%)
- Every removed ion has a noise score below 90 in noise_scores.csv
- noise_scores.csv contains all 2,654 original ion m/z values, each with a finite -log10(p-value) score
- noise_filtered_peak_table.csv preserves the original pixel count (53,812 rows) and X/Y coordinates of the input matrix
- Filtered dataset yields improved clustering that aligns better with tissue morphology compared to unprocessed data

## Limitations

- The noise-score threshold of 90 was calibrated on the AFADESI mouse brain dataset (2,654 ions) and may require re-tuning for other datasets or platforms
- No software version specified for SMQVP and no changelog available for the repository, complicating reproducibility
- Detailed input data requirements are deferred to the tutorial panel within SMQVP rather than fully documented

## Evidence

- [other] quadrat test scoring: [evidence span withheld — anchor status: not_found]
- [other] threshold and retention result: [evidence span withheld — anchor status: not_found]
- [readme] SMQVP noise filtering feature: "Noise Ion Filtering: Detect and filter out noise ions that may interfere with spatial pattern recognition."
- [readme] input format description: "The input for SMQVP is a peak table matrix with the following structure:
- First two columns: X and Y coordinates of the pixels"
