---
name: lc-hrms-preprocessing-metric-computation
description: Use when you have LC-HRMS mzML files, a benchmark of target molecules
  with known retention-time boundaries and sum formulas, and NPP outputs (e.g. XCMS
  aligned/unaligned CSVs) whose reliability you need to quantify per pre-processing
  stage.
when_to_use_negative:
- You have no benchmark of target molecules with known sum formulas and retention-time
  boundaries — mzRAPP requires user-provided targets from which enviPat can predict
  isotopologues
- Your data are targeted MRM/SRM peak areas only, not untargeted NPP outputs (feature
  tables from XCMS/MZmine/MS-DIAL) to be matched against the benchmark
- Data are low-resolution MS or non-LC acquisitions — the skill is scoped to liquid
  chromatography high-resolution mass spectrometry (LC-HRMS) with centroided mzML
  input
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: mzRAPP
  role: Generates/validates the benchmark, matches NPP outputs against it, and computes
    the NPP performance metrics
  repo: https://github.com/YasinEl/mzRAPP
- name: boot
  role: R package used to draw resamples of benchmark molecules and derive percentile
    confidence intervals for each metric
- name: XCMS
  role: Example NPP tool whose unaligned/aligned outputs serve as input to the metric
    computation
- name: enviPat
  role: Predicts isotopologues and resolution lists used during benchmark generation
    feeding the metrics
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btab231
    title: 'mzRAPP: a tool for reliability assessment of data pre-processing in non-targeted
      metabolomics'
  source_content_hash: sha256:d0001cbfa38e6f3cb6d5060136ba3385b265d8ce04e4a791c38ee0a9a3bf5527
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YasinEl/mzRAPP
  tools:
  - R
  - XCMS
  - boot
  - enviPat
  - mzRAPP
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# lc-hrms-preprocessing-metric-computation

## Summary

Compute NPP (non-targeted data pre-processing) performance metrics by matching a validated benchmark of curated isotopologue peaks against pre-processing tool outputs (e.g. XCMS, MZmine 2, MS-DIAL), then derive per-molecule, per-stage metrics such as found-peak fraction, split-peak percentage, degenerated isotopologue ratios, and alignment loss.

## When to use

Use when you have LC-HRMS mzML files, a benchmark of target molecules with known retention-time boundaries and sum formulas, and NPP outputs (e.g. XCMS aligned/unaligned CSVs) whose reliability you need to quantify per pre-processing stage.

## When NOT to use

- You have no benchmark of target molecules with known sum formulas and retention-time boundaries — mzRAPP requires user-provided targets from which enviPat can predict isotopologues
- Your data are targeted MRM/SRM peak areas only, not untargeted NPP outputs (feature tables from XCMS/MZmine/MS-DIAL) to be matched against the benchmark
- Data are low-resolution MS or non-LC acquisitions — the skill is scoped to liquid chromatography high-resolution mass spectrometry (LC-HRMS) with centroided mzML input

## Inputs

- Centroided mzML files of the LC-HRMS run (e.g. *_POS.mzML from MTBLS267)
- Benchmark dataset CSV of validated isotopologue peaks (Benchmark.csv) built from a target file with molecule, SumForm_c, main_adduct, user.rtmin/user.rtmax columns
- NPP tool output CSVs (e.g. XCMS_unaligned_run1.csv, XCMS_aligned_run1.csv, MZmine 2 output)
- Sample-group CSV (sample_name, sample_group) for benchmark generation

## Outputs

- Per-molecule, per-stage performance metrics (found-peak fraction, split-peak %, degenerated isotopologue ratio %, alignment loss %, missing peak/value classes)
- Bootstrap confidence-interval table (metric, stage, estimate, ci_lower, ci_upper) at alpha = 0.95
- mzRAPP reliability-assessment report comparing benchmark vs NPP output

## How to apply

