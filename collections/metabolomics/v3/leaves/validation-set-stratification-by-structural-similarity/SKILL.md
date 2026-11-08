---
name: validation-set-stratification-by-structural-similarity
description: Use when evaluating a trained RT prediction model on a held-out validation
  set and you need to determine whether prediction error is smaller for molecules
  that have a structurally similar molecule (e.g. ≥90% Tanimoto on ECFP fingerprints)
  in the training set.
when_to_use_negative:
- The goal is training or hyperparameter tuning of the RT prediction model rather
  than post-hoc evaluation of error dependence on training-set similarity
- The input structures lack SMILES/ECFP-computable representations, so Tanimoto similarity
  thresholds cannot be computed
- The validation table includes non-retained molecules or lacks paired predicted vs
  experimental RT values needed to compute relative RT prediction error
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_2269
tools:
- name: RDKit
  role: Computes ECFP fingerprints and Tanimoto similarity between each validation-set
    molecule and all training-set molecules to assign similarity-threshold groups
- name: SciPy
  role: Runs the Wilcoxon rank test on relative RT prediction errors, repeated 1000
    times with random subset draws for Bonferroni correction
- name: pandas
  role: Loads and joins the SMRT dataset and predicted-vs-experimental RT table, and
    writes similarity_threshold_statistical_tests.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-13680-7
    title: The METLIN small molecule dataset for machine learning-based retention
      time prediction
  source_content_hash: sha256:2a4321aed2c419cec961740db698da6812c95e09ccd684ea086a5a341b1ac540
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - RDKit
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# validation-set-stratification-by-structural-similarity

## Summary

Tests whether a DLM's RT prediction error depends on structural similarity to the training set by stratifying validation-set molecules into 'with similar counterpart' vs 'without counterpart' groups at ECFP/Tanimoto thresholds (95, 90, 80, 70, 50%) and comparing relative RT errors with a repeated, Bonferroni-corrected Wilcoxon rank test.

## When to use

Use when evaluating a trained RT prediction model on a held-out validation set and you need to determine whether prediction error is smaller for molecules that have a structurally similar molecule (e.g. ≥90% Tanimoto on ECFP fingerprints) in the training set.

## When NOT to use

- The goal is training or hyperparameter tuning of the RT prediction model rather than post-hoc evaluation of error dependence on training-set similarity
- The input structures lack SMILES/ECFP-computable representations, so Tanimoto similarity thresholds cannot be computed
- The validation table includes non-retained molecules or lacks paired predicted vs experimental RT values needed to compute relative RT prediction error

## Inputs

- SMRT dataset molecular structures with experimentally acquired reverse-phase chromatography retention times (80,038 small molecules)
- Predicted vs experimental RT table for the held-out (25%) validation-set molecules from the trained DLM, with non-retained molecules excluded
- Split designation of which molecules belong to the training set vs validation set

## Outputs

- similarity_threshold_statistical_tests.csv with per-threshold significance levels and effective sample sizes (expected: significant error reduction at 90/80/70/50% thresholds with n = 5607, 5425, 2718, 177; not significant at 95%)
- Per-threshold group assignments of validation molecules (with vs without a Tanimoto-similar training counterpart)

## How to apply

Load the SMRT dataset (structures + experimentally acquired reverse-phase chromatography RT) and the predicted-vs-experimental RT table for the held-out validation fraction (25% in the source study), excluding non-retained molecules. Compute ECFP fingerprints for every validation-set and training-set molecule with RDKit and assign each validation molecule to a 'with similar counterpart' or 'without counterpart' group at each Tanimoto threshold of 95, 90, 80, 70 and 50%. At each threshold, run a Wilcoxon rank test (SciPy) comparing the relative RT prediction error of the two groups, repeating the test 1000 times with each repeat drawing a random subset from the larger group sized to the smaller group (e.g. n90 = 5607) to control for group-size imbalance. Apply a Bonferroni correction to the 1000 P values per threshold and select as the final significance level the smallest level (n.s., P<0.05, P<0.01, P<0.001, P<0.0001) under which at least 95% of corrected P values fall. The rationale: the article reports that correct candidate ranking depends on predicted RT quality, so establishing whether error scales with training-set similarity quantifies how much of model performance reflects memorization of similar structures rather than generalization.

## Related tools

- **RDKit** (Computes ECFP fingerprints and Tanimoto similarity between each validation-set molecule and all training-set molecules to assign similarity-threshold groups)
- **SciPy** (Runs the Wilcoxon rank test on relative RT prediction errors, repeated 1000 times with random subset draws for Bonferroni correction)
- **pandas** (Loads and joins the SMRT dataset and predicted-vs-experimental RT table, and writes similarity_threshold_statistical_tests.csv)

## Examples

```
from scipy.stats import wilcoxon; stat, p = wilcoxon(err_with_similar, err_without_similar)  # repeated 1000x on random subsets, then Bonferroni-correct
```

## Evaluation signals

- The analysis must reproduce the reported pattern: statistically significant error reduction at 90/80/70/50% thresholds (reported P<0.0001 with n = 5607, 5425, 2718, 177) and no significant reduction at the 95% threshold
- At least 95% of the 1000 Bonferroni-corrected P values per threshold must fall at or below the reported final significance level
- Group sizes must match the reported effective sample sizes after random subsetting (e.g. n90 = 5607)
- Non-retained molecules must be excluded before computing relative RT prediction error, or group errors will be inflated
- Output CSV must contain one row per threshold (95, 90, 80, 70, 50%) with significance level and sample size columns

## Limitations

- No significant error reduction is observed at the 95% similarity threshold, so very-high-similarity counterparts do not guarantee lower error
- Results depend on the 25% train/validation split of the SMRT dataset and on excluding non-retained molecules; other splits may shift threshold behavior
- The repeated-subset Wilcoxon procedure (1000 draws) controls for group-size imbalance but makes the significance level a distributional summary (95th-percentile rule) rather than a single test result

## Evidence

- [abstract] SMRT dataset covers 80,038 small molecules with reverse-phase RT: "an experimentally acquired reverse-phase chromatography retention time dataset covering up to 80,038 small molecules"
- [abstract] Correct candidate ranking depends on predicted RT quality: "in 70\documentclass[12pt]{minimal} ... \begin{document}$$\%$$\end{document}% of the cases, the correct molecular identity was ranked among the top 3 candidates based on their predicted retention time"
- [intro] Article evaluated the model's predictive ability and transfer to other chromatographic methods: "We analyzed the model’s predictive ability along with the ability to project predicted RT onto other CMs"
- [intro] Predicted RT used to rank and filter candidates from accurate mass search: "Further, we focused on the ability to rank and filter putative candidates in real metabolomics experiments based on accurate mass search and predicted RT"
