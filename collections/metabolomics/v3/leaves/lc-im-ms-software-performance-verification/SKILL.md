---
name: lc-im-ms-software-performance-verification
description: Use when you need to quantify how a tool's runtime scales with input
  mzML file size (e.g., 460–1083 MB) and with the number of input molecular formulae,
  or to verify a runtime acceptance threshold such as <2 min per GB on a 32 GB RAM
  desktop.
when_to_use_negative:
- The goal is validating analytical output quality (e.g., CCS calibration accuracy
  or isotopologue matching) rather than computational runtime scaling.
- Input timing data comes from a non-fixed configuration (file size and formula count
  varied simultaneously), confounding the two linear relationships.
- The software being benchmarked does not accept mzML input or process molecular formulae,
  so file-size/formula-count scaling is not meaningful.
edam_operation: http://edamontology.org/operation_2945
tools:
- name: CCSfind
  role: Software under benchmark; processes mzML files for targeted molecular formulae
    and is timed across file sizes and formula counts
  repo: https://github.com/sangeeta97/ccs_find.git
- name: pandas
  role: Holds benchmark timing measurements in a DataFrame and writes fitted parameters
    and goodness-of-fit statistics to CSV with XlsxWriter
- name: scipy
  role: Provides scipy.stats.linregress for ordinary least-squares fitting of runtime
    versus file size and versus formula count
- name: matplotlib
  role: Renders the two fitted linear relationships (runtime vs file size, runtime
    vs formula count) as a figure
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/jms.5040
    title: 'CCSfind: A tool for chemically informed LC‐IM‐MS database building'
  source_content_hash: sha256:0081a40dc8ce83bfed69ea9abc227d8fef0ea282fe2bbbd099c1b7f142a0bf71
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# lc-im-ms-software-performance-verification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark and verify the computational performance of LC-IM-MS data processing software (e.g., CCSfind) by fitting linear models of total runtime versus mzML file size and versus number of targeted molecular formulae, then checking fitted runtimes against reported acceptance criteria.

## When to use

Use when you need to quantify how a tool's runtime scales with input mzML file size (e.g., 460–1083 MB) and with the number of input molecular formulae, or to verify a runtime acceptance threshold such as <2 min per GB on a 32 GB RAM desktop.

## When NOT to use

- The goal is validating analytical output quality (e.g., CCS calibration accuracy or isotopologue matching) rather than computational runtime scaling.
- Input timing data comes from a non-fixed configuration (file size and formula count varied simultaneously), confounding the two linear relationships.
- The software being benchmarked does not accept mzML input or process molecular formulae, so file-size/formula-count scaling is not meaningful.

## Inputs

- Total computational time measurements (s) paired with input mzML file sizes (MB) for a fixed formula set (e.g., 3 formulae)
- Incremental computational time measurements (s) paired with number of added molecular formulae on a fixed-size mzML file (~500 MB)
- Hardware specification of the benchmark machine (e.g., desktop with 32 GB RAM)

## Outputs

- benchmark_linear_fit_results.csv with fitted slope, intercept, and goodness-of-fit statistics (R²)
- Matplotlib figure rendering the runtime-versus-file-size and runtime-versus-formula-count linear fits
- Accept/reject verdict versus the <2 min per 1 GB runtime criterion

## How to apply

Load the benchmark timing measurements (total runtime 89–200 s for 460–1083 MB mzML files processed with three molecular formulae, and incremental runtime of 15–38 s per added formula on a ~500 MB file) into a pandas DataFrame. Fit an ordinary least-squares model of total processing time versus mzML file size and a second model of additional processing time versus number of input molecular formulae using scipy.stats.linregress, recording slope, intercept, and R². Check both fits against the reported acceptance criteria: runtime below 2 min for files up to 1 GB on a 32 GB RAM desktop, and linear correlation with file size and formula count. Write fitted parameters and goodness-of-fit statistics to benchmark_linear_fit_results.csv with pandas/XlsxWriter and render the two fitted relationships as a matplotlib figure.

## Related tools

- **CCSfind** (Software under benchmark; processes mzML files for targeted molecular formulae and is timed across file sizes and formula counts) — https://github.com/sangeeta97/ccs_find.git
- **pandas** (Holds benchmark timing measurements in a DataFrame and writes fitted parameters and goodness-of-fit statistics to CSV with XlsxWriter)
- **scipy** (Provides scipy.stats.linregress for ordinary least-squares fitting of runtime versus file size and versus formula count)
- **matplotlib** (Renders the two fitted linear relationships (runtime vs file size, runtime vs formula count) as a figure)

## Examples

```
from scipy.stats import linregress; slope, intercept, r, p, se = linregress(file_size_mb, runtime_s)  # fit total runtime vs mzML size
```

## Evaluation signals

- R² of the runtime-versus-file-size fit indicates a strong linear relationship, consistent with the reported linear scaling.
- R² of the runtime-versus-formula-count fit indicates a strong linear relationship, matching the reported 15–38 s per added formula range.
- Predicted runtimes fall within the reported 89–200 s range for 460–1083 MB mzML files with three formulae.
- Acceptance criterion satisfied: total runtime below 2 min for mzML files up to 1 GB on a 32 GB RAM desktop.
- benchmark_linear_fit_results.csv contains slope, intercept, and R² for both fits and is readable via pandas/XlsxWriter.

## Limitations

- Benchmarks are tied to a specific desktop configuration with 32 GB RAM; runtimes may differ on other hardware.
- The linear models were established for drift tube IM-MS mzML profile data with a fixed set of three targeted formulae; extrapolation to much larger formula lists or other data types is unverified.
- Re-running analysis on an already parsed mzML file is faster, so cold-parse timings should not be mixed with warm re-run timings in one fit.

## Evidence
