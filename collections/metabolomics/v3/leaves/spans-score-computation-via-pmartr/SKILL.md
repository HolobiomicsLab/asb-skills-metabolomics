---
name: spans-score-computation-via-pmartr
description: Decision support for choosing a normalization method for direct-injection
  FT-ICR MS peak abundance tables before launching the main MetaboDirect pipeline.
when_to_use_negative:
- The peak abundance table is already normalized or the user has already fixed a normalization
  method outside the SPANS evaluation
- Working with raw, unassigned FT-ICR MS spectra — MetaboDirect and its test_normalization
  script expect peak abundance and assigned molecular formula data produced after
  initial processing, not raw spectra
tools:
- name: MetaboDirect
  role: Provides the test_normalization companion script that wraps pmartR's spans_procedure
  repo: https://github.com/Coayala/MetaboDirect
- name: pmartR
  role: R package whose spans_procedure underlies the SPANS score computation
- name: pandas
  role: Collects the per-combination SPANS scores into a score table
- name: Matplotlib
  role: Renders the SPANS score heatmap
- name: seaborn
  role: Renders the SPANS score heatmap
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:7fe3301bee92496414ba13640820df77c02bb39af2ef7451afcb3db1d42d45a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Coayala/MetaboDirect
  tools:
  - Matplotlib
  - MetaboDirect
  - Python
  - pandas
  - pmartR
  - seaborn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# spans-score-computation-via-pmartr

## Summary

Computes SPANS scores by wrapping the pmartR spans_procedure in MetaboDirect's test_normalization companion script, evaluating combinations of candidate normalization methods (e.g., median, z score) and subset methods on FT-ICR MS peak abundance data to select the best normalization before running the main pipeline.

## When to use

Decision support for choosing a normalization method for direct-injection FT-ICR MS peak abundance tables before launching the main MetaboDirect pipeline.

## When NOT to use

- The peak abundance table is already normalized or the user has already fixed a normalization method outside the SPANS evaluation
- Working with raw, unassigned FT-ICR MS spectra — MetaboDirect and its test_normalization script expect peak abundance and assigned molecular formula data produced after initial processing, not raw spectra

## Inputs

- Peak abundance table from FT-ICR MS data (post molecular-formula assignment)
- Assigned molecular formula table for the same samples

## Outputs

- Table of SPANS scores per normalization-method/subset-method combination
- Heatmap of SPANS scores per dataset for visual selection of the best normalization method

## How to apply

Load the peak abundance table and the assigned molecular formula table for the dataset (e.g., the bacterium-phage dataset with 36 samples or the S. fallax leachate incubation dataset with 4 samples). Run the test_normalization companion script, which wraps a modified version of the pmartR spans_procedure (pmartR is listed as an R dependency 'for normalization tests'), evaluating combinations of candidate normalization methods and subset methods on each dataset. Collect the computed SPANS scores per normalization/subset combination into a score table. Render the scores as a heatmap with Matplotlib/seaborn so the best-performing normalization method can be visually selected; in the article's runs, median scored highest for the S. fallax dataset and z score for the bacterium-phage dataset. Run this before the main MetaboDirect pipeline so the chosen normalization feeds the downstream chemodiversity, statistical, and transformation-network analyses.

## Related tools

- **MetaboDirect** (Provides the test_normalization companion script that wraps pmartR's spans_procedure) — https://github.com/Coayala/MetaboDirect
- **pmartR** (R package whose spans_procedure underlies the SPANS score computation)
- **pandas** (Collects the per-combination SPANS scores into a score table)
- **Matplotlib** (Renders the SPANS score heatmap)
- **seaborn** (Renders the SPANS score heatmap)

## Evaluation signals

- A SPANS heatmap is generated for each dataset tested (e.g., both the phage and Sphagnum datasets)
- The heatmap resolves a single best-scoring normalization method per dataset
- The selected method matches the reported result: median for the S. fallax dataset, z score for the bacterium-phage dataset
- The score table contains one SPANS score for every evaluated normalization-method/subset-method combination

## Limitations

- SPANS scores are dataset-dependent: the same procedure selected median for the 4-sample S. fallax dataset but z score for the 36-sample bacterium-phage dataset, so results do not transfer across studies
- The script must be run before the main MetaboDirect pipeline; it supports normalization selection only, not the downstream analyses

## Evidence

- [intro] test_normalization is run before the main pipeline: [evidence span withheld — anchor status: not_found]
- [results] test_normalization used first on the showcased datasets: "the first step was to use the “test_normalization” companion script to help determine which normalization method worked the best"
- [readme] pmartR is the R dependency used for normalization tests: "pmartR (for normalization tests)"
- [abstract] Python stack used by MetaboDirect for tables and plots: [evidence span withheld — anchor status: not_found]
