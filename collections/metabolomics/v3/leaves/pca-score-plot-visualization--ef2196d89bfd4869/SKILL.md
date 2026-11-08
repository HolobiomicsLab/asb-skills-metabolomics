---
name: pca-score-plot-visualization
description: Use when a targeted LC–MS preprocessing run (e.g., TARDIS AUC table)
  needs a global quality check, and you want to confirm QC runs cluster tightly before
  downstream statistics.
when_to_use_negative:
- Input is already a fully preprocessed feature table from untargeted tools such as
  xcms, MZmine or OpenMS rather than TARDIS targeted AUC output.
- The goal is formal peak-quality filtering (e.g., via the TARDIS quality metrics
  or machine learning classification) rather than an exploratory QC visualization.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: TARDIS
  role: R package that performs targeted peak integration over raw LC–MS files and
    returns the AUC-per-target-per-run data.frame fed into the PCA
  repo: https://github.com/UGent-LIMET/TARDIS
- name: R
  role: Environment in which prcomp is run for exploratory PCA and the score plot
    is drawn
  repo: https://cloud.r-project.org/index.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:c66e618feb27469d9f1702c6ba8fb2c44df367b06daf6b01e081441a780aaea0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/UGent-LIMET/TARDIS
  repo_url: https://github.com/UGent-LIMET/TARDIS
  tools:
  - R
  - TARDIS
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# pca-score-plot-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Exploratory PCA on the target-by-run AUC table produced by TARDIS targeted integration, with a score plot colored by run type, used as a QC sanity check that QC injections cluster tightly while biological samples show variance.

## When to use

Use when a targeted LC–MS preprocessing run (e.g., TARDIS AUC table) needs a global quality check, and you want to confirm QC runs cluster tightly before downstream statistics.

## When NOT to use

- Input is already a fully preprocessed feature table from untargeted tools such as xcms, MZmine or OpenMS rather than TARDIS targeted AUC output.
- The goal is formal peak-quality filtering (e.g., via the TARDIS quality metrics or machine learning classification) rather than an exploratory QC visualization.

## Inputs

- data.frame of AUC per target per run extracted from the TARDIS results list
- run-type metadata (QC vs sample labels) for coloring the score plot

## Outputs

- PCA model object (e.g., prcomp result) on the integrated AUC matrix
- PCA score plot with samples colored by run type
- visual confirmation that QC runs cluster tightly

## How to apply

After running TARDIS targeted peak integration over the raw LC–MS files, extract the data.frame containing the AUC of each target in each run from the results list (the list also holds a tibble of average metrics per target in QC runs). Transpose/shape the AUC table into a features (targets) x samples (runs) matrix, apply an exploratory PCA with prcomp in R, and draw the score plot with points colored by run type (QC vs sample). For the MTBLS8735 case study, TARDIS was applied to public data from different LC–MS vendors using the target list from the case_study vignette. Judge success by whether QC runs form a tightly clustered group while sample runs show variance, which is the expected pattern for high-quality data sets.

## Related tools

- **TARDIS** (R package that performs targeted peak integration over raw LC–MS files and returns the AUC-per-target-per-run data.frame fed into the PCA) — https://github.com/UGent-LIMET/TARDIS
- **R** (Environment in which prcomp is run for exploratory PCA and the score plot is drawn) — https://cloud.r-project.org/index.html

## Examples

```
results <- TARDIS::integrate_targets(raw_files, target_list); auc <- results$auc; pca <- prcomp(t(auc), center = TRUE, scale. = TRUE); plot(pca$x, col = run_type)
```

## Evaluation signals

- QC runs form a tightly clustered group in the PCA score plot, as expected for high-quality data sets.
- Sample runs show variance in the score plot while QC points stay compact.
- The AUC matrix passed to prcomp covers every target in every run, matching the data.frame in the TARDIS results list.
- The score plot clearly distinguishes run type (QC vs sample) via point color.

## Limitations

- PCA quality clustering is exploratory; it does not replace TARDIS quality metrics (AUC, maximum intensity, points over the peak, custom signal-to-noise ratio, bell-curve similarity) for peak-level assessment.
- Connection timeouts can occur when installing TARDIS with its example data; increase the R timeout option (e.g., 300 seconds).

## Evidence

- [results] Exploratory PCA of integrated target AUCs showed data points of QC runs tightly clustered together while sample runs showed variance, as expected for high-quality data sets.: [evidence span withheld — anchor status: not_found]
- [results] Can TARDIS be readily applied to public data (MTBLS8735) originating from different LC–MS vendors: [evidence span withheld — anchor status: not_found]
- [readme] Read the vignettes for a tutorial on how to use TARDIS: [evidence span withheld — anchor status: external_doc]
