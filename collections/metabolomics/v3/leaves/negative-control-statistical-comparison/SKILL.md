---
name: negative-control-statistical-comparison
description: Use when you must show that a computational inference method (e.g., MetaboPAC's
  heuristic penalty-function optimization of response factors) significantly outperforms
  negative control methods across graded conditions such as percKnownKinetics = [0
  20 40 60 80 100].
when_to_use_negative:
- Data are noisy or sampled at low frequency (timepoints [50 15], CoV [5 15]) — the
  noise-add workflow MetaboPAC_*_noisy.m is the appropriate comparator setting, not
  the noiseless benchmark
- Response factors were sampled from a log-uniform distribution — use MetaboPAC_*_log.m
  at percKnownKinetics = 100 instead of the uniform noiseless protocol
- The goal is calibration/validation of a single method's absolute accuracy rather
  than a head-to-head significance comparison against negative controls
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MetaboPAC
  role: 'Method under test: its heuristic soft-constraint penalty-function optimization
    infers response factors whose accuracy is compared against negative control comparator
    methods'
  repo: https://github.com/gtStyLab/MetaboPAC
- name: MATLAB
  role: Execution environment for MetaboPAC_*.m runs, driver_genDatasets*.m data generation,
    and aggregation/statistical comparison scripts
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1039/d2mo00168c
    title: Towards inferring absolute concentrations from relative abundance in time-course
      GC-MS metabolomics data
  source_content_hash: sha256:e0d5f3d6d801a534d535aa766f0a1c7b25b130d17eb9347379d05cdccc55c8c5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/gtStyLab/MetaboPAC
  tools:
  - MATLAB
  - MetaboPAC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# negative-control-statistical-comparison

## Summary

Benchmark an inference method's output against negative control comparator methods using predefined error-tolerance bins and per-condition statistical tests, as done for MetaboPAC's response-factor inference on noiseless GC-MS-derived in silico data.

## When to use

Use when you must show that a computational inference method (e.g., MetaboPAC's heuristic penalty-function optimization of response factors) significantly outperforms negative control methods across graded conditions such as percKnownKinetics = [0 20 40 60 80 100].

## When NOT to use

- Data are noisy or sampled at low frequency (timepoints [50 15], CoV [5 15]) — the noise-add workflow MetaboPAC_*_noisy.m is the appropriate comparator setting, not the noiseless benchmark
- Response factors were sampled from a log-uniform distribution — use MetaboPAC_*_log.m at percKnownKinetics = 100 instead of the uniform noiseless protocol
- The goal is calibration/validation of a single method's absolute accuracy rather than a head-to-head significance comparison against negative controls

## Inputs

- Noiseless relative-abundance time-course matrices (concMatrix) with measurement times (timeVec) generated via GenData/driver_genDatasets*.m for each of the four metabolic models
- True response factor sets (20 replicate sets per system, uniform 1–1000) from the trueRF folder
- Negative control comparator results at each percKnownKinetics level

## Outputs

- Per-run response-factor error-tolerance bin fractions (log2(1.1), log2(1.3), log2(1.5)) for method and negative controls
- Aggregated fractions table heuristic_noiseless_rf_error_bins.csv (per model, per percKnownKinetics level)
- Statistical comparison results of MetaboPAC vs negative controls at each percKnownKinetics level

## How to apply

Generate or load noiseless relative-abundance time-course data (concMatrix with timeVec) for each metabolic model using the first cell of GenData/driver_genDatasets*.m in MATLAB. Run the method of interest (MetaboPAC_*.m) and the negative control comparators for repetitions [1:20] and percKnownKinetics values of [0 20 40 60 80 100]. For each run, compare inferred response factors to the true response factors (20 randomly generated replicate sets per system, uniform between 1 and 1000) and compute the fraction falling within error tolerance bins of log2(1.1), log2(1.3), and log2(1.5). Aggregate bin fractions across the 20 repetitions for each model and known-kinetics fraction, then statistically compare the method's bin fractions against the negative controls at each percKnownKinetics level. Write the aggregated fractions table to heuristic_noiseless_rf_error_bins.csv and verify the trend that significant improvement over negative controls emerges when at least 20% of kinetic terms are known a priori.

## Related tools

- **MetaboPAC** (Method under test: its heuristic soft-constraint penalty-function optimization infers response factors whose accuracy is compared against negative control comparator methods) — https://github.com/gtStyLab/MetaboPAC
- **MATLAB** (Execution environment for MetaboPAC_*.m runs, driver_genDatasets*.m data generation, and aggregation/statistical comparison scripts)

## Examples

```
matlab -batch "MetaboPAC_1"  % run MetaboPAC_*.m for repetitions [1:20], percKnownKinetics [0 20 40 60 80 100], then aggregate bin fractions to heuristic_noiseless_rf_error_bins.csv
```

## Evaluation signals

- Across all four tested models on noiseless data, MetaboPAC's bin fractions are significantly better than negative control comparator methods (reproducing the Figure S5–S6 trend)
- Significant improvement over negative controls emerges only when at least 20% of kinetic terms are known a priori, not at percKnownKinetics = 0
- Fractions of inferred response factors fall within error tolerances of log2(1.1), log2(1.3), and log2(1.5), with accuracy varying by model, data quality, and tolerance from ~40% to nearly 100%
- Results are aggregated over repetitions [1:20] for each of percKnownKinetics = [0 20 40 60 80 100], giving stable per-bin fraction estimates
- The aggregated output file heuristic_noiseless_rf_error_bins.csv is produced with one row per model/known-kinetics fraction for both method and negative controls

## Limitations

- Multiple sets of response factors had similar or even smaller penalty values than the true response factors, suggesting the heuristic penalties need refinement
- Accuracy varies widely (40% to nearly 100%) depending on the metabolic network, aspects of data quality, and error tolerance
- High noise levels in the data can violate mass balances and complicate assessment of response-factor feasibility, undermining the comparison
- Negative-control-relative performance was established under noiseless, uniform-RF, single-compartment intracellular assumptions; real GC-MS data with missing values, nonlinear response factors, or multi-compartment eukaryotic systems fall outside this validation

## Evidence
