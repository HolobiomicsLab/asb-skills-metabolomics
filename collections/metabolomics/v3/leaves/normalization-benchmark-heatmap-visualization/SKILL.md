---
name: normalization-benchmark-heatmap-visualization
description: Decision support for choosing a normalization method before running the
  MetaboDirect pipeline on peak abundance tables with assigned molecular formulas
  — trigger when the pipeline's normalization step must be justified per dataset.
when_to_use_negative:
- The input is raw FT-ICR MS spectra rather than peak abundance and assigned molecular
  formula data produced after initial processing
- A normalization method has already been fixed/validated for the dataset — re-benchmarking
  with SPANS is unnecessary
- The task is a downstream MetaboDirect step (chemodiversity, multivariate statistics,
  transformation networks) rather than the pre-pipeline normalization choice
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MetaboDirect
  role: Provides the test_normalization companion script that wraps the SPANS procedure;
    run before the main pipeline
  repo: https://github.com/Coayala/MetaboDirect
- name: pmartR
  role: R package supplying the spans_procedure whose modified version computes SPANS
    scores for normalization/subset combinations
  repo: https://github.com/Coayala/MetaboDirect
- name: Python
  role: Language in which MetaboDirect and the test_normalization script are written
  repo: https://github.com/Coayala/MetaboDirect
- name: pandas
  role: Handles the peak abundance and formula tables and collects SPANS scores into
    a score table
  repo: https://github.com/Coayala/MetaboDirect
- name: Matplotlib
  role: Renders the SPANS score heatmap for each dataset
  repo: https://github.com/Coayala/MetaboDirect
- name: seaborn
  role: Heatmap visualization of SPANS scores for visual selection of the best normalization
    method
  repo: https://github.com/Coayala/MetaboDirect
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:8fcfee02ded0978b1c9b6902041dd2b871123c7aef1285d3c7fe13106166eb8c
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
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# normalization-benchmark-heatmap-visualization

## Summary

Benchmarks candidate normalization methods for direct-injection FT-ICR MS metabolomics data using the SPANS procedure (via MetaboDirect's test_normalization companion script wrapping pmartR's spans_procedure), then renders the resulting SPANS scores as a heatmap so a single best-scoring method (e.g., median or z-score) can be selected before running the main MetaboDirect pipeline.

## When to use

Decision support for choosing a normalization method before running the MetaboDirect pipeline on peak abundance tables with assigned molecular formulas — trigger when the pipeline's normalization step must be justified per dataset.

## When NOT to use

- The input is raw FT-ICR MS spectra rather than peak abundance and assigned molecular formula data produced after initial processing
- A normalization method has already been fixed/validated for the dataset — re-benchmarking with SPANS is unnecessary
- The task is a downstream MetaboDirect step (chemodiversity, multivariate statistics, transformation networks) rather than the pre-pipeline normalization choice

## Inputs

- Peak abundance table (post molecular-formula-assignment FT-ICR MS data)
- Assigned molecular formula table
- Sample metadata for the FT-ICR MS dataset (e.g., 36 bacterium-phage samples or 4 S. fallax leachate incubation samples)

## Outputs

- SPANS score table of normalization-method × subset-method combinations
- SPANS score heatmap per dataset (Matplotlib/seaborn figure) resolving a single best-scoring normalization method
- Selected normalization method to pass to the main MetaboDirect pipeline

## How to apply

Load the peak abundance table and assigned molecular formula table for the dataset (e.g., the 36-sample bacterium-phage model system dataset or the 4-sample S. fallax leachate incubation dataset). Run the test_normalization companion script, which wraps a modified version of the pmartR spans_procedure and evaluates combinations of candidate normalization methods (e.g., median, z score) and subset methods on each dataset. Collect the computed SPANS scores per normalization/subset combination into a score table, then render the scores as a heatmap with Matplotlib/seaborn so the highest-SPANS method can be visually identified. Validate that exactly one best-scoring normalization method emerges; in the article's application, median scored highest for the S. fallax dataset and z score for the bacterium-phage dataset, showing the choice is dataset-dependent and must be re-run per input. Run this before the main single-line MetaboDirect pipeline so downstream chemodiversity and statistical analyses use the selected normalization.

## Related tools

- **MetaboDirect** (Provides the test_normalization companion script that wraps the SPANS procedure; run before the main pipeline) — https://github.com/Coayala/MetaboDirect
- **pmartR** (R package supplying the spans_procedure whose modified version computes SPANS scores for normalization/subset combinations) — https://github.com/Coayala/MetaboDirect
- **Python** (Language in which MetaboDirect and the test_normalization script are written) — https://github.com/Coayala/MetaboDirect
- **pandas** (Handles the peak abundance and formula tables and collects SPANS scores into a score table) — https://github.com/Coayala/MetaboDirect
- **Matplotlib** (Renders the SPANS score heatmap for each dataset) — https://github.com/Coayala/MetaboDirect
- **seaborn** (Heatmap visualization of SPANS scores for visual selection of the best normalization method) — https://github.com/Coayala/MetaboDirect

## Evaluation signals

- A SPANS heatmap is generated for both example datasets (bacterium-phage and S. fallax)
- Each heatmap resolves a single best-scoring normalization method (median for S. fallax, z score for bacterium-phage)
- The SPANS score table covers every evaluated normalization-method × subset-method combination from the spans_procedure run
- The heatmap is produced before the main MetaboDirect pipeline is launched, so its selected method feeds the pipeline run
- Re-running the script on the same inputs reproduces the same SPANS score ordering

## Limitations

- The best normalization method is dataset-dependent — median for the 4-sample S. fallax dataset vs z score for the 36-sample bacterium-phage dataset — so a choice must not be transferred between datasets
- SPANS evaluates only the candidate normalization and subset method combinations implemented in the modified pmartR spans_procedure wrapper
- Direct-injection FT-ICR MS signal suppression or enhancement can confound downstream analysis, making the normalization choice consequential for data quality

## Evidence

- [intro] test_normalization is run before the main pipeline: [evidence span withheld — anchor status: not_found]
- [results] first step in the showcased analysis: "the first step was to use the “test_normalization” companion script to help determine which normalization method worked the best"
- [abstract] Python visualization stack for the heatmap: [evidence span withheld — anchor status: not_found]
- [readme] pmartR dependency for normalization tests: "MetaboDirect requires Python (3.5 and above), R (4 and above) and Cytoscape (3.8 and above) with the following libraries/modules"
