---
name: peak-abundance-table-preparation
description: Use when direct-injection FT-ICR MS data have already been formula-assigned
  but still need peak filtering and a normalization-method choice before running the
  MetaboDirect pipeline on datasets like the bacterium-phage (36 samples) or S. fallax
  (4 samples) examples.
when_to_use_negative:
- Input is still raw FT-ICR MS spectra that have not undergone molecular formula assignment
  — MetaboDirect is designed for data after assignment has been performed
- The dataset was already peak-filtered by the original authors and no normalization
  decision is pending (as with the S. fallax dataset in the paper)
- Analysis requires chromatographic separation of chemical isomers — direct injection
  MS cannot separate isomers and suffers signal suppression/enhancement
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboDirect
  role: Pipeline that consumes the prepared peak abundance and molecular formula tables;
    ships the test_normalization companion script run before its main 6-step analysis
  repo: https://github.com/Coayala/MetaboDirect
- name: pmartR
  role: R package whose spans_procedure is wrapped (modified) by test_normalization
    to compute SPANS scores for normalization/subset combinations
- name: seaborn
  role: Used with Matplotlib/pandas to render the SPANS score heatmap for visually
    selecting the best normalization method
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:68e998d315662658ede375b06b256970611a7eb155f538cf7633280295c068bb
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
yes_rate: 0.7777777777777778
n_claims: 18
---

# peak-abundance-table-preparation

## Summary

Prepares the peak abundance and assigned molecular formula tables that MetaboDirect requires as input — applying m/z, isotopic (13C), formula-assignment-error (0.5 ppm), and minimum-sample filtering — and runs the SPANS-based test_normalization companion script to select the best normalization method before launching the main pipeline.

## When to use

Use when direct-injection FT-ICR MS data have already been formula-assigned but still need peak filtering and a normalization-method choice before running the MetaboDirect pipeline on datasets like the bacterium-phage (36 samples) or S. fallax (4 samples) examples.

## When NOT to use

- Input is still raw FT-ICR MS spectra that have not undergone molecular formula assignment — MetaboDirect is designed for data after assignment has been performed
- The dataset was already peak-filtered by the original authors and no normalization decision is pending (as with the S. fallax dataset in the paper)
- Analysis requires chromatographic separation of chemical isomers — direct injection MS cannot separate isomers and suffers signal suppression/enhancement

## Inputs

- Peak abundance table from initial processing of raw direct-injection FT-ICR MS spectra
- Assigned molecular formula table (e.g., produced by MFAssignR or similar formula-assignment tool)
- User-defined filtering parameters (m/z range, isotopic 13C removal flag, 0.5 ppm formula-error tolerance, minimum number of samples)

## Outputs

- Filtered peak abundance table ready for the MetaboDirect main pipeline
- SPANS score table for normalization × subset method combinations
- SPANS score heatmap per dataset (PNG/figure via Matplotlib/seaborn)
- Selected normalization method (e.g., median, z score) to be used downstream

## How to apply

Start from the peak abundance and assigned molecular formula tables produced by upstream processing of raw FT-ICR MS spectra (MetaboDirect does not do formula assignment itself; tools like MFAssignR are cited as an alternative to Formularity). Filter detected peaks by user-specified m/z boundaries, remove peaks whose assigned formulas contain isotopic carbon (~200 masses were dropped from the bacterium-phage dataset), require formula-assignment error ≤ 0.5 ppm, and enforce a user-defined minimum number of samples; skip filtering entirely if the dataset was already filtered by its original authors (as for the S. fallax data). With the cleaned table, run the test_normalization companion script, which wraps a modified pmartR spans_procedure (R pmartR library) and evaluates combinations of candidate normalization and subset methods, collecting SPANS scores into a table. Render the SPANS scores as a heatmap (Python: pandas, Matplotlib, seaborn) per dataset and pick the highest-scoring normalization — in the paper's examples median for S. fallax and z score for the bacterium-phage system. Only then launch the main 6-step MetaboDirect pipeline (pre-processing, diagnostics, exploration, chemodiversity, statistics, transformation networks), which the README installs via pip (requires Python ≥3.5 and R ≥4).

## Related tools

- **MetaboDirect** (Pipeline that consumes the prepared peak abundance and molecular formula tables; ships the test_normalization companion script run before its main 6-step analysis) — https://github.com/Coayala/MetaboDirect
- **pmartR** (R package whose spans_procedure is wrapped (modified) by test_normalization to compute SPANS scores for normalization/subset combinations)
- **seaborn** (Used with Matplotlib/pandas to render the SPANS score heatmap for visually selecting the best normalization method)

## Evaluation signals

- A SPANS heatmap is generated for both the phage and Sphagnum datasets and each resolves a single best-scoring normalization method
- Selected normalization matches the reported choices: median for the S. fallax dataset and z score for the bacterium-phage dataset (highest SPANS score)
- Isotopic filtering removes ~200 masses with 13C-containing assigned formulas from the bacterium-phage dataset
- No peaks are filtered from the S. fallax dataset, since it was previously filtered by the original authors
- Filtered peaks retain formula assignments with error within the 0.5 ppm threshold

## Limitations

- Direct injection MS cannot separate chemical isomers and suffers signal suppression or enhancement, which can confound downstream data analysis
- The minimum-number-of-samples filter threshold is user-defined, so filtering outcomes depend on user choices
- test_normalization must be run before the main MetaboDirect pipeline; normalization choice is dataset-specific (median vs z score in the paper's two examples)
- MetaboDirect requires a multi-language environment: Python (≥3.5), R (≥4) with pmartR, and Cytoscape (≥3.8)

## Evidence

- [results] the first step was to use the “test_normalization” companion script to help determine which normalization method worked the best: "the first step was to use the “test_normalization” companion script to help determine which normalization method worked the best"
- [intro] An additional script, "test_normalization" is included to help users select the best normalization method and can be run before the main MetaboDirect pipeline: [evidence span withheld — anchor status: not_found]
- [intro] detected peaks are filtered by their m/z values (based on the user's input), isotopic presence (13C peaks), error in formula assignment (0.5 ppm): [evidence span withheld — anchor status: not_found]
- [results] ~200 masses were filtered out from the data set because they were assigned molecular formulas containing an isotopic carbon: "~200 masses were filtered out from the data set because they were assigned molecular formulas containing an isotopic carbon"
- [intro] The pipeline accepts peak abundance and assigned molecular formula data produced after an initial processing of raw FT-ICR MS spectra: "The pipeline accepts peak abundance and assigned molecular formula data produced after an initial processing of raw FT-ICR MS spectra"
- [readme] MetaboDirect requires Python (3.5 and above), R (4 and above) and Cytoscape (3.8 and above): "MetaboDirect requires Python (3.5 and above), R (4 and above) and Cytoscape (3.8 and above)"
