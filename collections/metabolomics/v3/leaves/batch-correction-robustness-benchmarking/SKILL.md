---
name: batch-correction-robustness-benchmarking
description: Use when evaluating whether an adversarial batch-correction method for
  multi-batch untargeted metabolomics data tolerates randomly distributed missing
  values, or when comparing several normalization methods on replicate-containing
  benchmarking data.
when_to_use_negative:
- Data are single-batch intrabatch LC-MS drift problems that should be handled by
  first normalizing each batch individually, before applying RALPS as a second harmonization
  step
- The dataset has no reference/replicate samples present across batches, so interbatch
  differences cannot be assessed from replicate measurements
- The goal is exploratory normalization of a new study rather than a controlled robustness
  or benchmark comparison with defined evaluation criteria
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: RALPS
  role: adversarial batch-correction method being stress-tested and benchmarked; run
    via 'python ralps.py -n path/to/config.csv' from the src directory
  repo: https://github.com/zamboni-lab/RALPS
- name: pandas
  role: loading the data matrix and batch info file, writing degraded CSV matrices
    and config files
- name: numpy
  role: randomly replacing fractions of matrix values with 1000 counts and computing
    correlations/VCs
- name: scipy
  role: Pearson cross-correlation of replicate samples and mean batch VC computation
- name: PyTorch
  role: backend for RALPS training (torch==1.8.1) with optional checkpoint keeping
- name: matplotlib
  role: plotting mean replicate cross-correlation vs. missing-value fraction and per-method
    spectra
- name: seaborn
  role: box plots for qualitative spectrum-preservation assessment
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad096
    title: Regularized adversarial learning for normalization of multi-batch untargeted
      metabolomics data
  source_content_hash: sha256:463130478ce144dde2f6eb1c74d7245de013d9a669acf846260580f69acbae32
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
  - scipy
  - seaborn
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5217391304347826
n_claims: 23
---

# batch-correction-robustness-benchmarking

## Summary

A method-rich procedure for stress-testing the RALPS batch-correction method against missing values (0%–30% of matrix values replaced with 1000 counts) and benchmarking it against ComBat, EigenMS, LEV+EIG, PQN+POW, WaveICA and NormAE on a multi-batch metabolomics matrix using replicate cross-correlation, batch VC, and VC-inflation criteria.

## When to use

Use when evaluating whether an adversarial batch-correction method for multi-batch untargeted metabolomics data tolerates randomly distributed missing values, or when comparing several normalization methods on replicate-containing benchmarking data.

## When NOT to use

- Data are single-batch intrabatch LC-MS drift problems that should be handled by first normalizing each batch individually, before applying RALPS as a second harmonization step
- The dataset has no reference/replicate samples present across batches, so interbatch differences cannot be assessed from replicate measurements
- The goal is exploratory normalization of a new study rather than a controlled robustness or benchmark comparison with defined evaluation criteria

## Inputs

- benchmarking metabolomics data matrix (CSV, samples x features)
- batch info file (CSV) with 'group' column for replicate matching and a benchmark column flagging reference groups
- RALPS config.csv with data_path, info_path, out_path and parameters (epochs=30, train_ratio=0.9, min_relevant_intensity=1000, allowed_vc_increase=0.05)
- degraded matrices with 0/5/10/15/20/30% of values replaced by 1000 counts

## Outputs

- RALPS-normalized data matrices per missing-value level and per comparison method
- mean replicate cross-correlation vs. missing-value fraction plot (matplotlib/seaborn, reproducing Figure 4d)
- summary CSV of the four evaluation criteria per method
- figure2_criteria.png multi-panel figure with per-method spectra and box plots

## How to apply

Load the benchmarking data matrix and batch info file with pandas, then create a config.csv pointing to both files with epochs=30, train_ratio=0.9, min_relevant_intensity=1000 and allowed_vc_increase=0.05, flagging reference groups in the benchmark column so they are excluded from regularization. For the robustness ablation, generate six degraded matrices by randomly replacing 0%, 5%, 10%, 15%, 20% and 30% of matrix values with 1000 counts (the default min_relevant_intensity), writing one config per level. Run normalization from the src directory with 'python ralps.py -n path/to/config.csv'; the randomized grid search samples classifier/autoencoder learning rates in 0.00005–0.005 and latent_dim is derived automatically from PCA. Compute the mean Pearson cross-correlation among replicate samples (matched via the 'group' column, excluding benchmark samples) and the mean batch coefficient of variation per method, plus the fraction of features whose VC is inflated by at least 5%. Plot mean replicate cross-correlation against missing-value fraction with matplotlib/seaborn; the correct outcome is a marginal decline from ~0.98 (0% missing) to ~0.94 (30% missing), reproducing Figure 4d, and RALPS should be the only method excelling at batch-bias suppression while controlling mean batch VC, replicate VCs and spectral transformation.

## Related tools

- **RALPS** (adversarial batch-correction method being stress-tested and benchmarked; run via 'python ralps.py -n path/to/config.csv' from the src directory) — https://github.com/zamboni-lab/RALPS
- **pandas** (loading the data matrix and batch info file, writing degraded CSV matrices and config files)
- **numpy** (randomly replacing fractions of matrix values with 1000 counts and computing correlations/VCs)
- **scipy** (Pearson cross-correlation of replicate samples and mean batch VC computation)
- **PyTorch** (backend for RALPS training (torch==1.8.1) with optional checkpoint keeping)
- **matplotlib** (plotting mean replicate cross-correlation vs. missing-value fraction and per-method spectra)
- **seaborn** (box plots for qualitative spectrum-preservation assessment)

## Examples

```
python ralps.py -n path/to/config.csv
```

## Evaluation signals

- Mean replicate cross-correlation of the ablation curve declines only marginally, from ~0.98 at 0% missing to ~0.94 at 30% missing (Figure 4d)
- Fraction of features with VC inflated by >=5% stays low for RALPS relative to ComBat, EigenMS, LEV+EIG, PQN+POW, WaveICA and NormAE
- Mean batch VC of reference samples is controlled (allowed_vc_increase=0.05) while batch biases are suppressed
- Per-method spectra plots show RALPS preserves spectral properties without drastic transformation
- Six degraded CSV matrices and one config.csv per level are produced with exactly 0/5/10/15/20/30% replacement fractions

## Limitations

- Missing values are simulated by replacement with the default min_relevant_intensity (1000 counts); other imputation strategies are not covered
- Benchmark samples flagged in the benchmark column must be excluded from regularization and replicate cross-correlation computation
- Benchmark results are shown for the two-reference-group scenario; other experimental designs are supported but not benchmarked here

## Evidence

- [results] ablation finding: [evidence span withheld — anchor status: not_found]
- [abstract] benchmark finding: "Using two large metabolomics datasets, we showcase the superior performance of RALPS as compared with six state-of-the-art methods for batch correction."
- [readme] run command: "Run the following command from the `src` directory to normalize data with RALPS: `python ralps.py -n path/to/config.csv`"
- [readme] min_relevant_intensity: "missing values before normalization are replaced with this;<br/>values below this after normalization are masked with zeros"
- [methods] latent_dim: "-1 (automatically derived from PCA)"
