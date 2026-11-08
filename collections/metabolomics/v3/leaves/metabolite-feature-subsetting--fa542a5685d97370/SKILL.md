---
name: metabolite-feature-subsetting
description: Use when you must quantify the robustness of a batch-correction or normalization
  method against reduced dataset size, by systematically removing batches chronologically
  and metabolites randomly down to a minimal feature floor.
when_to_use_negative:
- The goal is absolute normalization quality benchmarking against other methods rather
  than sensitivity to dataset size
- The dataset has so few metabolites (<~17) or batches (<2) that meaningful batch-wise
  or feature-wise ablation subsets cannot be constructed
- Input data is single-batch and lacks the cross-batch replicate/reference samples
  RALPS requires for training
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: RALPS
  role: normalization method applied to each ablated subset via config-driven run
  repo: https://github.com/zamboni-lab/RALPS
- name: pandas
  role: loading the dataset and batch info file and constructing/writing the batch-wise
    and metabolite-wise subset csvs
- name: numpy
  role: numerical computation of coefficients of variation across replicate subsets
- name: scikit-learn
  role: PCA used to derive latent_dim automatically at variance_ratio 0.9/0.95/0.99
- name: PyTorch
  role: backend for RALPS adversarial training (30 epochs, batch_size 32/64/128)
- name: matplotlib
  role: plotting mean batch VC vs subset size and normalization-induced variance vs
    number of metabolites
- name: seaborn
  role: statistical visualization of the ablation trends
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad096
    title: Regularized adversarial learning for normalization of multi-batch untargeted
      metabolomics data
  source_content_hash: sha256:389a64efb3262609bbf63b201cfab2d015cd02b15a5fbb20aeab975cf71388bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zamboni-lab/RALPS
  tools:
  - PyTorch
  - RALPS
  - matplotlib
  - numpy
  - pandas
  - scikit-learn
  - seaborn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# metabolite-feature-subsetting

## Summary

A data-shrink ablation protocol that progressively subsets a multi-batch metabolomics dataset — first chronologically by batches, then randomly by metabolites down to ~10% of features — to probe how downstream normalization performance (mean batch VC) degrades with shrinking input.

## When to use

Use when you must quantify the robustness of a batch-correction or normalization method against reduced dataset size, by systematically removing batches chronologically and metabolites randomly down to a minimal feature floor.

## When NOT to use

- The goal is absolute normalization quality benchmarking against other methods rather than sensitivity to dataset size
- The dataset has so few metabolites (<~17) or batches (<2) that meaningful batch-wise or feature-wise ablation subsets cannot be constructed
- Input data is single-batch and lacks the cross-batch replicate/reference samples RALPS requires for training

## Inputs

- csv metabolomics feature table (untargeted, multi-batch, with replicate/reference samples per batch)
- csv batch information file mapping sample_id to batch
- RALPS config csv with data_path, info_path, out_path and hyperparameter ranges (d_lr, g_lr, d_lambda, g_lambda, v_lambda, batch_size, variance_ratio, epochs)

## Outputs

- per-subset csv data files and matching batch info files spanning full down to ~10% of metabolites
- normalized data for each batch-wise and metabolite-wise subset
- mean batch VC versus subset size plot (Figure 4b analogue)
- normalization-induced variance versus number of metabolites plot (Figure 4c analogue)

## How to apply

Load the benchmarking dataset and batch information file with pandas, then construct batch-wise subsets retaining the chronologically first 2 up to all 7 batches. For each batch subset, draw metabolite-wise subsets spanning the full feature range down to ~10% of metabolites (roughly 17 metabolites), writing each subset as a csv data file plus a matching batch info file. Run RALPS on every subset via the config-driven command `python ralps.py -n path/to/config.csv`, letting latent_dim be derived automatically from PCA (variance_ratio 0.9/0.95/0.99) and hyperparameters d_lr, g_lr, d_lambda, g_lambda, v_lambda, and batch_size (32/64/128) be sampled by the randomized grid search over 30 epochs. Collect the normalized data per subset and compute the mean batch coefficient of variation (VC) across replicates after normalization. Finally, plot mean batch VC versus subset size (Figure 4b trend) and normalization-induced variance versus number of metabolites (Figure 4c) with matplotlib/seaborn to locate the ablation point where performance tangibly worsens.

## Related tools

- **RALPS** (normalization method applied to each ablated subset via config-driven run) — https://github.com/zamboni-lab/RALPS
- **pandas** (loading the dataset and batch info file and constructing/writing the batch-wise and metabolite-wise subset csvs)
- **numpy** (numerical computation of coefficients of variation across replicate subsets)
- **scikit-learn** (PCA used to derive latent_dim automatically at variance_ratio 0.9/0.95/0.99)
- **PyTorch** (backend for RALPS adversarial training (30 epochs, batch_size 32/64/128))
- **matplotlib** (plotting mean batch VC vs subset size and normalization-induced variance vs number of metabolites)
- **seaborn** (statistical visualization of the ablation trends)

## Examples

```
python ralps.py -n path/to/config.csv
```

## Evaluation signals

- Mean batch VC across batch-wise subsets stays near constant across all subset sizes (Figure 4b-like flat trend)
- Normalization-induced variance only increases tangibly at the last ablation point, at approximately 17 metabolites (~10% of initial features)
- Each generated subset has both a csv data file and a matching batch info file so `python ralps.py -n config.csv` runs without schema errors
- latent_dim resolves automatically from PCA at variance_ratio 0.9/0.95/0.99 rather than being hard-coded
- The full feature range of ablation points is covered, ending at roughly 10% of the initial metabolite count

## Limitations

- Performance degradation is only visible at the extreme ablation end (~17 metabolites), so intermediate shrinkage yields near-constant VC and may not expose other failure modes
- Hyperparameters are randomly sampled (d_lr, g_lr 0.00005-0.005; d_lambda, g_lambda, v_lambda 0.-10.; batch_size 32/64/128), so runs are stochastic and a single grid run may not capture worst-case behavior
- Training is costly: RALPS took on average 5.82 minutes per 30-epoch run on ~3000 samples and ~150 metabolites, so a full batch×metabolite ablation grid multiplies runtime

## Evidence

- [results] RALPS's mean batch variance remained stable during metabolite-wise shrinkage of the dataset down to 10% of the initial number of metabolites, and only tangibly worsened at the last ablation point, corresponding to 17 metabolites.: [evidence span withheld — anchor status: not_found]
- [methods] Run the following command from the `src` directory to normalize data with RALPS: `python ralps.py -n path/to/config.csv`: "Run the following command from the `src` directory to normalize data with RALPS: `python ralps.py -n path/to/config.csv`"
- [readme] latent_dim | dimension of the bottleneck linear layer | -1 (automatically derived from PCA) | variance_ratio | percent of explained variance to derive latent_dim | 0.9,0.95,0.99: "-1 (automatically derived from PCA)"
- [readme] Training time required to normalize a dataset with ~3000 samples and ~150 metabolites was 5.82 minutes per run on average (30 epochs).: "Training time required to normalize a dataset with ~3000 samples and ~150 metabolites was 5.82 minutes per run on average (30 epochs)."
