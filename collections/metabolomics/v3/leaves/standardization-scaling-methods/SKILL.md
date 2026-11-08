---
name: standardization-scaling-methods
description: Use when raw ICP-MS ionomics concentrations have already been log-transformed
  and batch-corrected and line-level z-score profiles are needed for PCA, hierarchical
  clustering, or relevance network extraction.
when_to_use_negative:
- Input data are already normalised and standardised — IonFlow can directly process
  such data for genetic network extraction and element-element correlation analysis
  without re-running PreProcessing standardisation.
- Data contain no replicate structure and no external control-derived standard deviations
  are available, so mad/std scaling cannot be validated against control measurements.
- The analysis target is metabolomics spectra rather than element concentrations from
  ICP-MS experiments; the standardisation parameters were defined for ionome data.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: IonFlow
  role: R package whose PreProcessing function implements std/mad/custom standardisation
    and emits data.line.zscores
  repo: https://github.com/wanchanglin/ionflow
- name: Galaxy
  role: Web interface wrapping the IonFlow standardisation step for non-command-line
    users
  repo: https://github.com/galaxyproject/galaxy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-021-01841-z
    title: 'IonFlow: a galaxy tool for the analysis of ionomics data sets'
  source_content_hash: sha256:042c0c4d03c9166ee77076773c5fef067e139477758665d187b1b0cbe615249c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wanchanglin/ionflow
  tools:
  - Galaxy
  - IonFlow
  - R
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# standardization-scaling-methods

## Summary

Standardisation step of the IonFlow PreProcessing pipeline, which scales log-transformed and batch-corrected ICP-MS ion concentrations into z-score profiles using std, mad, or user-supplied custom standard deviations from control strains, producing line-level z-score tables for downstream clustering and network analysis.

## When to use

Use when raw ICP-MS ionomics concentrations have already been log-transformed and batch-corrected and line-level z-score profiles are needed for PCA, hierarchical clustering, or relevance network extraction.

## When NOT to use

- Input data are already normalised and standardised — IonFlow can directly process such data for genetic network extraction and element-element correlation analysis without re-running PreProcessing standardisation.
- Data contain no replicate structure and no external control-derived standard deviations are available, so mad/std scaling cannot be validated against control measurements.
- The analysis target is metabolomics spectra rather than element concentrations from ICP-MS experiments; the standardisation parameters were defined for ionome data.

## Inputs

- Raw ionomics data frame of ion concentrations (e.g. IonData: 14 ions — Ca44, Cd111, Co59, Cu65, Fe56, K39, Mg25, Mn55, Mo95, Na23, Ni60, P31, S34, Zn66 — in ppm, OD-adjusted, from 1454 single-gene haploid yeast knockouts)
- Optional pre-defined per-ion standard deviations (stdev) computed from control genes/strains

## Outputs

- data.line.zscores table (line-level z-score profiles per ion)
- data.line.symb symbolic profiles after applying the thres_symb threshold
- Diagnostic stats tables (stats.raw_data, stats.outliers, stats.median_batch_corrected_data)

## How to apply

After log-transformation and median batch correction (scaling each ion to the median within each batch, e.g. YDL227C control strain in the S. cerevisiae dataset), call the PreProcessing standardisation step with the chosen method: std (classical standard deviation), mad (robust median absolute deviation), or custom scaling. The custom option accepts a pre-defined set of ion standard deviations, typically computed from control samples, so that replicate structure is respected. If no external standard deviations are supplied, they are computed from the data itself. The scaled output is assembled into data.line.zscores (and, after thresholding at thres_symb, data.line.symb), which feed the exploratory, clustering, and network-analysis sections of the workflow.

## Related tools

- **IonFlow** (R package whose PreProcessing function implements std/mad/custom standardisation and emits data.line.zscores) — https://github.com/wanchanglin/ionflow
- **Galaxy** (Web interface wrapping the IonFlow standardisation step for non-command-line users) — https://github.com/galaxyproject/galaxy

## Examples

```
pre_proc <- PreProcessing(data=IonData, stdev=pre_defined_sd)
```

## Evaluation signals

- Median of each batch-corrected ion column is exactly 0 before standardisation (see stats.median_batch_corrected_data table).
- data.line.zscores exists, has one row per knockout line, and z-score columns for all 14 ions; no missing values (README states there are none in the yeast data).
- Choosing mad vs std changes the scale factors; the stats tables should reflect the selected method and, for custom mode, match the supplied pre_defined_sd values.
- Downstream sections (clustering, network analysis) accept the cleaned z-score dataset without further preprocessing.
- Outlier percentages per ion (stats.outliers) remain consistent with the fence-based detection (Q1 - 3*IQ / Q3 + 3*IQ) applied before standardisation.

## Limitations

- The standard deviation input is optional; if not provided, SDs are computed from the data, which may differ from control-derived values and affect z-score comparability.
- Standardisation operates on log-transformed, median-batch-corrected values, so applying it out of order or to raw concentrations gives wrong scales.
- The paper reports no quantitative benchmark metrics for the two case studies, so correctness must be judged from the output tables and diagnostic plots rather than published reference values.

## Evidence

- [readme] Optional pre-defined SD input: [evidence span withheld — anchor status: external_doc]
- [readme] Fallback to data-derived SDs: [evidence span withheld — anchor status: external_doc]
- [readme] Purpose of pre-processing: [evidence span withheld — anchor status: external_doc]
- [intro] Pipeline context and instrument: "we developed IonFlow, an R-based Galaxy tool designed for the analysis of ionome data sets from ICP-MS experiments"
- [discussion] Already-standardised data can skip this step: "IonFlow is very versatile and it can be used to process raw data as well as directly process normalised and standardised data for advanced analysis"
