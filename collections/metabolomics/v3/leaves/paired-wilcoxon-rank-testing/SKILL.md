---
name: paired-wilcoxon-rank-testing
description: Use when comparing two related sets of continuous error scores (e.g.,
  DLM vs. naive k-NN relative RT errors on the same validation molecules, or errors
  of molecules with vs. without a >=threshold Tanimoto-similar training counterpart)
  and you need a distribution-free significance test.
when_to_use_negative:
- The two error distributions being compared are not paired (different molecules in
  each group) — use an unpaired rank-sum test instead
- More than two models or groups must be compared simultaneously — a Wilcoxon rank
  test handles only two related samples
- The comparison targets nominal or categorical outcomes rather than continuous relative
  RT errors
edam_operation: http://edamontology.org/operation_2292
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3174
tools:
- name: SciPy
  role: Provides the paired Wilcoxon rank test (scipy.stats.wilcoxon) used within
    each threshold group and across the 1000-resample repeats
- name: pandas
  role: Loads the predicted vs. experimental RT validation table and writes the per-threshold
    significance results to CSV/TSV
- name: RDKit
  role: Computes ECFP fingerprints used to derive the Tanimoto threshold groups whose
    errors are compared by the Wilcoxon test
- name: Keras
  role: Trains the DLM (4 hidden layers of 1000/500/200/100 relu units, L2 0.0001,
    adam lr 0.01, MSE loss, 20 epochs, batch size 35) whose errors are tested against
    the naive baseline
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-13680-7
    title: The METLIN small molecule dataset for machine learning-based retention
      time prediction
  source_content_hash: sha256:ce3ffa7ab9fa29ce713e1954d821d8765e3f41e3870f8f624259b1020e8224bd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Keras
  - Python
  - RDKit
  - SciPy
  - Wilcoxon rank test
  - pandas
  edam_operation: http://edamontology.org/operation_2292
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# paired-wilcoxon-rank-testing

## Summary

Non-parametric paired Wilcoxon rank testing applied to relative RT prediction errors from the METLIN SMRT deep learning model (DLM) versus the naive k-NN (k=3) median-RT baseline, or between validation molecules with vs. without a structurally similar training counterpart, at Tanimoto thresholds of 95/90/80/70/50%.

## When to use

Use when comparing two related sets of continuous error scores (e.g., DLM vs. naive k-NN relative RT errors on the same validation molecules, or errors of molecules with vs. without a >=threshold Tanimoto-similar training counterpart) and you need a distribution-free significance test.

## When NOT to use

- The two error distributions being compared are not paired (different molecules in each group) — use an unpaired rank-sum test instead
- More than two models or groups must be compared simultaneously — a Wilcoxon rank test handles only two related samples
- The comparison targets nominal or categorical outcomes rather than continuous relative RT errors

## Inputs

- Per-molecule relative RT prediction errors for the DLM and the naive k-NN (k=3) median-RT baseline on the 25% validation set (e.g., knn_dlm_error_comparison.tsv)
- Group assignments of validation molecules at each Tanimoto similarity threshold (95/90/80/70/50%) computed from ECFP fingerprints against the 75% training set
- SMRT dataset RT values in seconds and ECFP fingerprints for training/validation molecules

## Outputs

- Per-threshold Wilcoxon P values and significance levels (e.g., similarity_threshold_statistical_tests.csv)
- Group summaries (median, IQR) and per-molecule errors with paired-test results (knn_dlm_error_comparison.tsv)

## How to apply

Group the 25% validation molecules of the METLIN SMRT dataset by ECFP Tanimoto similarity to training molecules at thresholds of 95, 90, 80, 70 and 50%. For the DLM-vs-baseline comparison, run a paired Wilcoxon rank test on the relative RT errors of the two models within each threshold group (paired because both models predict the same molecules). For the with/without-counterpart comparison, repeat the Wilcoxon test 1000 times, each draw taking a random subset of the larger group sized to the smaller group (e.g., n90 = 5607), then apply a Bonferroni correction to the 1000 P values and report the smallest significance level (n.s., P<0.05, P<0.01, P<0.001, P<0.0001) under which at least 95% of corrected P values fall. Rationale: relative RT errors are non-normal, so a rank-based paired test is used instead of a paired t-test.

## Related tools

- **SciPy** (Provides the paired Wilcoxon rank test (scipy.stats.wilcoxon) used within each threshold group and across the 1000-resample repeats)
- **pandas** (Loads the predicted vs. experimental RT validation table and writes the per-threshold significance results to CSV/TSV)
- **RDKit** (Computes ECFP fingerprints used to derive the Tanimoto threshold groups whose errors are compared by the Wilcoxon test)
- **Keras** (Trains the DLM (4 hidden layers of 1000/500/200/100 relu units, L2 0.0001, adam lr 0.01, MSE loss, 20 epochs, batch size 35) whose errors are tested against the naive baseline)

## Examples

```
from scipy.stats import wilcoxon; stat, p = wilcoxon(dlm_rel_err, knn_rel_err)
```

## Evaluation signals

- Recovered pattern matches the report: no significant mean-error difference at the 95% Tanimoto threshold, and P < 0.0001 at the 90/80/70/50% thresholds for DLM vs. naive k-NN
- With/without-counterpart test reproduces significance at 90/80/70/50% (reported P<0.0001, n = 5607, 5425, 2718, 177) and non-significance at 95%
- After Bonferroni correction, at least 95% of the 1000 repeated P values fall under the reported smallest significance level
- Output files contain the required columns: per-molecule relative errors, group medians and IQRs, and Wilcoxon P values
- Group sizes at each threshold are consistent with the reported effective sample sizes (e.g., n=5607 at 90%)

## Limitations

- At the 95% Tanimoto similarity threshold the difference in mean error was not significant, so the test cannot claim DLM superiority for molecules with near-identical training counterparts
- The with/without-counterpart comparison requires repeated random subsampling (1000 draws) plus Bonferroni correction when group sizes are unbalanced, inflating computational cost
- Findings are tied to the specific 75/25 random split of the 80,038-molecule SMRT dataset and reverse-phase chromatography RTs in seconds; other splits or chromatographic methods may shift significance

## Evidence

- [abstract] SMRT dataset coverage: "an experimentally acquired reverse-phase chromatography retention time dataset covering up to 80,038 small molecules"
- [intro] model evaluation framing: "We analyzed the model’s predictive ability along with the ability to project predicted RT onto other CMs"
- [intro] candidate ranking motivation: "Further, we focused on the ability to rank and filter putative candidates in real metabolomics experiments based on accurate mass search and predicted RT"
