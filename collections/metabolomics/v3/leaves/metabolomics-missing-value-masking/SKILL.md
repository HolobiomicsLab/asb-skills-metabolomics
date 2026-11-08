---
name: metabolomics-missing-value-masking
description: Use when a feature count matrix destined for RALPS normalization contains
  missing values, or when stress-testing normalization robustness by randomly replacing
  matrix values with the 1000-count minimum.
when_to_use_negative:
- The input matrix is complete (no missing values) and the task is batch-effect harmonization
  only, so threshold replacement/masking adds nothing on its own
- Missing values require feature-aware or model-based imputation rather than RALPS's
  flat 1000-count minimum replacement
- True low-abundance signals below 1000 counts must be preserved after normalization
  instead of being zero-masked
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: RALPS
  role: Reads min_relevant_intensity (default 1000) from the config csv to replace
    missing values before training and to zero-mask sub-threshold values in the normalized
    output; run as 'python ralps.py -n path/to/config.csv'
  repo: https://github.com/zamboni-lab/RALPS
- name: pandas
  role: Loads the benchmarking feature count matrix and batch info file and writes
    the six degraded CSV matrices at the 0-30% replacement levels
- name: numpy
  role: Randomly selects the requested fraction of matrix values to replace with the
    1000-count minimum at each degradation level
- name: PyTorch
  role: Backend for the RALPS adversarial autoencoder trained on each degraded matrix
    (torch==1.8.1)
- name: matplotlib
  role: Renders the mean replicate cross-correlation versus missing-value fraction
    curve with plots_extension=png
- name: seaborn
  role: Styles the robustness evaluation plot of cross-correlation against missing
    fraction
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad096
    title: Regularized adversarial learning for normalization of multi-batch untargeted
      metabolomics data
  source_content_hash: sha256:3079f8c886698163d0eabf84aee98fc210ab4e7042e19fb62af100dc267c9040
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zamboni-lab/RALPS
  tools:
  - PyTorch
  - RALPS
  - hdbscan
  - matplotlib
  - numpy
  - pandas
  - scikit-learn
  - seaborn
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# metabolomics-missing-value-masking

## Summary

Missing-value handling for RALPS normalization of multi-batch untargeted metabolomics matrices: missing values are replaced with min_relevant_intensity (default 1000 counts) before training, sub-threshold values are zero-masked in the normalized output, and RALPS tolerates up to 30% randomly replaced values with only marginal loss of replicate cross-correlation.

## When to use

Use when a feature count matrix destined for RALPS normalization contains missing values, or when stress-testing normalization robustness by randomly replacing matrix values with the 1000-count minimum.

## When NOT to use

- The input matrix is complete (no missing values) and the task is batch-effect harmonization only, so threshold replacement/masking adds nothing on its own
- Missing values require feature-aware or model-based imputation rather than RALPS's flat 1000-count minimum replacement
- True low-abundance signals below 1000 counts must be preserved after normalization instead of being zero-masked

## Inputs

- Feature count matrix CSV with missing values (e.g. the 170-feature x 2856-sample seven-batch benchmarking metabolomics matrix)
- Batch info CSV with batch, group, and benchmark columns identifying replicate groups (e.g. P2_S_0001, P2_S_PP_0001)
- RALPS config CSV specifying data_path, info_path, out_path, epochs, and min_relevant_intensity=1000

## Outputs

- Degraded CSV matrices with 0/5/10/15/20/30% of values randomly replaced by 1000 counts (ablation inputs)
- normalized/data.csv in which post-normalization values below min_relevant_intensity are masked with zeros
- Evaluation plot (plots_extension=png) of mean replicate cross-correlation versus missing-value fraction

## How to apply

Set min_relevant_intensity=1000 in the RALPS config csv: missing values in the input data file are replaced with this minimum before training, and after normalization values below 1000 counts are masked with zeros in normalized/data.csv. To reproduce the missing-value robustness ablation, load the 170-feature x 2856-sample benchmarking matrix with pandas/numpy and, for each level 0%, 5%, 10%, 15%, 20%, and 30%, randomly replace that fraction of matrix values with 1000 counts, writing six degraded csv files. Create a config.csv per degraded matrix (epochs=30, latent_dim=-1 derived from PCA at variance ratios 0.9,0.95,0.99, randomized grid search sampling d_lr/g_lr in 0.00005-0.005) and run 'python ralps.py -n path/to/config.csv' from the src directory. Match replicate samples via the 'group' column of the batch info file (excluding benchmark samples, e.g. P2_S_0001 and P2_S_PP_0001, that are not used for regularization) and compute the mean replicate cross-correlation on each run's normalized output. Plot mean cross-correlation against missing-value fraction with matplotlib/seaborn and accept if the decline is marginal, from about 0.98 at 0% missing to about 0.94 at 30% missing as in the paper's Figure 4d.

## Related tools

- **RALPS** (Reads min_relevant_intensity (default 1000) from the config csv to replace missing values before training and to zero-mask sub-threshold values in the normalized output; run as 'python ralps.py -n path/to/config.csv') — https://github.com/zamboni-lab/RALPS
- **pandas** (Loads the benchmarking feature count matrix and batch info file and writes the six degraded CSV matrices at the 0-30% replacement levels)
- **numpy** (Randomly selects the requested fraction of matrix values to replace with the 1000-count minimum at each degradation level)
- **PyTorch** (Backend for the RALPS adversarial autoencoder trained on each degraded matrix (torch==1.8.1))
- **matplotlib** (Renders the mean replicate cross-correlation versus missing-value fraction curve with plots_extension=png)
- **seaborn** (Styles the robustness evaluation plot of cross-correlation against missing fraction)

## Examples

```
python ralps.py -n path/to/config.csv
```

## Evaluation signals

- normalized/data.csv contains no values in the open interval (0, min_relevant_intensity): everything below 1000 counts is masked to zeros
- Each degraded CSV has exactly the requested fraction (0, 5, 10, 15, 20, or 30%) of values set to 1000 counts at randomly chosen positions
- Mean replicate cross-correlation (replicates matched via the 'group' column, benchmark samples excluded from regularization matching) is about 0.98 at 0% missing and drops only to about 0.94 at 30% missing, consistent with Figure 4d
- Batch-wise coefficients of variation of the normalized data are reduced and replicate cross-correlations are increased relative to pre-normalization
- Output matrix shape matches the input (e.g. 170 features x 2856 samples) after replacement and masking

## Limitations

- Robustness was only characterized up to 30% randomly replaced values; behavior beyond that level is untested
- Missing values are filled with a single flat minimum (1000 counts), not feature-aware imputation, and sub-threshold intensities are irreversibly zero-masked post-normalization
- The ablation injected randomly distributed values; systematic or feature-dependent missingness mechanisms were not assessed

## Evidence

- [readme] Pre- and post-normalization min_relevant_intensity handling: "missing values before normalization are replaced with this;<br/>values below this after normalization are masked with zeros"
- [abstract] RALPS robustness and missing-value handling claim: "Further, we demonstrate that RALPS scales well, is robust, deals with missing values and can handle different experimental designs."
- [intro] Spectral preservation and missing-value robustness: "RALPS preserves spectral properties and is robust against missing values."
- [methods] Normalization run command: "Run the following command from the `src` directory to normalize data with RALPS: `python ralps.py -n path/to/config.csv`"
