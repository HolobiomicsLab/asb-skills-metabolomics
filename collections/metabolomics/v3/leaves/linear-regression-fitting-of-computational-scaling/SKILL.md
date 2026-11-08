---
name: linear-regression-fitting-of-computational-scaling
description: Use when benchmark timing measurements exist for a tool's runtime across
  varying input file sizes or parameter counts, and you must verify a claimed linear
  scaling relationship quantitatively.
when_to_use_negative:
- The goal is to benchmark a different tool or hardware configuration for which no
  timing measurements at multiple file sizes or formula counts have been collected
- The suspected scaling behavior is non-linear (e.g. quadratic or exponential), where
  a single ordinary least-squares linear fit would not test the relationship meaningfully
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: CCSfind
  role: Tool whose benchmark timing data (total runtime for 460–1083 MB mzML files,
    incremental runtime per added molecular formula) is being fitted
  repo: https://github.com/sangeeta97/ccs_find.git
- name: scipy
  role: Provides scipy.stats.linregress for the ordinary least-squares fits of runtime
    versus mzML file size and versus number of molecular formulae
- name: pandas
  role: Loads the benchmark timing measurements into a DataFrame and writes the fitted
    parameters and goodness-of-fit statistics to benchmark_linear_fit_results.csv
- name: XlsxWriter
  role: Backend engine used with pandas to write benchmark_linear_fit_results.csv
- name: matplotlib
  role: Renders the two fitted linear scaling relationships as a figure for visual
    verification
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/jms.5040
    title: 'CCSfind: A tool for chemically informed LC‐IM‐MS database building'
  source_content_hash: sha256:65dfecfddba30a06f5abbfbd64f49ca9e097a6116a7775bf527a6305a623e762
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sangeeta97/ccs_find
  repo_url: https://github.com/sangeeta97/ccs_find.git
  tools:
  - CCSfind
  - Python
  - XlsxWriter
  - matplotlib
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# linear-regression-fitting-of-computational-scaling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fits ordinary least-squares linear models to benchmark timing data from CCSfind runs, testing whether total computational time scales linearly with mzML input file size (89–200 s for 460–1083 MB) and with the number of targeted molecular formulae (15–38 s per added formula on a ~500 MB file).

## When to use

Use when benchmark timing measurements exist for a tool's runtime across varying input file sizes or parameter counts, and you must verify a claimed linear scaling relationship quantitatively.

## When NOT to use

- The goal is to benchmark a different tool or hardware configuration for which no timing measurements at multiple file sizes or formula counts have been collected
- The suspected scaling behavior is non-linear (e.g. quadratic or exponential), where a single ordinary least-squares linear fit would not test the relationship meaningfully

## Inputs

- Benchmark timing table: total CCSfind runtime (s) vs profile mzML file size (MB), e.g. 89 s at 460 MB to 200 s at 1083 MB for three molecular formulae
- Benchmark timing table: incremental runtime (s) per added molecular formula on a ~500 MB mzML file, e.g. 15–38 s
- Acceptance criteria: runtime below 2 min for files up to 1 GB on a 32 GB RAM desktop; linear correlation with file size and formula count

## Outputs

- benchmark_linear_fit_results.csv with fitted slope, intercept, and goodness-of-fit statistics (R²) from scipy.stats.linregress
- matplotlib figure rendering the two fitted linear relationships (runtime vs file size; runtime vs formula count)

## How to apply

Load the Section 3.3 benchmark measurements into a pandas DataFrame: total CCSfind runtimes of 89–200 s for profile mzML files of 460–1083 MB processed with three molecular formulae, and incremental runtimes of 15–38 s per added formula on a ~500 MB file. Fit total processing time versus mzML file size with scipy.stats.linregress to obtain slope, intercept, and R², then fit additional processing time versus number of input molecular formulae the same way. Accept each fit only if R² indicates a strong linear relationship and the predicted runtimes fall within the reported 89–200 s (file-size) and 15–38 s (per-formula) ranges, consistent with the acceptance criterion of sub-2-min processing of files up to 1 GB on a 32 GB RAM desktop. Write the fitted parameters and goodness-of-fit statistics to benchmark_linear_fit_results.csv with pandas/XlsxWriter and render both fitted relationships as a matplotlib figure for visual confirmation of linearity.

## Related tools

- **CCSfind** (Tool whose benchmark timing data (total runtime for 460–1083 MB mzML files, incremental runtime per added molecular formula) is being fitted) — https://github.com/sangeeta97/ccs_find.git
- **scipy** (Provides scipy.stats.linregress for the ordinary least-squares fits of runtime versus mzML file size and versus number of molecular formulae)
- **pandas** (Loads the benchmark timing measurements into a DataFrame and writes the fitted parameters and goodness-of-fit statistics to benchmark_linear_fit_results.csv)
- **XlsxWriter** (Backend engine used with pandas to write benchmark_linear_fit_results.csv)
- **matplotlib** (Renders the two fitted linear scaling relationships as a figure for visual verification)

## Examples

```
from scipy.stats import linregress; fit = linregress(file_sizes_mb, runtimes_s); print(fit.slope, fit.intercept, fit.rvalue**2)
```

## Evaluation signals

- R² from scipy.stats.linregress indicates a strong linear relationship for both the file-size and formula-count fits
- Predicted total runtimes from the file-size fit fall within the reported 89–200 s range for 460–1083 MB mzML files
- Predicted per-formula incremental runtimes fall within the reported 15–38 s range on the ~500 MB file
- benchmark_linear_fit_results.csv is written and contains slope, intercept, and goodness-of-fit statistics
- The fitted file-size relationship is consistent with sub-2-min processing of files up to 1 GB on a 32 GB RAM desktop

## Limitations

- The linear scaling claim is validated only over the measured range of 460–1083 MB mzML files with three molecular formulae; extrapolation beyond this range is untested
- Per-formula timing (15–38 s) was measured on a single ~500 MB file, so the formula-count fit may not generalize to other file sizes
- The acceptance criterion (sub-2-min runtime for files up to 1 GB) is hardware-specific: a desktop computer with 32 GB RAM

## Evidence

- [results] Input mzML files of 460 MB up to 1083 MB required total computational times of 89 s up to 200 s for three molecular formulae, and the expected linear relationship between computational time and input file size was observed.: [evidence span withheld — anchor status: not_found]
