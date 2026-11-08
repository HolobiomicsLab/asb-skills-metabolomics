---
name: retention-time-prediction-error-analysis
description: Use when you have a trained RT prediction model (e.g., a DLM on the METLIN
  SMRT dataset) plus predicted-vs-experimental RT for a validation split, and want
  to test whether structurally similar training-set counterparts reduce prediction
  error.
when_to_use_negative:
- You do not have a paired predicted-vs-experimental RT table for a held-out validation
  set (e.g., you only have the training corpus or a model checkpoint)
- The dataset includes non-retained molecules and you have not excluded them first,
  since the error comparison assumes retained-compound RTs
- The goal is model training or hyperparameter tuning rather than quantifying how
  training-set similarity affects prediction error
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: RDKit
  role: Generate ECFP fingerprints and compute Tanimoto similarity between validation-set
    and training-set molecules
- name: SciPy
  role: Run the Wilcoxon rank test comparing relative RT prediction error between
    similarity groups across 1000 random subsamples
- name: pandas
  role: Load the SMRT structures/RT and the predicted-vs-experimental RT table; write
    similarity_threshold_statistical_tests.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-13680-7
    title: The METLIN small molecule dataset for machine learning-based retention
      time prediction
  source_content_hash: sha256:2e8ee1c2e57b1cb1d266eae94ddd0f74d1e78c6a7e2537b0686f1be36e260f0c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - RDKit
  - SciPy
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# retention-time-prediction-error-analysis

## Summary

Statistical procedure to test whether a deep learning model's RT prediction error depends on the presence of structurally similar molecules in the training set. It groups validation-set molecules by their maximum Tanimoto similarity (ECFP) to training molecules and compares relative RT prediction error across groups with repeated Wilcoxon rank tests.

## When to use

Use when you have a trained RT prediction model (e.g., a DLM on the METLIN SMRT dataset) plus predicted-vs-experimental RT for a validation split, and want to test whether structurally similar training-set counterparts reduce prediction error.

## When NOT to use

- You do not have a paired predicted-vs-experimental RT table for a held-out validation set (e.g., you only have the training corpus or a model checkpoint)
- The dataset includes non-retained molecules and you have not excluded them first, since the error comparison assumes retained-compound RTs
- The goal is model training or hyperparameter tuning rather than quantifying how training-set similarity affects prediction error

## Inputs

- SMRT dataset structures and experimentally acquired reverse-phase chromatography retention times (retained molecules only)
- Predicted vs experimental RT table for the held-out (25%) validation set from the trained DLM
- ECFP fingerprints of validation-set and training-set molecules

## Outputs

- similarity_threshold_statistical_tests.csv with per-threshold (95/90/80/70/50%) significance levels and effective sample sizes
- Per-molecule assignment of validation molecules to 'with similar counterpart' / 'without counterpart' groups at each similarity threshold

## How to apply

Load the SMRT dataset (structures + RT) and the predicted-vs-experimental RT table for the 25% validation set from the trained DLM, excluding non-retained molecules. Compute Tanimoto similarity on ECFP fingerprints between each validation-set molecule and all training-set molecules, and assign each validation molecule to a 'with similar counterpart' or 'without counterpart' group at thresholds of 95, 90, 80, 70 and 50%. For each threshold, run a Wilcoxon rank test comparing the relative RT prediction error of the two groups, repeated 1000 times with each repeat drawing a random subset from the larger group sized to the smaller group (e.g., n90 = 5607). Apply the Bonferroni correction to the 1000 resulting P values and select as the final significance level the smallest level (n.s., P<0.05, P<0.01, P<0.001, P<0.0001) under which at least 95% of corrected P values fall. Write per-threshold significance levels and effective sample sizes to similarity_threshold_statistical_tests.csv.

## Related tools

- **RDKit** (Generate ECFP fingerprints and compute Tanimoto similarity between validation-set and training-set molecules)
- **SciPy** (Run the Wilcoxon rank test comparing relative RT prediction error between similarity groups across 1000 random subsamples)
- **pandas** (Load the SMRT structures/RT and the predicted-vs-experimental RT table; write similarity_threshold_statistical_tests.csv)

## Examples

```
from scipy.stats import wilcoxon; stat, p = wilcoxon(err_with_counterpart, err_without_counterpart)
```

## Evaluation signals

- The analysis reproduces the reported pattern: a statistically significant error reduction at the 90, 80, 70 and 50% similarity thresholds (reported P<0.0001, n = 5607, 5425, 2718, 177) and no significant reduction at the 95% threshold
- Effective sample sizes match the smaller-group sizes at each threshold (e.g., n90 = 5607)
- At each threshold, at least 95% of the 1000 Bonferroni-corrected P values fall under the selected significance level
- similarity_threshold_statistical_tests.csv exists and contains one row per threshold (95, 90, 80, 70, 50%) with significance level and effective n
- Non-retained molecules are absent from the validation error table before testing

## Limitations

- No significant error reduction was observed at the 95% similarity threshold, so near-identical training counterparts do not guarantee improved predictions under this test
- P values depend on 1000 random subsamples of the larger group to the smaller group's size; results are ensemble-based rather than from a single test
- Only retained molecules are analyzed; error statistics are undefined for compounds without a measured reverse-phase RT

## Evidence

- [abstract] SMRT dataset scale: "an experimentally acquired reverse-phase chromatography retention time dataset covering up to 80,038 small molecules"
- [intro] RT-based candidate ranking motivation: "Further, we focused on the ability to rank and filter putative candidates in real metabolomics experiments based on accurate mass search and predicted RT"
- [intro] Motivation for large-scale RT prediction models: "the lack of experimental RT datasets has hampered the design of ML methods for large-scale RT prediction"
