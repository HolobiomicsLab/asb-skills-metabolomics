---
name: large-scale-feature-matrix-processing
description: Use when a large-scale cohort metabolomics experiment (e.g. over 1,000
  human plasma samples run over 44 days on an Agilent 1260 Infinity HPLC with Qtrap5500)
  yields a raw abundance matrix that must be filtered, drift-corrected and made complete
  before hierarchical RUV-III correction.
when_to_use_negative:
- The input is already a complete, drift-corrected matrix with batch annotations—this
  skill would duplicate the filtering and imputation steps described in the hRUV workflow
- The study is a small single-batch metabolomics experiment where pooled-QC-based
  drift correction and inter-batch replicates are unnecessary; the method was designed
  for multi-batch cohort-scale studies
- The data matrix is derived from untargeted spectral features rather than targeted
  metabolite abundances, since the 50% detection filter and replicate-embedding design
  were validated on targeted LC-MS/MS panels
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: hRUV
  role: R package that packages the filtering, imputation (via DMwR2/impute) and drift-correction
    routines preceding its hierarchical RUV-III correction; also exposes the plateDesign
    function to generate replicate layouts
  repo: github.com/SydneyBioX/hRUV
- name: hRUV Shiny application
  role: Web interface for designing the embedded replicate arrangements (pooled QC,
    short replicates, batch replicates) used in this pre-processing step
- name: MultiQuant (SCIEX)
  role: Source of the raw integrated abundance values that feed into this processing
    pipeline
- name: Agilent 1260 Infinity HPLC System
  role: Instrument generating the chromatographic separation underlying the metabolite
    abundance matrix
- name: Qtrap5500 (Sciex)
  role: Mass spectrometer that determined m/z for the metabolite features being filtered
    and corrected
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:22e89da98cbdd84ef2fb6991f6f674337efa582edf226da826d920b64581befd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Agilent 1260 Infinity HPLC System
  - MultiQuant (SCIEX)
  - Qtrap5500 (Sciex)
  - R
  - hRUV
  - hRUV Shiny application
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# large-scale-feature-matrix-processing

## Summary

This skill handles the pre-processing of large-scale targeted metabolomics feature matrices (samples × metabolites) so they are clean, drift-corrected, and batch-annotated before inter-batch unwanted-variation removal. It covers filtering metabolites with >50% missingness, removing outlying QC measurements, imputing residual missing values (hRUV requires complete matrices), and organising batch/replicate design information from 96-well plate layouts.

## When to use

Use when a large-scale cohort metabolomics experiment (e.g. over 1,000 human plasma samples run over 44 days on an Agilent 1260 Infinity HPLC with Qtrap5500) yields a raw abundance matrix that must be filtered, drift-corrected and made complete before hierarchical RUV-III correction.

## When NOT to use

- The input is already a complete, drift-corrected matrix with batch annotations—this skill would duplicate the filtering and imputation steps described in the hRUV workflow
- The study is a small single-batch metabolomics experiment where pooled-QC-based drift correction and inter-batch replicates are unnecessary; the method was designed for multi-batch cohort-scale studies
- The data matrix is derived from untargeted spectral features rather than targeted metabolite abundances, since the 50% detection filter and replicate-embedding design were validated on targeted LC-MS/MS panels

## Inputs

- Integrated peak-abundance table (samples × metabolites) exported from MultiQuant (SCIEX) peak integration
- Plate layout / run-order annotation listing 96-well plate position, run index, and batch assignment for each sample
- Replicate design metadata identifying pooled QC wells, short (within-row) replicate pairs, and inter-batch replicate pairs

## Outputs

- Filtered, drift-corrected metabolite abundance matrix with no missing values, samples in rows and metabolites in columns
- Per-plate replicate design map (Supplementary Table 1 style) recording positions of pooled QCs, short replicates and batch replicates
- Batch label vector aligning each sample to its acquisition batch/day

## How to apply

