---
name: response-factor-negative-control-simulation
description: Decision support for validating response factor inference on GC-MS relative
  abundance data when no published comparator method exists, by simulating random
  and constant response factor baselines.
when_to_use_negative:
- MetaboPAC has already been shown to outperform these baselines in the manuscript
  — the comparison only serves as validation, not as a primary analysis of absolute
  concentration inference.
- The dataset contains known kinetic equations for all metabolites; negative controls
  are only needed when no a priori kinetic information exists (percKnownKinetics =
  0).
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3520
tools:
- name: MATLAB
  role: Environment for running MetaboPAC and simulating the random and constant response
    factor baselines across repetitions and percKnownKinetics settings
  repo: https://github.com/gtStyLab/MetaboPAC
- name: MetaboPAC
  role: Produces the inferred response factors that are compared against the random
    and constant 500 baselines; the generic entry point is MetaboPAC_generic.m
  repo: https://github.com/gtStyLab/MetaboPAC
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1039/d2mo00168c
    title: Towards inferring absolute concentrations from relative abundance in time-course
      GC-MS metabolomics data
  source_content_hash: sha256:0d85317c3b68c221667bd6db0a4e0f95c6e3b9bdaf8540e320da5b82d0fdaa46
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/gtStyLab/MetaboPAC
  tools:
  - MATLAB
  - MetaboPAC
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# response-factor-negative-control-simulation

## Summary

Generates negative-control baselines for MetaboPAC's response factor inference — random response factors sampled uniformly between 1 and 1000, and a constant response factor of 500 — so that MetaboPAC's accuracy can be judged against naive baselines.

## When to use

Decision support for validating response factor inference on GC-MS relative abundance data when no published comparator method exists, by simulating random and constant response factor baselines.

## When NOT to use

- MetaboPAC has already been shown to outperform these baselines in the manuscript — the comparison only serves as validation, not as a primary analysis of absolute concentration inference.
- The dataset contains known kinetic equations for all metabolites; negative controls are only needed when no a priori kinetic information exists (percKnownKinetics = 0).

## Inputs

- MetaboPAC inferred response factors from completed runs across repetitions [1:20] and percKnownKinetics [0 20 40 60 80 100]
- True response factors from trueRF folder (20 replicate sets per system, sampled uniformly between 1 and 1000)

## Outputs

- baseline_rf_comparison.csv tabulating accuracy fractions within log2(1.1), log2(1.3), log2(1.5) error tolerances for MetaboPAC vs random RF vs constant 500 RF baselines

## How to apply

First load MetaboPAC's inferred response factors and the true response factors (sampled uniformly between 1 and 1000) from completed runs across repetitions [1:20] and percKnownKinetics values of [0 20 40 60 80 100]. Then generate the random-response-factor baseline by sampling response factors uniformly between 1 and 1000, and the constant-response-factor baseline by fixing every response factor to 500. For each of the three models (MetaboPAC, random RF, constant 500 RF), compute the fraction of predicted response factors falling within log2(1.1), log2(1.3), and log2(1.5) of the true values. Tabulate accuracy fractions across all error tolerances and repetitions in baseline_rf_comparison.csv. The rationale is that since no other published approach exists for inferring response factors for arbitrary metabolites, naive baselines are the only fair comparator for significance testing.

## Related tools

- **MATLAB** (Environment for running MetaboPAC and simulating the random and constant response factor baselines across repetitions and percKnownKinetics settings) — https://github.com/gtStyLab/MetaboPAC
- **MetaboPAC** (Produces the inferred response factors that are compared against the random and constant 500 baselines; the generic entry point is MetaboPAC_generic.m) — https://github.com/gtStyLab/MetaboPAC

## Examples

```
MATLAB: run MetaboPAC_*.m for repetitions [1:20] and percKnownKinetics [0 20 40 60 80 100], then compare against baseline_random_rf and baseline_500_rf baselines to populate baseline_rf_comparison.csv.
```

## Evaluation signals

- MetaboPAC's accuracy fractions within log2(1.1), log2(1.3), and log2(1.5) error tolerances must exceed those of both the random RF and constant 500 RF baselines, particularly when percKnownKinetics = 20%.
- The random RF baseline should show near-chance accuracy given uniform sampling between 1 and 1000, while the constant 500 RF baseline should perform no better than random across error tolerances.
- The comparison must be run across the same repetitions [1:20] and percKnownKinetics values [0 20 40 60 80 100] used for MetaboPAC runs to ensure fair paired comparison.
- baseline_rf_comparison.csv must contain accuracy fraction values for all three models across all error tolerances and repetitions.

## Limitations

- MetaboPAC's inference accuracy is bounded by the accuracy of the flux calculation approaches used, so baseline comparisons reflect that upstream uncertainty.
- Penalty functions may not discriminate well: multiple sets of response factors had similar or even smaller penalty values than the true response factors, suggesting the penalties need refinement.
- Prediction accuracy varies widely (40% to nearly 100%) depending on the metabolic network, data quality, and error tolerance, so a single baseline comparison may not generalize across systems.

## Evidence

- [methods] Baseline accuracy computed as fraction within log2(1.1), log2(1.3), log2(1.5) error tolerances: [evidence span withheld — anchor status: not_found]
