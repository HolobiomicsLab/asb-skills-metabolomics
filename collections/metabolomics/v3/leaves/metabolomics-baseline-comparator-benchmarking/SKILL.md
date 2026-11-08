---
name: metabolomics-baseline-comparator-benchmarking
description: Use when evaluating a response-factor inference method for relative-abundance
  metabolomics where no external published competitor exists, and negative-control
  baselines must substitute for published-method comparison.
when_to_use_negative:
- A published competitor method for response-factor inference exists — head-to-head
  comparison against that method should be used instead of negative controls
- The metabolomics dataset is not time-course relative-abundance GC-MS data with an
  associated kinetic model, so mass-balance-based inference (and its comparison to
  random/constant baselines) is inapplicable
- The evaluation target is metabolite concentration accuracy rather than response-factor
  accuracy — the log2-fold tolerance metric must be recomputed on inferred concentrations,
  not response factors
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3520
tools:
- name: MetaboPAC
  role: Source of the inferred response factors being benchmarked; runs are executed
    in MATLAB over repetitions [1:20] and percKnownKinetics values [0 20 40 60 80
    100]
  repo: https://github.com/gtStyLab/MetaboPAC
- name: MATLAB
  role: Execution environment for running MetaboPAC_*.m scripts and assembling the
    baseline comparator accuracy table
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1039/d2mo00168c
    title: Towards inferring absolute concentrations from relative abundance in time-course
      GC-MS metabolomics data
  source_content_hash: sha256:e0a637ced35d271ad3f41c13fa1371b6afb2f87f773e3aa694c89d2503e61128
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/gtStyLab/MetaboPAC
  tools:
  - MATLAB
  - MetaboPAC
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# Metabolomics baseline comparator benchmarking

## Summary

A benchmarking skill for MetaboPAC response-factor inference: construct negative-control baselines (random response factors sampled uniformly from 1–1000, and a constant response factor of 500) and compare prediction accuracy against the inferred response factors at fixed error tolerances.

## When to use

Use when evaluating a response-factor inference method for relative-abundance metabolomics where no external published competitor exists, and negative-control baselines must substitute for published-method comparison.

## When NOT to use

- A published competitor method for response-factor inference exists — head-to-head comparison against that method should be used instead of negative controls
- The metabolomics dataset is not time-course relative-abundance GC-MS data with an associated kinetic model, so mass-balance-based inference (and its comparison to random/constant baselines) is inapplicable
- The evaluation target is metabolite concentration accuracy rather than response-factor accuracy — the log2-fold tolerance metric must be recomputed on inferred concentrations, not response factors

## Inputs

- Inferred response factors from MetaboPAC runs (results folder outputs for repetitions [1:20] and percKnownKinetics [0 20 40 60 80 100])
- True response factors from trueRF folder (20 replicate sets per system, sampled uniformly between 1 and 1000)

## Outputs

- baseline_rf_comparison.csv: tabulated accuracy fractions (fraction of response factors within log2(1.1), log2(1.3), log2(1.5) of true response factors) for MetaboPAC, baseline_random_rf, and baseline_500_rf across repetitions and error tolerances

## How to apply

Load the inferred response factors and the true response factors (sampled uniformly between 1 and 1000, 20 replicate sets per system) from completed MetaboPAC runs across repetitions [1:20] and percKnownKinetics values [0 20 40 60 80 100]. Build two negative controls: baseline_random_rf by sampling response factors uniformly between 1 and 1000, and baseline_500_rf by fixing every response factor to 500. For each model, compute the fraction of predicted response factors falling within log2(1.1), log2(1.3), and log2(1.5) of the true response factors — i.e. within a constant multiplicative error factor of 1.1, 1.3, or 1.5. Tabulate accuracy fractions across all error tolerances and repetitions in baseline_rf_comparison.csv. The rationale is that MetaboPAC is the first computational platform for standard-free inference of absolute concentrations, so no published method exists as a direct comparator; random and constant response factors serve as negative controls to demonstrate that mass-balance constraints produce inference significantly better than chance.

## Related tools

- **MetaboPAC** (Source of the inferred response factors being benchmarked; runs are executed in MATLAB over repetitions [1:20] and percKnownKinetics values [0 20 40 60 80 100]) — https://github.com/gtStyLab/MetaboPAC
- **MATLAB** (Execution environment for running MetaboPAC_*.m scripts and assembling the baseline comparator accuracy table)

## Examples

```
matlab -batch "run('MetaboPAC_noiseless.m'); rf_true = load('trueRF/randomRF_1.mat'); tol = log2([1.1 1.3 1.5]); acc = mean(abs(log2(inferredRF./trueRF)) <= tol); writematrix(acc,'baseline_rf_comparison.csv')"
```

## Evaluation signals

- MetaboPAC's accuracy fraction is significantly higher than both baseline_random_rf and baseline_500_rf when percKnownKinetics = 20 (matching the reported significant outperformance at 20% known kinetics)
- Accuracy fractions are computed as the fraction of response factors within log2(1.1), log2(1.3), and log2(1.5) of true response factors — verify all three tolerance columns present per model
- baseline_500_rf accuracy must be identical across all tolerance levels for a given true-RF set if true response factors are uniformly distributed relative to 500 — sanity-check the table for implausible uniformity
- baseline_rf_comparison.csv contains rows spanning all 20 repetitions and all 6 percKnownKinetics values, with three model columns

## Limitations

- Accuracy varies widely (40% to nearly 100%) depending on metabolic network, data quality, and error tolerance, so single-tolerance summaries can be misleading
- Multiple sets of response factors had similar or smaller penalty values than the true ones, so even successful benchmarking does not guarantee the inferred set is uniquely correct
- High noise levels violate mass balances and complicate feasibility assessment, degrading both MetaboPAC and baseline accuracy comparability
- Negative-control comparison validates relative performance only; it does not establish that MetaboPAC reaches practically useful absolute accuracy at low percKnownKinetics

## Evidence

- [methods] Accuracy metric definition: [evidence span withheld — anchor status: not_found]
- [results] Significant outperformance condition: [evidence span withheld — anchor status: not_found]
