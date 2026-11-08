---
name: auc-feature-table-extraction-from-qc-runs
description: Use when TARDIS targeted peak integration of raw LC–MS files (e.g. MetaboLights
  MTBLS8735 or cohort runs) has completed and you need the per-target, per-run AUC
  data.frame and QC-averaged metric tibble as a processed feature table.
when_to_use_negative:
- Input is already a processed feature table or AUC matrix — peak integration and
  table extraction have already been done
- The analysis goal is untargeted peak finding, which TARDIS does not perform (TARDIS
  targets targeted peak extraction/integration)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: TARDIS
  role: R package performing targeted peak integration and returning the results list
    holding the AUC data.frame and QC-metrics tibble
  repo: https://github.com/UGent-LIMET/TARDIS
- name: R
  role: Runtime environment (>= 4.4.0) in which TARDIS runs and in which the AUC table
    is exported and PCA (prcomp) is performed
  repo: https://cloud.r-project.org/index.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:dbee4ddf43230b2e4ba2ad0dce0d286b5b97a6cac7abbdfecb6594cf878c6859
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# AUC feature table extraction from QC runs

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts the processed AUC feature table (area under the curve per target per run) from TARDIS targeted peak-integration results on LC–MS data, together with the companion tibble of average per-target metrics computed over QC runs, ready for downstream PCA or other exploratory analysis.

## When to use

Use when TARDIS targeted peak integration of raw LC–MS files (e.g. MetaboLights MTBLS8735 or cohort runs) has completed and you need the per-target, per-run AUC data.frame and QC-averaged metric tibble as a processed feature table.

## When NOT to use

- Input is already a processed feature table or AUC matrix — peak integration and table extraction have already been done
- The analysis goal is untargeted peak finding, which TARDIS does not perform (TARDIS targets targeted peak extraction/integration)

## Inputs

- TARDIS `results` list object from targeted peak integration of raw LC–MS files (e.g. vendor formats from MetaboLights study MTBLS8735)
- Target list distributed with the TARDIS case_study vignette

## Outputs

- data.frame of AUC per target per run (processed AUC table)
- tibble with average quality metrics per target in QC runs
- PCA score plot with samples colored by run type

## How to apply

After running TARDIS targeted peak integration over raw LC–MS vendor files with a target list, access the returned `results` object, which is an R `list`: extract the `data.frame` containing the AUC of each target in each run and export it as the processed AUC table. The same list also contains a `tibble` holding a feature table with the average metrics (AUC, maximum intensity, points over the peak, custom signal-to-noise ratio, bell-curve similarity) for each target in the QC runs. In the case_study vignette workflow, this AUC matrix is then fed directly into exploratory PCA (e.g. `prcomp`) with samples colored by run type to confirm data quality. Correctness is judged by QC runs forming a tightly clustered group in the PCA score plot while sample runs show variance.

## Related tools

- **TARDIS** (R package performing targeted peak integration and returning the results list holding the AUC data.frame and QC-metrics tibble) — https://github.com/UGent-LIMET/TARDIS
- **R** (Runtime environment (>= 4.4.0) in which TARDIS runs and in which the AUC table is exported and PCA (prcomp) is performed) — https://cloud.r-project.org/index.html

## Examples

```
library(TARDIS); auc_df <- results$data_frame; write.csv(auc_df, 'auc_table.csv'); pca <- prcomp(t(auc_df))
```

## Evaluation signals

- The results object is an R list whose data.frame contains one AUC value per target per run
- The companion tibble contains average metrics per target restricted to QC runs
- PCA (prcomp) on the AUC matrix shows QC runs tightly clustered together while sample runs show variance
- All five quality metrics (AUC, maximum intensity, points over the peak, custom signal-to-noise ratio, bell-curve similarity) are populated in the QC feature table
- Exported processed AUC table matches the number of runs in the raw LC–MS file set (e.g. vignette case study files)

## Limitations

- Connection timeouts are common during installation/usage because the package contains example data; increase the R timeout setting (options(timeout = '300'))
- Extraction presumes successful targeted integration; peak quality must be checked via the metrics and EIC diagnostic plots saved in the output folder
- No TARDIS package version number is stated in the article; only R >= 4.4.0 and BiocManager 3.20 requirements are documented

## Evidence

- [results] The `results` object is a `list` that contains a `data.frame` with the AUC of each target in each run: "The `results` object is a `list` that contains a `data.frame` with the AUC of each target in each run"
- [results] QC runs tibble: "a `tibble` that contains a feature table with the average metrics for each target in the QC runs."
- [readme] Vignettes available: [evidence span withheld — anchor status: external_doc]
- [abstract] Quality metrics included: "Both established (area under the curve, maximum intensity and points over the peak) and recently developed (custom signal-to-noise ratio and bell-curve similarity) quality metrics were included to"
- [other] PCA QC clustering finding: [evidence span withheld — anchor status: not_found]
