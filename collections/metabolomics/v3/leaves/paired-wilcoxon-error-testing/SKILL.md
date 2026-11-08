---
name: paired-wilcoxon-error-testing
description: Use when two or more imputation methods have been run on the same masked
  datasets across repeated simulations (e.g. 10–30% MCAR values), yielding paired
  per-simulation error rates (NRMSE/PFC) that need significance testing against a
  reference method.
when_to_use_negative:
- 'MICE results on the Children''s Hospital data: nearly dependent variables were
  removed and imputation was ad-hoc, so no significance statements should be reported'
- Comparisons where no results exist for a method (e.g. MissPALasso on the high-dimensional
  p=12,626 Insulin data, where computation is infeasible) — there are no paired error
  rates to test
- Unpaired or independently sampled error rates from non-identical masking designs;
  the test requires paired observations from the same simulations
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: missForest
  role: Reference method whose per-simulation NRMSE/PFC error rates serve as one side
    of every paired Wilcoxon comparison
  repo: http://stat.ethz.ch/CRAN/
- name: KNNimpute
  role: Competing method whose per-simulation error rates are tested against missForest's
    in the paired Wilcoxon test
- name: MissPALasso
  role: Competing method whose per-simulation error rates are tested against missForest's
    (skipped for the Insulin data where computation is infeasible)
- name: MICE
  role: Competing method for categorical and mixed-type data whose PFC values are
    tested against missForest's, except on Children's Hospital data where no significance
    is reported
- name: R
  role: Statistical software environment in which the paired Wilcoxon tests are implemented
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btr597
    title: MissForest—non-parametric missing value imputation for mixed-type data
  source_content_hash: sha256:bb68aa85afc0e5654c0777e983762bd636e0eb888905f28a34107584ebe985b9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - KNNimpute
  - MICE
  - MissPALasso
  - R
  - missForest
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# paired-wilcoxon-error-testing

## Summary

A non-parametric paired Wilcoxon significance test comparing per-simulation imputation error rates (NRMSE for continuous, PFC for categorical entries) of each baseline method against missForest across the 50 independent simulations (10 for the high-dimensional Insulin dataset). It establishes whether missForest's error reductions are statistically significant rather than simulation noise.

## When to use

Use when two or more imputation methods have been run on the same masked datasets across repeated simulations (e.g. 10–30% MCAR values), yielding paired per-simulation error rates (NRMSE/PFC) that need significance testing against a reference method.

## When NOT to use

- MICE results on the Children's Hospital data: nearly dependent variables were removed and imputation was ad-hoc, so no significance statements should be reported
- Comparisons where no results exist for a method (e.g. MissPALasso on the high-dimensional p=12,626 Insulin data, where computation is infeasible) — there are no paired error rates to test
- Unpaired or independently sampled error rates from non-identical masking designs; the test requires paired observations from the same simulations

## Inputs

- Per-simulation NRMSE values (continuous missing entries) for missForest and each competitor, paired by simulation index across 50 independent runs (10 for the Insulin dataset)
- Per-simulation PFC values (categorical missing entries) for missForest, MICE, and dummy-coded KNNimpute, paired by simulation index
- Dataset–missingness condition labels (10%, 20%, 30% values removed completely at random) identifying which pairs belong together

## Outputs

- Paired Wilcoxon test outcomes (significance flags) for each compared method versus missForest, per dataset and missingness condition
- Aggregated mean errors with significance outcomes in imputation_comparison.csv
- Stratified relative error reductions with significance flags in missforest_gain_summary.csv

## How to apply

Collect the per-simulation error rates produced in step 6 of the benchmark workflow: NRMSE over the continuous missing entries and PFC over the categorical missing entries for every method and simulation. Because each simulation applies the same artificial masking (values removed completely at random at 10%, 20% or 30%) to all methods, the 50 error-rate pairs per dataset-condition are dependent, so a paired (not unpaired) Wilcoxon signed-rank test is used. For each competitor (KNNimpute, MissPALasso, MICE, dummy-coded KNNimpute), test that method's error rates versus missForest's error rates and record the test outcome per comparison. Aggregate mean errors and the significance outcomes across simulations into imputation_comparison.csv, and propagate the significance flags into missforest_gain_summary.csv when ranking relative error reductions. Skip significance statements where the comparison itself was ad-hoc (MICE on the Children's Hospital data).

## Related tools

- **missForest** (Reference method whose per-simulation NRMSE/PFC error rates serve as one side of every paired Wilcoxon comparison) — http://stat.ethz.ch/CRAN/
- **KNNimpute** (Competing method whose per-simulation error rates are tested against missForest's in the paired Wilcoxon test)
- **MissPALasso** (Competing method whose per-simulation error rates are tested against missForest's (skipped for the Insulin data where computation is infeasible))
- **MICE** (Competing method for categorical and mixed-type data whose PFC values are tested against missForest's, except on Children's Hospital data where no significance is reported)
- **R** (Statistical software environment in which the paired Wilcoxon tests are implemented)

## Examples

```
wilcox.test(errors_competitor, errors_missForest, paired = TRUE)  # per dataset-condition, on NRMSE or PFC vectors
```

## Evaluation signals

- Every comparison must use paired error vectors of equal length (50 per dataset-condition; 10 for Insulin), matched by simulation index
- Significance flags must be recorded per method-vs-missForest comparison per dataset and missingness level, not collapsed across conditions
- Mean error values fed to the test must be on the paper's scale: good performance near 0, bad performance around 1 for both NRMSE and PFC
- No significance statement should appear for MICE on the Children's Hospital data, and no MissPALasso significance entry for the Insulin data
- Significance outcomes in imputation_comparison.csv must be consistent with the flags propagated into missforest_gain_summary.csv

## Limitations

- Only 10 simulations instead of 50 were performed for the Insulin data due to computation time, reducing test power for that dataset
- Due to ad-hoc manipulation needed to make the MICE implementation work on the Children's Hospital data, no significance statements are reported there
- The test only supports the comparison table; the OOB imputation error estimates of missForest are a separate informal reliability check and are not part of the significance assessment

## Evidence

- [methods] error-scale convention: [evidence span withheld — anchor status: not_found]
