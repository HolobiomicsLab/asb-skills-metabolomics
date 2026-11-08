---
name: time-course-concentration-inference-evaluation
description: Use when evaluating an inference method for absolute metabolite concentrations
  from relative abundance time-course data where no published comparator exists, so
  negative controls (random / constant response factors) must be constructed and scored.
when_to_use_negative:
- A published competing method for response-factor inference exists — use a direct
  head-to-head comparison instead of negative controls
- The input data are already absolute concentrations rather than relative abundance
  / response-factor outputs that need accuracy scoring
- The system is a multi-compartment eukaryotic model where MetaboPAC-style mass-balance
  inference (and thus this evaluation) does not apply
edam_operation: http://edamontology.org/operation_2424
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3272
tools:
- name: MetaboPAC
  role: Source of the inferred response factors being evaluated; its runs over repetitions
    [1:20] and percKnownKinetics values [0 20 40 60 80 100] provide the predictions
    compared against controls
  repo: https://github.com/gtStyLab/MetaboPAC
- name: MATLAB
  role: Environment for implementing the negative-control baselines (random and constant
    response factors) and computing accuracy fractions within log2 error tolerances
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1039/d2mo00168c
    title: Towards inferring absolute concentrations from relative abundance in time-course
      GC-MS metabolomics data
  source_content_hash: sha256:fac208f944148e317b325cf93a72f36bb43f141fa0f2353ebb99c05dd2897918
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/gtStyLab/MetaboPAC
  tools:
  - MATLAB
  - MetaboPAC
  edam_operation: http://edamontology.org/operation_2424
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# time-course-concentration-inference-evaluation

## Summary

Benchmark a time-course GC-MS concentration-inference method (e.g., MetaboPAC) against negative-control baselines — random response factors and a constant response factor of 500 — by scoring the fraction of inferred response factors within fixed log2 error tolerances across repetitions and known-kinetics percentages.

## When to use

Use when evaluating an inference method for absolute metabolite concentrations from relative abundance time-course data where no published comparator exists, so negative controls (random / constant response factors) must be constructed and scored.

## When NOT to use

- A published competing method for response-factor inference exists — use a direct head-to-head comparison instead of negative controls
- The input data are already absolute concentrations rather than relative abundance / response-factor outputs that need accuracy scoring
- The system is a multi-compartment eukaryotic model where MetaboPAC-style mass-balance inference (and thus this evaluation) does not apply

## Inputs

- Inferred response factors from the method under test (e.g., MetaboPAC runs over repetitions [1:20] and percKnownKinetics [0 20 40 60 80 100])
- True response factors from trueRF (randomly generated sets, 20 replicates per system)
- Optionally noise-added runs from MetaboPAC_*_noisy.m at timepoints [50 15], CoV [5 15]

## Outputs

- Accuracy-fraction table comparing MetaboPAC vs. baseline_random_rf vs. baseline_500_rf (e.g., baseline_rf_comparison.csv)
- Per-tolerance accuracy fractions at log2(1.1), log2(1.3), log2(1.5) error tolerances
- Significance assessment of method vs. controls (e.g., outperformance at 20% known kinetics)

## How to apply

Load the inferred response factors and the true response factors from completed runs across repetitions [1:20] and percKnownKinetics values of [0 20 40 60 80 100]. Build two negative controls in MATLAB: baseline_random_rf with response factors sampled uniformly between 1 and 1000, and baseline_500_rf with every response factor fixed to 500 — both mirroring how MetaboPAC's true response factors are sampled (uniform 1–1000; a log-uniform variant is handled by MetaboPAC_*_log.m). For each model, compute the fraction of predicted response factors falling within log2(1.1), log2(1.3), and log2(1.5) of the true values, tabulating accuracy fractions across all error tolerances and repetitions (e.g., baseline_rf_comparison.csv). Judge success by confirming the method of interest outperforms both controls significantly — in the paper, MetaboPAC significantly beat the controls when 20% of kinetic terms were known a priori.

## Related tools

- **MetaboPAC** (Source of the inferred response factors being evaluated; its runs over repetitions [1:20] and percKnownKinetics values [0 20 40 60 80 100] provide the predictions compared against controls) — https://github.com/gtStyLab/MetaboPAC
- **MATLAB** (Environment for implementing the negative-control baselines (random and constant response factors) and computing accuracy fractions within log2 error tolerances)

## Examples

```
run MetaboPAC_*.m for repetitions [1:20] and percKnownKinetics [0 20 40 60 80 100], then tabulate accuracy fractions for MetaboPAC vs baseline_random_rf (uniform 1–1000) vs baseline_500_rf into baseline_rf_comparison.csv
```

## Evaluation signals

- The negative-control definitions match the paper exactly: baseline_random_rf samples uniformly between 1 and 1000; baseline_500_rf fixes all response factors to 500
- Accuracy is computed as fraction of predicted response factors within log2(1.1), log2(1.3), and log2(1.5) of true values — three fixed tolerances, not a continuous metric
- Comparison spans all repetitions [1:20] and percKnownKinetics values [0 20 40 60 80 100]; results are tabulated per tolerance and repetition
- The method under test significantly outperforms both controls; check the headline condition (significant outperformance when 20% of kinetic terms are known a priori) is reproduced
- Output file baseline_rf_comparison.csv contains accuracy fractions for all three models across tolerances and repetitions

## Limitations

- Penalty-based inference can yield multiple response-factor sets with similar or smaller penalty values than the true ones, so accuracy fractions may not fully reflect inference quality without penalty refinement
- Accuracy varies widely (40% to nearly 100%) depending on metabolic network, data quality (noise, sampling frequency), and error tolerance
- High noise levels violate mass balances and complicate assessment; missing values (~10%) with imputation reduce accuracy of inferred concentrations
- Response factors are assumed constant with concentration; linear dynamic range or ion suppression may violate this assumption
- MetaboPAC (the evaluated method) assumes all metabolites are directly connected via reactions in a single intracellular compartment and does not consider transport reactions across compartments

## Evidence
