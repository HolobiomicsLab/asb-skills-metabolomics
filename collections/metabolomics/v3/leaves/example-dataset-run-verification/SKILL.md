---
name: example-dataset-run-verification
description: Use when a fresh MetaboDirect install or new environment must be validated
  against the article's example FT-ICR MS datasets, or when a user needs to select
  a normalization method via test_normalization before the main pipeline.
when_to_use_negative:
- Data has not yet had molecular formula assignment performed — MetaboDirect (and
  its normalization test) requires post-assignment peak abundance and formula tables
  as input
- Peaks still need filtering (m/z, 13C isotopes, 0.5 ppm formula error, minimum sample
  threshold) — the S. fallax set was already filtered by its original authors; test_normalization
  assumes cleaned input
- The goal is downstream analysis (chemodiversity, multivariate statistics, transformation
  networks) rather than normalization-method selection before the main pipeline
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboDirect
  role: Provides the test_normalization companion script that wraps the SPANS procedure
  repo: https://github.com/Coayala/MetaboDirect
- name: pmartR
  role: R library supplying the spans_procedure used for normalization testing
  repo: https://github.com/Coayala/MetaboDirect
- name: pandas
  role: Handles the peak abundance and formula tables while computing/collecting SPANS
    scores
  repo: https://github.com/Coayala/MetaboDirect
- name: Matplotlib
  role: Rendering the SPANS score heatmaps
- name: seaborn
  role: Heatmap visualization of SPANS scores per normalization/subset combination
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:ce6aa12073eb9eb4c823a19aae981c6e7a07349b40edaaa22a2217db782d1b1c
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# example-dataset-run-verification

## Summary

Run MetaboDirect's SPANS-based test_normalization companion script on the two published example FT-ICR MS datasets (bacterium-phage, 36 samples; S. fallax leachate, 4 samples) to reproduce the SPANS score heatmaps and confirm the reported normalization choices (median and z score) before launching the main pipeline.

## When to use

Use when a fresh MetaboDirect install or new environment must be validated against the article's example FT-ICR MS datasets, or when a user needs to select a normalization method via test_normalization before the main pipeline.

## When NOT to use

- Data has not yet had molecular formula assignment performed — MetaboDirect (and its normalization test) requires post-assignment peak abundance and formula tables as input
- Peaks still need filtering (m/z, 13C isotopes, 0.5 ppm formula error, minimum sample threshold) — the S. fallax set was already filtered by its original authors; test_normalization assumes cleaned input
- The goal is downstream analysis (chemodiversity, multivariate statistics, transformation networks) rather than normalization-method selection before the main pipeline

## Inputs

- Peak abundance table for the bacterium-phage model system dataset (36 samples)
- Peak abundance table for the S. fallax leachate incubation metabolomics dataset (4 samples)
- Assigned molecular formula tables produced after initial processing of the raw FT-ICR MS spectra

## Outputs

- SPANS score table per dataset (normalization method × subset method combinations)
- SPANS score heatmap for the bacterium-phage dataset
- SPANS score heatmap for the S. fallax dataset with a single best-scoring normalization method resolved

## How to apply

Load the peak abundance and assigned molecular formula tables for the bacterium-phage example dataset (36 samples from bacteria infected by two different phages) and the S. fallax leachate incubation dataset (4 samples). Run the test_normalization companion script, which wraps a modified version of the pmartR spans_procedure and evaluates combinations of candidate normalization methods and subset methods on each dataset. Collect the computed SPANS scores per normalization/subset combination into a score table, then render the scores as a heatmap (Matplotlib/seaborn) for each dataset so a single best-performing normalization method can be visually selected prior to the main MetaboDirect pipeline. Verification targets: median scores highest for the S. fallax dataset and z score scores highest for the bacterium-phage dataset, matching the article's reported choices.

## Related tools

- **MetaboDirect** (Provides the test_normalization companion script that wraps the SPANS procedure) — https://github.com/Coayala/MetaboDirect
- **pmartR** (R library supplying the spans_procedure used for normalization testing) — https://github.com/Coayala/MetaboDirect
- **pandas** (Handles the peak abundance and formula tables while computing/collecting SPANS scores) — https://github.com/Coayala/MetaboDirect
- **Matplotlib** (Rendering the SPANS score heatmaps)
- **seaborn** (Heatmap visualization of SPANS scores per normalization/subset combination)

## Evaluation signals

- A SPANS heatmap is generated for both the phage and the Sphagnum (S. fallax) datasets
- Each heatmap resolves a single best-scoring normalization method rather than ties across combinations
- Median has the highest SPANS score for the S. fallax dataset, matching the reported choice
- Z score has the highest SPANS score for the bacterium-phage dataset, matching the reported choice
- The R environment includes pmartR (listed in README as required 'for normalization tests')

## Limitations

- Direct injection FT-ICR MS cannot separate chemical isomers and suffers signal suppression or enhancement that can confound downstream data analysis, affecting the abundance values being normalized
- test_normalization is a pre-pipeline step; it selects the normalization method but does not itself produce the pipeline's analysis outputs
- The two datasets differ drastically in sample count (36 vs 4), so SPANS behavior on very small-sample datasets (S. fallax, 4 samples) may not generalize to larger designs

## Evidence

- [results] test_normalization companion script: "the first step was to use the “test_normalization” companion script to help determine which normalization method worked the best"
- [intro] test_normalization availability: [evidence span withheld — anchor status: not_found]
- [results] bacterium-phage dataset size: "The bacterium-phage model system data set included metabolites or mass spectrometry peaks present in 36 samples from bacteria infected by two different phages"
- [results] S. fallax dataset size: "The S. fallax leachate incubation metabolomics data set consisted of 4 samples"
- [readme] pmartR dependency: "pmartR (for normalization tests)"
