---
name: lc-hrms-peak-matching-against-benchmark
description: Use when you have a validated mzRAPP benchmark dataset plus one or more
  NPP output tables (e.g., XCMS unaligned/aligned run CSVs from LC-HRMS mzML data)
  and need reliability metrics such as % peaks found, split peaks, peaks lost in alignment,
  or degenerated isotopologue ratios.
when_to_use_negative:
- You only have raw vendor or mzML spectra and no NPP feature table yet — generate
  the NPP output (e.g., run XCMS) before matching
- No validated benchmark exists for the same mzML files/target molecules — mzRAPP
  matching requires BM peaks extracted from the identical LC-HRMS dataset
- The aim is targeted quantification rather than assessing the reliability of a non-targeted
  pre-processing workflow
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: mzRAPP
  role: Provides the matching and metric-derivation functions (match_peaks_to_benchmark,
    match_features_to_benchmark, assess_alignment, check_IR_biases, derive_performance_metrics)
    applied to the benchmark and NPP tables
  repo: https://github.com/YasinEl/mzRAPP
- name: XCMS
  role: Source of the non-targeted pre-processing output (unaligned/aligned feature
    tables) whose reliability is assessed via the benchmark matching
- name: enviPat
  role: Predicts the isotopologues whose extracted chromatographic peaks form the
    benchmark that NPP peaks are matched against
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btab231
    title: 'mzRAPP: a tool for reliability assessment of data pre-processing in non-targeted
      metabolomics'
  source_content_hash: sha256:a82da2479c83069d055735e10b0d522f46259a40b1dcf4350f373014f860617f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YasinEl/mzRAPP
  tools:
  - R
  - XCMS
  - enviPat
  - mzRAPP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# lc-hrms-peak-matching-against-benchmark

## Summary

Matches mzRAPP benchmark peaks/features (target molecules with known RT boundaries and enviPat-predicted isotopologues) against non-targeted pre-processing (NPP) output tables (e.g., XCMS unaligned/aligned CSVs) to quantify found/split peaks, alignment errors, and isotopologue-ratio degeneration.

## When to use

Use when you have a validated mzRAPP benchmark dataset plus one or more NPP output tables (e.g., XCMS unaligned/aligned run CSVs from LC-HRMS mzML data) and need reliability metrics such as % peaks found, split peaks, peaks lost in alignment, or degenerated isotopologue ratios.

## When NOT to use

- You only have raw vendor or mzML spectra and no NPP feature table yet — generate the NPP output (e.g., run XCMS) before matching
- No validated benchmark exists for the same mzML files/target molecules — mzRAPP matching requires BM peaks extracted from the identical LC-HRMS dataset
- The aim is targeted quantification rather than assessing the reliability of a non-targeted pre-processing workflow

## Inputs

- Benchmark dataset CSV from mzRAPP benchmark generation (e.g., Benchmark.csv)
- Unaligned NPP output table (e.g., XCMS_unaligned_run1.csv)
- Aligned NPP output table (e.g., XCMS_aligned_run1.csv)
- Centroided mzML LC-HRMS files underlying the NPP run (same files the benchmark was built from)

## Outputs

- Match tables of benchmark peaks/features vs. NPP peaks/features
- Per-stage reliability metrics: found/not found peaks, split-peak percentage, missing-peak classification, degenerated isotopologue-ratio fraction, alignment error counts
- Bootstrapped 95% confidence intervals (alpha = 0.95) on all metrics
- Summary assessment table (e.g., xcms_run1_npp_assessment.csv)

## How to apply

Launch mzRAPP in R and load the benchmark (Benchmark.csv) together with the unaligned and aligned NPP output tables (e.g., XCMS_unaligned_run1.csv and XCMS_aligned_run1.csv). Validate with check_nonTargeted_input(), then import via import_ungrouped_xcms()/import_grouped_xcms() with algo='XCMS'. Match chromatographic peaks to the benchmark with match_peaks_to_benchmark() and match_peaks_to_benchmark_split() to quantify found peaks and split-peak matches, then match features with match_features_to_benchmark() and run assess_alignment()/count_alignment_errors() to count peaks lost when going from unaligned to aligned output. Assess isotopologue-ratio degeneration with check_IR_biases() and summarize per-stage performance with derive_performance_metrics(), estimating confidence intervals by bootstrapping (alpha = 0.95). Judged correct when metrics fall near published run-1 values: 2660/2870 benchmark peaks found, up to 12% split peaks, 19-35% peaks lost in alignment, 83-94% post-alignment detection, and 28-53% degenerated isotopologue ratios (small variation expected from bootstrap randomness).

## Related tools

- **mzRAPP** (Provides the matching and metric-derivation functions (match_peaks_to_benchmark, match_features_to_benchmark, assess_alignment, check_IR_biases, derive_performance_metrics) applied to the benchmark and NPP tables) — https://github.com/YasinEl/mzRAPP
- **XCMS** (Source of the non-targeted pre-processing output (unaligned/aligned feature tables) whose reliability is assessed via the benchmark matching)
- **enviPat** (Predicts the isotopologues whose extracted chromatographic peaks form the benchmark that NPP peaks are matched against)

## Examples

```
library(mzRAPP); ungrp <- import_ungrouped_xcms('XCMS_unaligned_run1.csv', algo='XCMS'); grp <- import_grouped_xcms('XCMS_aligned_run1.csv', algo='XCMS'); pm <- match_peaks_to_benchmark(BM, ungrp); metrics <- derive_performance_metrics(match_features_to_benchmark(BM, grp))
```

## Evaluation signals

- check_nonTargeted_input() passes on all supplied tables before import
- Found-peak count close to reported run-1 benchmark recovery of 2660/2870 peaks
- Split-peak matches up to ~12% and peaks lost during alignment in the 19-35% range
- Post-alignment detection of 83-94% and degenerated isotopologue ratios of 28-53%, reproduced within bootstrap randomness (alpha = 0.95 CIs)
- Metric summary is written out (e.g., xcms_run1_npp_assessment.csv) with confidence intervals present

## Limitations

- Benchmark peaks are only retained if the most abundant plus at least one additional isotopologue are detected, with peak area/height within 30% of predicted values and Pearson correlation >= 0.85 vs. the highest isotopologue
- Metrics derived by bootstrapping vary slightly between runs due to bootstrap randomness
- Matching assumes the benchmark and NPP output derive from the same mzML files and that the main adduct (e.g., M+H or M-H) of each molecule was detected

## Evidence

- [readme] mzRAPP extracts and validates chromatographic peaks for which boundaries are provided for all (enviPat predicted) isotopologues of those target molecules directly from mzML files.: "mzRAPP extracts and validates chromatographic peaks for which boundaries are provided for all (enviPat predicted) isotopologues of those target molecules directly from mzML files."
- [readme] The resulting benchmark dataset is used to extract different performance metrics for NPP performed on the same mzML files.: [evidence span withheld — anchor status: external_doc]
- [readme] Isotopologue peaks with an area or height which is more than 30% off the predicted value or a Pearson Correlation coef < 0.85 (as compared to the highest isotopologue) are removed.: [evidence span withheld — anchor status: not_found]
- [abstract] which generates and validates a benchmark from user-supplied information and later utilizes it for reliability assessment of data pre-processing: "which generates and validates a benchmark from user-supplied information and later utilizes it for reliability assessment of data pre-processing"
