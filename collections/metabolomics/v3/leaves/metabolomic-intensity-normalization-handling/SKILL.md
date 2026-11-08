---
name: metabolomic-intensity-normalization-handling
description: Use when a post-formula-assignment FT-ICR MS peak table (e.g., Report.csv
  with peak abundances and molecular formulas) must be normalized across samples before
  multivariate statistics, and the normalization method itself must be selected empirically
  via test_normalization.
when_to_use_negative:
- The input is raw FT-ICR MS spectra that still needs molecular formula assignment
  — MetaboDirect requires formula assignment to be performed prior to use
- The peak table was already normalized upstream (e.g., the S. fallax data set was
  previously filtered/processed by the original authors), so re-normalization would
  be redundant or inappropriate
- The goal is cheminformatics on structural/MS2 data (e.g., molecular networking of
  isomer-resolved spectra) rather than direct-injection FT-ICR MS peak-intensity tables
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetaboDirect
  role: Pipeline whose pre-processing step normalizes peak intensities using the method
    chosen by test_normalization
  repo: https://github.com/Coayala/MetaboDirect
- name: pmartR
  role: R package used by test_normalization to run the normalization-method tests
- name: pandas
  role: Loads Report.csv and metadata.csv and builds the peak-intensity DataFrame
    for filtering and normalization
- name: Python
  role: Language in which MetaboDirect and its normalization handling are implemented
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:79dd5e2e6242310e070ccbc82831b9b1f4ce1844fbce93d57f2f2d5ab310d81b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Coayala/MetaboDirect
  tools:
  - Matplotlib
  - MetaboDirect
  - NumPy
  - Python
  - pandas
  - pmartR
  - scikit-learn
  - seaborn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# metabolomic-intensity-normalization-handling

## Summary

Handles the intensity-normalization stage of MetaboDirect: selecting the best normalization method with the test_normalization companion script, then applying it to FT-ICR MS peak-intensity tables so downstream dissimilarity, PERMANOVA, NMDS/PCA, and chemodiversity analyses operate on comparable per-sample abundances.

## When to use

Use when a post-formula-assignment FT-ICR MS peak table (e.g., Report.csv with peak abundances and molecular formulas) must be normalized across samples before multivariate statistics, and the normalization method itself must be selected empirically via test_normalization.

## When NOT to use

- The input is raw FT-ICR MS spectra that still needs molecular formula assignment — MetaboDirect requires formula assignment to be performed prior to use
- The peak table was already normalized upstream (e.g., the S. fallax data set was previously filtered/processed by the original authors), so re-normalization would be redundant or inappropriate
- The goal is cheminformatics on structural/MS2 data (e.g., molecular networking of isomer-resolved spectra) rather than direct-injection FT-ICR MS peak-intensity tables

## Inputs

- Report.csv: FT-ICR MS peak abundance table with assigned molecular formulas (direct-injection, post formula assignment)
- metadata.csv: sample metadata defining sample groups/treatments
- User-specified normalization method preference (selected via the test_normalization companion script)

## Outputs

- .csv table of filtered peaks with normalized and unnormalized intensities
- test_normalization diagnostic output ranking candidate normalization methods
- Normalized peak-intensity matrix feeding downstream dissimilarity matrices and PERMANOVA .csv statistics tables

## How to apply

First run the standalone test_normalization companion script (which relies on the R package pmartR) on the raw peak-abundance table to rank candidate normalization methods before launching the main pipeline. Then, in the MetaboDirect pre-processing step, load the Report.csv peak table and metadata.csv with pandas, filter peaks by m/z range, removal of isotopic 13C-containing formulas, formula-assignment error <= 0.5 ppm, and minimum sample presence, and normalize the retained peak intensities across samples using the method selected in the test step. Export the filtered peak table with both normalized and unnormalized intensities as .csv outputs in the pipeline directory. Rationale: normalization choice directly affects downstream Bray-Curtis/Euclidean dissimilarity matrices and the PERMANOVA grouping test, so it must be validated empirically (as done for the bacterium-phage data set, where the normalized table still supported a significant phage-treatment effect at p < 0.05 despite subtle ordination separation).

## Related tools

- **MetaboDirect** (Pipeline whose pre-processing step normalizes peak intensities using the method chosen by test_normalization) — https://github.com/Coayala/MetaboDirect
- **pmartR** (R package used by test_normalization to run the normalization-method tests)
- **pandas** (Loads Report.csv and metadata.csv and builds the peak-intensity DataFrame for filtering and normalization)
- **Python** (Language in which MetaboDirect and its normalization handling are implemented)

## Evaluation signals

- test_normalization was run before the main pipeline and its ranked recommendation is documented before a method is applied
- The filtered peak table .csv is generated, readable, and contains both normalized and unnormalized intensity columns
- Isotopic 13C-containing masses are removed during the same pre-processing pass (~200 masses removed in the bacterium-phage example)
- Downstream Bray-Curtis/Euclidean/Jaccard dissimilarity matrices computed on normalized intensities reproduce the reported PERMANOVA significance (p < 0.05) for the phage-treatment grouping
- Re-running the pipeline with a different normalization method produces visibly different NMDS/PCA ordinations, confirming normalization choice propagates to multivariate outputs

## Limitations

- Direct-injection MS cannot separate chemical isomers and suffers signal suppression or enhancement, which can confound downstream data analysis regardless of normalization
- Normalization quality depends on prior peak filtering; in the S. fallax data set no peaks were filtered because that data set was already filtered by the original authors
- The full stack requires Python >= 3.5, R >= 4 (with pmartR, vegan, SYNCSA, etc.), and Cytoscape >= 3.8 for some downstream steps

## Evidence

- [intro] test_normalization selection precedes pipeline: [evidence span withheld — anchor status: not_found]
- [results] first applied step in the example workflow: "the first step was to use the “test_normalization” companion script to help determine which normalization method worked the best"
- [readme] pmartR powers the normalization tests: "pmartR (for normalization tests)"
- [abstract] Python stack underlying the pre-processing/normalization step: [evidence span withheld — anchor status: not_found]
