---
name: fticr-ms-normalization-method-selection
description: Use when a direct-injection FT-ICR MS peak abundance table must be normalized
  before MetaboDirect analysis and the choice of normalization method (e.g., median
  vs z score) should be data-driven rather than arbitrary.
when_to_use_negative:
- Raw (unprocessed) FT-ICR MS spectra — MetaboDirect expects data after molecular
  formula assignment has already been performed, not mzML/NetCDF raw files
- Datasets where peak filtering has already been finalized and a normalization method
  has already been established by the original study (e.g., the S. fallax dataset,
  which was previously filtered by its authors)
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MetaboDirect
  role: Provides the test_normalization companion script that runs SPANS scoring before
    the main pipeline
  repo: https://github.com/Coayala/MetaboDirect
- name: pmartR
  role: R package whose spans_procedure (modified) is wrapped by test_normalization
    to compute SPANS scores
- name: seaborn
  role: Python library used to render the SPANS score heatmap for visual method selection
- name: pandas
  role: Python library used to collect and tabulate SPANS scores per normalization/subset
    combination
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:fba4bee1af410151e8a41c5eacd8005441f693f7a125a5b786f06de6a93aa9fc
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
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# fticr-ms-normalization-method-selection

## Summary

Runs MetaboDirect's test_normalization companion script, which wraps a modified pmartR SPANS (span normalization) procedure, to score candidate normalization and subset methods on FT-ICR MS peak abundance tables before launching the main pipeline.

## When to use

Use when a direct-injection FT-ICR MS peak abundance table must be normalized before MetaboDirect analysis and the choice of normalization method (e.g., median vs z score) should be data-driven rather than arbitrary.

## When NOT to use

- Raw (unprocessed) FT-ICR MS spectra — MetaboDirect expects data after molecular formula assignment has already been performed, not mzML/NetCDF raw files
- Datasets where peak filtering has already been finalized and a normalization method has already been established by the original study (e.g., the S. fallax dataset, which was previously filtered by its authors)

## Inputs

- Peak abundance table from processed direct-injection FT-ICR MS spectra (e.g., bacterium-phage dataset, 36 samples; S. fallax leachate dataset, 4 samples)
- Assigned molecular formula table (post molecular-formula-assignment, e.g., from MFAssignR/Formularity)

## Outputs

- SPANS score table with one score per normalization/subset-method combination
- SPANS score heatmap per dataset for visual selection of the best normalization method
- Chosen normalization method (e.g., median or z score) to pass to the main MetaboDirect pipeline

## How to apply

Load the peak abundance table and the assigned molecular formula table produced by prior FT-ICR MS processing, then run the test_normalization companion script, which evaluates combinations of candidate normalization methods and subset methods using a modified version of the pmartR spans_procedure. Collect the computed SPANS scores per normalization/subset combination into a score table, then render the scores as a heatmap (Python pandas, Matplotlib, seaborn) so the best-performing normalization method can be visually selected. On the MetaboDirect test datasets this procedure resolved a single best-scoring method per dataset: median for the 4-sample S. fallax leachate incubation dataset and z score for the 36-sample bacterium-phage dataset. Run this step before the main MetaboDirect pipeline, since the selected normalization affects all downstream diagnostics, chemodiversity, and statistics.

## Related tools

- **MetaboDirect** (Provides the test_normalization companion script that runs SPANS scoring before the main pipeline) — https://github.com/Coayala/MetaboDirect
- **pmartR** (R package whose spans_procedure (modified) is wrapped by test_normalization to compute SPANS scores)
- **seaborn** (Python library used to render the SPANS score heatmap for visual method selection)
- **pandas** (Python library used to collect and tabulate SPANS scores per normalization/subset combination)

## Evaluation signals

- A SPANS heatmap is generated for both test datasets (bacterium-phage and S. fallax)
- The heatmap resolves a single best-scoring normalization method consistent with the reported choices (median for S. fallax, z score for bacterium-phage)
- The score table contains one SPANS score for each evaluated normalization/subset-method combination
- The script completes before the main MetaboDirect pipeline is launched, as it is intended to be run beforehand

## Limitations

- Direct-injection MS cannot separate chemical isomers and suffers signal suppression or enhancement, which can confound downstream data analysis including normalization
- SPANS results are dataset-dependent — the same procedure selected different methods (median vs z score) on the two example datasets, so scores do not transfer across datasets
- MetaboDirect requires Python 3.5+, R 4+, Cytoscape 3.8+, and the pmartR R package for normalization tests, so environment setup is non-trivial

## Evidence

- [intro] An additional script, "test_normalization" is included to help users select the best normalization method and can be run before the main MetaboDirect pipeline: [evidence span withheld — anchor status: not_found]
- [results] the first step was to use the “test_normalization” companion script to help determine which normalization method worked the best: "the first step was to use the “test_normalization” companion script to help determine which normalization method worked the best"
- [readme] pmartR (for normalization tests): "pmartR (for normalization tests)"
- [intro] The pipeline accepts peak abundance and assigned molecular formula data produced after an initial processing of raw FT-ICR MS spectra: "The pipeline accepts peak abundance and assigned molecular formula data produced after an initial processing of raw FT-ICR MS spectra"
- [other] Running the SPANS-based test_normalization script on the two test datasets identified median (for the S. fallax dataset) and z score (for the bacterium-phage dataset) as the appropriate normalization methods: [evidence span withheld — anchor status: not_found]
