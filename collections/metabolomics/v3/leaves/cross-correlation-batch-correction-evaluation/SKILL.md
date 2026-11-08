---
name: cross-correlation-batch-correction-evaluation
description: Use when comparing batch-correction methods on multi-batch untargeted
  metabolomics data (e.g. the Cherkaoui et al. cancer-cell-line dataset) and you need
  quantitative metrics to rank methods or test robustness of normalization hyperparameters.
when_to_use_negative:
- Data are single-batch with no replicate reference samples across batches, so cross-correlation
  of interbatch replicates cannot be computed
- Evaluating performance claims on datasets other than multi-batch untargeted metabolomics
  (the metrics rely on replicate measurements across batches of MS data)
- You need per-feature differential analysis rather than method-level normalization
  quality ranking
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: RALPS
  role: Produces the normalized data being evaluated; one of the methods ranked by
    cross-correlation improvement and spectral preservation
  repo: https://github.com/zamboni-lab/RALPS
- name: scikit-learn
  role: Derives latent_dim from PCA at variance thresholds used in the robustness
    sweep
- name: pandas
  role: Loads normalized matrices and batch info (batch/group/benchmark columns) for
    metric computation
- name: numpy
  role: Computes cross-correlations and negative-value fractions across methods
- name: matplotlib
  role: Plots mean replicate cross-correlation versus variance_ratio (Figure 4e)
- name: seaborn
  role: Supports the evaluation plots of metric versus hyperparameter setting
provenance:
  source_task_ids:
  - task_004
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad096
    title: Regularized adversarial learning for normalization of multi-batch untargeted
      metabolomics data
  source_content_hash: sha256:be64fe48d51c5cf85132e1633e65ed18718ba71914c34f35b9fc88e22280f64c
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# cross-correlation-batch-correction-evaluation

## Summary

Evaluation methodology from the RALPS paper (Dmitrenko et al., Bioinformatics 2023) that judges multi-batch metabolomics normalization quality by computing cross-correlation improvement of replicate measurements across batches, spectral preservation, and the fraction of negative values in normalized matrices.

## When to use

Use when comparing batch-correction methods on multi-batch untargeted metabolomics data (e.g. the Cherkaoui et al. cancer-cell-line dataset) and you need quantitative metrics to rank methods or test robustness of normalization hyperparameters.

## When NOT to use

- Data are single-batch with no replicate reference samples across batches, so cross-correlation of interbatch replicates cannot be computed
- Evaluating performance claims on datasets other than multi-batch untargeted metabolomics (the metrics rely on replicate measurements across batches of MS data)
- You need per-feature differential analysis rather than method-level normalization quality ranking

## Inputs

- Normalized feature-by-sample CSV matrix (output of RALPS or a baseline batch-correction method)
- Batch info CSV with batch, group (reference replicates, e.g. MDAMB231 and MCF7), and benchmark columns
- Raw annotated-feature metabolomics data file (e.g. Cherkaoui et al. cancer cell-line dataset)
- RALPS config.csv specifying data_path, info_path, and out_path

## Outputs

- Per-method cross-correlation improvement values across batches
- Spectral preservation scores per method
- Fraction of negative values in each normalized matrix
- Mean replicate cross-correlation versus variance_ratio plot (e.g. Figure 4e style, matplotlib/seaborn)

## How to apply

After running each normalization method (RALPS via 'python ralps.py -n path/to/config.csv' plus six state-of-the-art baselines such as ComBat and WaveICA) on the same annotated-feature data and batch info file, compute evaluation metrics across all methods: cross-correlation improvement of replicate/reference samples across batches (using the batch and group columns, e.g. MDAMB231 and MCF7 reference replicates), spectral preservation, and the fraction of negative values in the normalized matrix. For robustness sweeps, vary latent_dim derivation by setting variance_ratio to values such as 0.99, 0.95, 0.9, 0.8, 0.7 and plot mean replicate cross-correlation versus variance_ratio. Validate that RALPS/WaveICA attain the best cross-correlation improvement, RALPS ranks top for spectral preservation, all RALPS outputs are strictly positive, and ComBat/WaveICA produce roughly 1% negative values. Mean cross-correlation above 0.98 at every setting indicates robust performance.

## Related tools

- **RALPS** (Produces the normalized data being evaluated; one of the methods ranked by cross-correlation improvement and spectral preservation) — https://github.com/zamboni-lab/RALPS
- **scikit-learn** (Derives latent_dim from PCA at variance thresholds used in the robustness sweep)
- **pandas** (Loads normalized matrices and batch info (batch/group/benchmark columns) for metric computation)
- **numpy** (Computes cross-correlations and negative-value fractions across methods)
- **matplotlib** (Plots mean replicate cross-correlation versus variance_ratio (Figure 4e))
- **seaborn** (Supports the evaluation plots of metric versus hyperparameter setting)

## Examples

```
python ralps.py -n path/to/config.csv
```

## Evaluation signals

- Mean replicate cross-correlation across batches exceeds 0.98 at every variance_ratio setting (0.99, 0.95, 0.9, 0.8, 0.7) with no clear monotonic trend
- RALPS normalized outputs are strictly positive (no negative values), whereas ComBat and WaveICA yield negative values in about 1% of samples
- RALPS and WaveICA attain the best cross-correlation improvement on the Cherkaoui et al. data, and RALPS ranks top for spectral preservation
- All six baseline methods and RALPS are evaluated on the identical input data and batch info file so metrics are directly comparable

## Limitations

- Varying the neuron count via PCA variance thresholds showed no clear trend in any of the four evaluation metrics, so metric changes may not be attributable to architecture size
- Baseline methods such as ComBat and WaveICA can produce negative normalized values (~1% of samples), which complicates comparison of strictly positive methods
- RALPS normalization should be applied only after each batch is normalized individually to suppress intrabatch problems such as temporal LC-MS drifts; evaluating raw intrabatch-confounded data is out of scope

## Evidence

- [results] RALPS and WaveICA were the best methods for normalizing the Cherkaoui et al. data; RALPS better preserved low-intensity data and produced strictly positive values, while ComBat and WaveICA yielded negative values in about 1% of samples.: [evidence span withheld — anchor status: not_found]
- [results] Varying the neuron count via PCA variance thresholds (99% to 70%) produced no clear trend in any of the four evaluation metrics; mean cross-correlation of replicates remained above 0.98 in all cases (Fig. 4e).: [evidence span withheld — anchor status: not_found]
- [methods] Compute the evaluation metrics across methods, including cross-correlation improvement, spectral preservation, and the fraction of negative values in the normalized matrices: [evidence span withheld — anchor status: not_found]
- [readme] each batch should be first normalized individually to suppress intrabatch problems, e.g. temporal trends associated to drifts in LC-MS. RALPS is used in a second step to harmonize multiple batches: "each batch should be first normalized individually to suppress intrabatch problems, e.g. temporal trends associated to drifts in LC-MS. RALPS is used in a second step to harmonize multiple batches"