Build a benchmark with mzRAPP by extracting peaks for all enviPat-predicted isotopologues of target molecules, rejecting isotopologues whose area/height deviate more than 30% from prediction or whose Pearson correlation with the most abundant isotopologue is below 0.85, and keeping only entries with the most-abundant plus at least one additional isotopologue (parameters: lowest isotopologue 0.05, min 6 scans per peak, 6 ppm mz precision, 5 ppm mz accuracy). Export the NPP tool output (e.g. XCMS unaligned and aligned run CSVs) and match its peaks against the benchmark (Benchmark.csv) using mzRAPP's matching functions (match_peaks_to_benchmark/compare_peaks). Compute stage-wise performance metrics with derive_performance_metrics(comparison_data): found/not-found peak fraction, split-peak percentage, missing peak/value classification, degenerated isotopologue ratios, and alignment error counting. Because each metric is computed over a finite set of benchmark molecules (e.g. ~47), quantify reliability by bootstrapping over molecules with the R boot package: resample molecules with replacement, recompute each metric per replicate, and report percentile confidence intervals at alpha = 0.95 (2.5th and 97.5th percentiles). Report, per metric and per pre-processing stage, the point estimate plus ci_lower/ci_upper (e.g. 83-94% detected peaks and 28-53% degenerated isotopologue ratios post-alignment in the paper's example).

## Related tools

- **mzRAPP** (Generates/validates the benchmark, matches NPP outputs against it, and computes the NPP performance metrics) — https://github.com/YasinEl/mzRAPP
- **boot** (R package used to draw resamples of benchmark molecules and derive percentile confidence intervals for each metric)
- **XCMS** (Example NPP tool whose unaligned/aligned outputs serve as input to the metric computation)
- **enviPat** (Predicts isotopologues and resolution lists used during benchmark generation feeding the metrics)

## Examples

```
devtools::install_github("YasinEl/mzRAPP", dependencies = TRUE, build_vignettes = TRUE); vignette("Vignette_mzRAPP_Example_workflow")
```

## Evaluation signals

- Every metric's 95% bootstrap interval brackets its point estimate (2.5th/97.5th percentile CIs) and overlaps the variability ranges reported in the vignette (some randomness from resampling is expected)
- Reproduced example ranges such as 83-94% detected peaks and 28-53% degenerated isotopologue ratios in post-alignment
- Benchmark QC: only isotopologues with the most-abundant plus at least one additional isotopologue survive; peaks >30% off predicted area/height or with Pearson correlation < 0.85 are removed
- Script runs end-to-end on deposited inputs (Benchmark.csv plus XCMS run-1 CSVs) and writes bootstrap_ci_metrics.csv with columns metric, stage, estimate, ci_lower, ci_upper

## Limitations

- Metrics are computed over a limited set of benchmark molecules (~47 in the example), so bootstrap intervals can be wide and sensitive to resampling
- The paper describes the bootstrap CI mechanism as part of the workflow without separately reporting validation results for it
- If the main_adduct of a molecule is not detected, other adducts are not accepted; benchmark peaks are rejected when isotopic information does not fit
- Only metrics for the supported pre-processing tools (five of the most commonly used) can be computed directly; other tools need exported outputs in a matching format

## Evidence

- [abstract] mzRAPP produces several performance metrics for different steps of the pre-processing workflow: "mzRAPP produces several performance metrics for different steps of the pre-processing workflow"
- [readme] The resulting benchmark dataset is used to extract different performance metrics for NPP performed on the same mzML files.: [evidence span withheld — anchor status: external_doc]
- [readme] Isotopologue peaks with an area or height which is more than 30% off the predicted value or a Pearson Correlation coef < 0.85 (as compared to the highest isotopologue) are removed.: [evidence span withheld — anchor status: not_found]
- [other] mzRAPP derives empirical confidence intervals (alpha = 0.95) for all of its NPP performance metrics by bootstrapping over the supplied molecules: [evidence span withheld — anchor status: not_found]
- [readme] If you want to go through some examples on how to use mzRAPP and interpret results you can go through some use-cases: [evidence span withheld — anchor status: external_doc]