Start from the integrated abundance table produced by peak integration in MultiQuant (SCIEX), keeping samples as rows and metabolites as columns. Filter out metabolites that were not present in at least 50% of the samples, then examine the three consecutive pooled QC samples embedded at the start of each 96-well plate and remove any outlying measurements they reveal. Because hRUV expects data without missing values—which is often not possible in large-scale metabolomics—impute any remaining missing entries (the package uses the DMwR2 and impute R libraries for this purpose). Record the batch structure and replicate design: pooled QC placed in the first three wells then after every 10 runs, short within-row replicates (one biological sample from the previous row replicated at a random position in each row), and batch replicates (5 random samples from the previous plate embedded after the first three pooled QCs). Signal drift is then corrected with robust linear or non-linear smoothers fitted to the pooled QC time-course before handing the complete, drift-corrected matrix to the downstream hierarchical RUV-III step.

## Related tools

- **hRUV** (R package that packages the filtering, imputation (via DMwR2/impute) and drift-correction routines preceding its hierarchical RUV-III correction; also exposes the plateDesign function to generate replicate layouts) — github.com/SydneyBioX/hRUV
- **hRUV Shiny application** (Web interface for designing the embedded replicate arrangements (pooled QC, short replicates, batch replicates) used in this pre-processing step)
- **MultiQuant (SCIEX)** (Source of the raw integrated abundance values that feed into this processing pipeline)
- **Agilent 1260 Infinity HPLC System** (Instrument generating the chromatographic separation underlying the metabolite abundance matrix)
- **Qtrap5500 (Sciex)** (Mass spectrometer that determined m/z for the metabolite features being filtered and corrected)

## Examples

```
Rscript -e 'library(hRUV); data(processed_data); design <- plateDesign(replicates=5, plate_size=96)'
```

## Evaluation signals

- After the 50% detection filter, no metabolite column should have missingness above 50% of samples
- The resulting matrix must contain zero NA/missing entries, since hRUV requires complete data
- The number of samples (rows) and metabolites (columns) must match the original acquisition dimensions after imputation, with no sample lost to filtering
- Drift-corrected pooled QC values should show reduced time-trend signal relative to the raw QC series, indicating the robust smoother captured the drift
- The replicate design file must correctly map each short replicate pair to the same biological sample across rows and each batch replicate pair to the same sample across consecutive plates

## Limitations

- By chance, the randomly selected batch replicates can come from only one half of the previous plate, limiting the ability to capture unwanted variability across the whole plate (observed between batches 6–7 and 13–14, with a slight residual shift in signal between those batches)
- hRUV expects data without missing values, which is often not possible in large-scale metabolomics data, forcing reliance on imputation whose accuracy can affect downstream correction
- Signals of metabolites depend on their individual chemical properties, making selection of appropriate negative-control metabolites for batch correction a challenge

## Evidence

- [discussion] Metabolite presence filter: "metabolites that were not present in at least 50% of the samples were filtered out"
- [discussion] Outlier removal from initial QC runs: "We examined the three consecutive QC samples embedded at the start of each plate and removed any outlying measurements."
- [discussion] Missing-data requirement for hRUV: "While the proposed hRUV algorithm expects data without missing values, this is often not possible in large-scale metabolomics data"
- [discussion] Pooled QC layout specification: "the first three wells were populated with three pooled QC samples, then a single pooled QC sample was repeated after every 10 runs"
- [discussion] Batch replicate embedding rule: "after the first three pooled QC samples, a random selection of 5 biological samples from the previous plate was repeated (batch replicate)"
- [discussion] Short replicate definition: "we randomly selected a single biological sample from the previous row to be replicated at random position in a row (short replicate)"
- [intro] Drift-correction method statement: "applying signal drift correction with robust linear or non-linear smoothers"
- [discussion] Instrument chain for abundance generation: "Metabolite abundance peaks were integrated using the area under the curve for calibrated peaks from MultiQuant (SCIEX)"
- [readme] hRUV dependency on imputation libraries: "### CRAN

dplyr
tidyr
rlang
ggplot2
plotly
S4Vectors
tibble
DMwR2
MASS"
- [readme] hRUV package scope statement: "`hRUV` is a package for normalisation of multiple batches of metabolomics data in a hierarchical strategy with use of samples replicates in large-scale studies."
