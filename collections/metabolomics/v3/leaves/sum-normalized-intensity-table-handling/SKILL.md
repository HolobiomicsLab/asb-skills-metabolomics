---
name: sum-normalized-intensity-table-handling
description: Use when raw FT-ICR MS peak abundances with assigned molecular formulas
  must be normalized before chemodiversity or statistical analysis, and the normalization
  method itself must be chosen empirically rather than fixed a priori.
when_to_use_negative:
- The pipeline accepts peak abundance and assigned molecular formula data produced
  after an initial processing of raw FT-ICR MS spectra, so do not feed raw spectra
  lacking formula assignments — Rao's Q requires elemental-composition trait vectors
- The input table has already been normalized by an upstream workflow; re-running
  sum normalization could distort relative abundances
- Data are peak lists without sample-level grouping metadata, since box plots of diversity
  indices are grouped by the user-defined grouping variable
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0092
tools:
- name: MetaboDirect
  role: Command-line pipeline whose test_normalization companion script and chemodiversity
    step handle the sum-normalized intensity table and downstream diversity computation
  repo: https://github.com/Coayala/MetaboDirect
- name: pmartR
  role: R library used by MetaboDirect's normalization tests to select the best normalization
    method
- name: SPANS / test_normalization
  role: Companion script run before the main pipeline to determine which normalization
    method worked best
- name: vegan
  role: R library supporting the diversity index computation in the chemodiversity
    step
- name: pandas
  role: Writes the per-sample chemodiversity results as .csv files
- name: seaborn
  role: Renders the box plots of diversity indices grouped by the user's grouping
    variable
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:df4ac725d535fed85aa8c1b23e2292a113ad3c480be94910f493168c996cc0cb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Coayala/MetaboDirect
  tools:
  - Matplotlib
  - MetaboDirect
  - NumPy
  - Python
  - SPANS / test_normalization
  - pandas
  - pmartR
  - seaborn
  - vegan
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# sum-normalized-intensity-table-handling

## Summary

Prepares and applies the sum-normalized FT-ICR MS peak intensity table that MetaboDirect uses for chemodiversity analysis: the test_normalization (SPANS) companion script selects the best normalization method, and the sum-normalized abundances feed Shannon, Gini-Simpson, Chao1, and Rao's quadratic entropy calculations.

## When to use

Use when raw FT-ICR MS peak abundances with assigned molecular formulas must be normalized before chemodiversity or statistical analysis, and the normalization method itself must be chosen empirically rather than fixed a priori.

## When NOT to use

- The pipeline accepts peak abundance and assigned molecular formula data produced after an initial processing of raw FT-ICR MS spectra, so do not feed raw spectra lacking formula assignments — Rao's Q requires elemental-composition trait vectors
- The input table has already been normalized by an upstream workflow; re-running sum normalization could distort relative abundances
- Data are peak lists without sample-level grouping metadata, since box plots of diversity indices are grouped by the user-defined grouping variable

## Inputs

- FT-ICR MS peak abundance table (peak intensities per sample with assigned molecular formulas) from post-processing of raw spectra
- Sample metadata with the user-defined grouping variable (e.g., phage treatment or incubation condition)
- Normalization-method comparison output from the test_normalization/SPANS companion script

## Outputs

- Sum-normalized intensity table used downstream by chemodiversity and multivariate statistics
- Per-sample chemodiversity .csv tables (Shannon, Gini-Simpson, Chao1, Rao's Q) written with pandas
- Box plot figures of each diversity index grouped by the user's grouping variable (Matplotlib/seaborn)

## How to apply

Before the main pipeline, run the test_normalization companion script (SPANS, backed by the R pmartR library) on the peak abundance table to determine which normalization method works best, since the article reports this was the first step in both case studies. Apply the selected method so each sample's peak intensities are sum-normalized to a common total, keeping the table keyed to masses with assigned molecular formulas after filtering by m/z, 13C isotopic peaks, 0.5 ppm formula assignment error, and a user-defined minimum sample threshold. Use this normalized table, not raw intensities, as the input to chemodiversity: compute Shannon, Gini-Simpson, and Chao1 as abundance-based indices, and derive molecular trait vectors from elemental composition (elemental ratios, double-bond equivalents/AI_mod aromaticity, reactivity proxies) to compute Rao's quadratic entropy as functional diversity (R SYNCSA/vegan). Assemble per-sample diversity values into .csv tables with pandas and render box plots grouped by the user's grouping variable with Matplotlib/seaborn. Validate that every metric is computed for every sample and that the .csv outputs contain the expected per-sample values and grouping columns.

## Related tools

- **MetaboDirect** (Command-line pipeline whose test_normalization companion script and chemodiversity step handle the sum-normalized intensity table and downstream diversity computation) — https://github.com/Coayala/MetaboDirect
- **pmartR** (R library used by MetaboDirect's normalization tests to select the best normalization method)
- **SPANS / test_normalization** (Companion script run before the main pipeline to determine which normalization method worked best)
- **vegan** (R library supporting the diversity index computation in the chemodiversity step)
- **pandas** (Writes the per-sample chemodiversity results as .csv files)
- **seaborn** (Renders the box plots of diversity indices grouped by the user's grouping variable)

## Examples

```
pip install metabodirect && metabodirect -h
```

## Evaluation signals

- After sum normalization, each sample column's intensities sum to the same constant total across samples
- The chemodiversity .csv tables contain a value for every diversity metric (Shannon, Gini-Simpson, Chao1, Rao's Q) for every sample, plus the grouping columns
- Chao1 richness is at least as large as the observed number of molecular formulas per sample
- Abundance-based indices (Shannon/Gini-Simpson/Chao1) are clearly distinguished from the functional index (Rao's Q) in the outputs, and box plots render per grouping variable without error
- Isotopic 13C peaks are removed before normalization (e.g., ~200 masses filtered out from the bacterium-phage data set)

## Limitations

- Direct injection FT-ICR MS cannot separate chemical isomers and suffers signal suppression or enhancement that can confound downstream data analysis of the intensity table
- Normalization method choice materially affects results; the article explicitly runs test_normalization/SPANS first rather than assuming a default
- Upstream filtering matters: no peaks were filtered in the S. fallax data set because it was previously filtered by the original authors, so the normalized table inherits whatever filtering was done beforehand

## Evidence

- [intro] test_normalization companion script: [evidence span withheld — anchor status: not_found]
- [intro] pipeline input scope: "The pipeline accepts peak abundance and assigned molecular formula data produced after an initial processing of raw FT-ICR MS spectra"
- [intro] chemodiversity pipeline step: "The MetaboDirect pipeline consists of 6 major steps/categories (Fig. 1): (i) data pre-processing, (ii) data diagnostics, (iii) data exploration, (iv) chemodiversity analysis"
- [results] case-study normalization step: [evidence span withheld — anchor status: not_found]
- [results] isotopic filtering example: "~200 masses were filtered out from the data set because they were assigned molecular formulas containing an isotopic carbon"
- [abstract] software stack: [evidence span withheld — anchor status: not_found]
- [readme] R dependency list: "MetaboDirect requires Python (3.5 and above), R (4 and above) and Cytoscape (3.8 and above) with the following libraries/modules:"
