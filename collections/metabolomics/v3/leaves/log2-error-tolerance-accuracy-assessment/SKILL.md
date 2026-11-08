---
name: log2-error-tolerance-accuracy-assessment
description: Use when evaluating accuracy of inferred response factors or absolute
  concentrations against known true values, particularly for comparing MetaboPAC against
  negative control baselines like random or constant response factors.
when_to_use_negative:
- Input data is relative abundance measurements without any known true response factors
  for comparison — the skill requires ground truth to compute error tolerances
- Absolute concentrations were already determined by chemical standards, making response
  factor inference and its accuracy assessment unnecessary
- The metabolic system has transport reactions across intracellular compartments or
  multiple compartments, as MetaboPAC itself does not apply in those settings
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: MetaboPAC
  role: Source of inferred response factors to be evaluated using the log2 error tolerance
    framework
  repo: gtStyLab/MetaboPAC
- name: MATLAB
  role: Implementation environment for computing accuracy fractions across error tolerances
    and generating the baseline_rf_comparison.csv output
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1039/d2mo00168c
    title: Towards inferring absolute concentrations from relative abundance in time-course
      GC-MS metabolomics data
  source_content_hash: sha256:3af7d49a74cb5cb7a588cd449e7e7d08f571d88291842d94737f4b8fefd1ea64
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/gtStyLab/MetaboPAC
  tools:
  - MATLAB
  - MetaboPAC
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# log2-error-tolerance-accuracy-assessment

## Summary

Assesses accuracy of inferred metabolite response factors by computing the fraction of predictions falling within log2(1.1), log2(1.3), and log2(1.5) multiplicative error tolerances of true values, enabling model comparison for MetaboPAC.

## When to use

Use when evaluating accuracy of inferred response factors or absolute concentrations against known true values, particularly for comparing MetaboPAC against negative control baselines like random or constant response factors.

## When NOT to use

- Input data is relative abundance measurements without any known true response factors for comparison — the skill requires ground truth to compute error tolerances
- Absolute concentrations were already determined by chemical standards, making response factor inference and its accuracy assessment unnecessary
- The metabolic system has transport reactions across intracellular compartments or multiple compartments, as MetaboPAC itself does not apply in those settings

## Inputs

- Inferred response factors from MetaboPAC (from runs at repetitions [1:20] and percKnownKinetics [0 20 40 60 80 100])
- True response factors (randomly generated sets of 20 replicate sets per system from trueRF folder)
- Negative control response factor sets: baseline_random_rf (sampled uniformly between 1 and 1000) and baseline_500_rf (constant 500 for every response factor)

## Outputs

- baseline_rf_comparison.csv: table of accuracy fractions across error tolerances (log2(1.1), log2(1.3), log2(1.5)) and repetitions for MetaboPAC and both negative control baselines
- Binary comparison verdict: whether MetaboPAC performs significantly better than the negative control approaches (result_better_than_controls_20pct)

## How to apply

For each model being compared (MetaboPAC, baseline_random_rf, baseline_500_rf), compute the fraction of predicted response factors falling within log2(1.1), log2(1.3), and log2(1.5) of the true response factors. This log2-based error tolerance quantifies how close each predicted response factor is to its true value on a multiplicative scale — log2(1.1) means the prediction is within ~10% of the true factor, log2(1.3) within ~30%, and log2(1.5) within ~50%. Load inferred response factors from completed runs across repetitions [1:20] and percKnownKinetics values of [0 20 40 60 80 100], then tabulate accuracy fractions across all error tolerances and repetitions in baseline_rf_comparison.csv. This approach was specifically needed because no other published approach exists for inferring response factors for arbitrary metabolites, so negative controls (uniformly random response factors between 1 and 1000, and a constant response factor of 500) serve as the comparison baselines.

## Related tools

- **MetaboPAC** (Source of inferred response factors to be evaluated using the log2 error tolerance framework) — gtStyLab/MetaboPAC
- **MATLAB** (Implementation environment for computing accuracy fractions across error tolerances and generating the baseline_rf_comparison.csv output)

## Examples

```
load('results/baseline_rf_comparison.csv'); % compute fraction within log2(1.1), log2(1.3), log2(1.5) of trueRF across repetitions [1:20]
```

## Evaluation signals

- For each repetition and percKnownKinetics level, verify that MetaboPAC's accuracy fraction at each tolerance (log2(1.1), log2(1.3), log2(1.5)) is consistently higher than baseline_random_rf and baseline_500_rf
- Confirm the key validation signal: MetaboPAC performs significantly better than the two negative control approaches when 20% of kinetic terms are known a priori (result_better_than_controls_20pct)
- Check that baseline_rf_comparison.csv contains accuracy fractions for all three models across all error tolerances and all 20 repetitions
- Verify negative controls are correctly constructed: baseline_random_rf values are sampled uniformly between 1 and 1000, and baseline_500_rf is fixed at 500 for every response factor
- Prediction accuracy should fall in a plausible range — the paper notes different networks/data quality/tolerances can lead to accuracy from 40% to nearly 100%, so extreme values outside this range warrant investigation

## Limitations

- Accuracy varies widely (40% to nearly 100%) depending on the metabolic network, data quality, and error tolerance chosen, so results are not generalizable across systems without context
- Multiple sets of response factors can have similar or even smaller penalty values than the true response factors, suggesting penalty functions need refinement before accuracy assessment is fully informative
- High noise levels in data can lead to violations of mass balances and thus complicate assessment of biological feasibility, degrading accuracy measurement reliability
- MetaboPAC assumes all metabolites are directly connected via reactions in a single compartment and does not consider transport reactions across intracellular compartments in eukaryotic systems
- Response factors are assumed constant with concentration; linear dynamic range or ion suppression may violate this assumption and skew accuracy metrics

## Evidence

- [other] Log2 error tolerance accuracy assessment: [evidence span withheld — anchor status: not_found]
