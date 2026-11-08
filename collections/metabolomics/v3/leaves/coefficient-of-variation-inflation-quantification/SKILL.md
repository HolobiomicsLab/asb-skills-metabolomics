---
name: coefficient-of-variation-inflation-quantification
description: 'Use when a candidate batch-correction output on multi-batch metabolomics
  data must be checked for variance inflation: compute per-feature VCs before and
  after normalization and the fraction of features whose VC increased by >=5% (allowed_vc_increase).'
when_to_use_negative:
- Single-batch datasets with no reference samples replicated across batches — VC inflation
  relative to batch effects cannot be assessed as designed
- Batches that have not first been normalized individually to suppress intrabatch
  LC-MS temporal drifts, which RALPS-based benchmarking assumes as a prerequisite
- Assessing a method on VC inflation alone without replicate cross-correlation and
  qualitative spectrum preservation, which risks selecting an over-shrinking normalization
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: RALPS
  role: Reference method whose variation-loss term (allowed_vc_increase) defines the
    5% VC-inflation tolerance used as the evaluation criterion
  repo: https://github.com/zamboni-lab/RALPS
- name: pandas
  role: Load data matrix, batch info file, and write the criteria summary CSV
  repo: https://github.com/zamboni-lab/RALPS
- name: numpy
  role: Compute per-feature VCs and VC-inflation fractions on raw vs normalized matrices
- name: scipy
  role: Pearson replicate cross-correlations of reference samples used jointly with
    VC metrics
- name: matplotlib
  role: Render the multi-panel criteria figure (figure2_criteria.png)
- name: seaborn
  role: Box and spectra plots for qualitative spectrum preservation
- name: PyTorch
  role: Backend executing RALPS normalization whose checkpoints are compared on VC
    criteria
  repo: https://github.com/zamboni-lab/RALPS
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad096
    title: Regularized adversarial learning for normalization of multi-batch untargeted
      metabolomics data
  source_content_hash: sha256:7a162a6e609686ae3da70a4479c16acd7f1c77f2656158c4c856a15abd601515
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
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# Coefficient-of-variation-inflation quantification

## Summary

Quantify how much a batch-correction method inflates per-feature coefficients of variation (VCs) relative to the input data, a core criterion in the RALPS benchmark where RALPS was the only method of seven (ComBat, EigenMS, LEV+EIG, PQN+POW, WaveICA, NormAE) to suppress batch bias while controlling VC expansion.

## When to use

Use when a candidate batch-correction output on multi-batch metabolomics data must be checked for variance inflation: compute per-feature VCs before and after normalization and the fraction of features whose VC increased by >=5% (allowed_vc_increase).

## When NOT to use

- Single-batch datasets with no reference samples replicated across batches — VC inflation relative to batch effects cannot be assessed as designed
- Batches that have not first been normalized individually to suppress intrabatch LC-MS temporal drifts, which RALPS-based benchmarking assumes as a prerequisite
- Assessing a method on VC inflation alone without replicate cross-correlation and qualitative spectrum preservation, which risks selecting an over-shrinking normalization

## Inputs

- Raw (pre-normalization) data matrix csv, samples x metabolite features
- Normalized data matrix csv per batch-correction method (RALPS output from `python ralps.py -n config.csv` plus ComBat, EigenMS, LEV+EIG, PQN+POW, WaveICA, NormAE outputs)
- Batch info csv mapping sample_id to batch and reference/benchmark group membership
- RALPS config.csv with min_relevant_intensity (default 1000) and allowed_vc_increase (default 0.05)

## Outputs

- Per-method fraction of features with VC inflated by >=5%
- Per-method mean batch VC of reference samples
- Per-method replicate cross-correlation of reference samples
- Summary CSV of the four benchmark criteria in the out_path results folder
- Multi-panel criteria figure (figure2_criteria.png) plus spectra/box plots via matplotlib/seaborn

## How to apply

Load the raw and normalized data matrices (csv, samples x features) with pandas and the batch info file, then compute the per-feature VC (SD/mean) within batches for both versions using numpy/scipy. Aggregate to the mean batch VC of reference samples (e.g. pooled study samples or NIST 1950 SRM present in every batch) and count the fraction of features whose VC is inflated by at least 5%, matching RALPS's allowed_vc_increase=0.05 parameter that defines the tolerated VC increase not penalized by the variation loss. Alongside VC metrics, compute replicate cross-correlation of reference samples per batch (scipy/numpy Pearson correlations) to confirm that VC control does not come at the cost of lost replicate similarity. Compare these criteria across all methods (RALPS, ComBat, EigenMS, LEV+EIG, PQN+POW, WaveICA, NormAE) and aggregate into a summary CSV and a multi-panel figure (e.g. figure2_criteria.png) with matplotlib/seaborn. The rationale is that a normalization that removes batch effects by inflating variance destroys quantitative information; the correct method must jointly suppress batch bias, keep mean batch VC low, minimize the >=5% VC-inflated feature fraction, and preserve spectral shape qualitatively in box/spectra plots.

## Related tools

- **RALPS** (Reference method whose variation-loss term (allowed_vc_increase) defines the 5% VC-inflation tolerance used as the evaluation criterion) — https://github.com/zamboni-lab/RALPS
- **pandas** (Load data matrix, batch info file, and write the criteria summary CSV) — https://github.com/zamboni-lab/RALPS
- **numpy** (Compute per-feature VCs and VC-inflation fractions on raw vs normalized matrices)
- **scipy** (Pearson replicate cross-correlations of reference samples used jointly with VC metrics)
- **matplotlib** (Render the multi-panel criteria figure (figure2_criteria.png))
- **seaborn** (Box and spectra plots for qualitative spectrum preservation)
- **PyTorch** (Backend executing RALPS normalization whose checkpoints are compared on VC criteria) — https://github.com/zamboni-lab/RALPS

## Examples

```
import pandas as pd, numpy as np; raw, norm = pd.read_csv('raw.csv', index_col=0), pd.read_csv('normalized.csv', index_col=0); infl = ((norm.std() / norm.mean()) / (raw.std() / raw.mean()) - 1 >= 0.05).mean()
```

## Evaluation signals

- The fraction of features with VC inflated by >=5% is reported for every method and RALPS is expected to show the lowest inflation among ComBat, EigenMS, LEV+EIG, PQN+POW, WaveICA and NormAE in the two-reference-group scenario
- Mean batch VC of reference samples after normalization does not exceed the pre-normalization value beyond the allowed_vc_increase tolerance (0.05)
- Replicate cross-correlation of reference samples across batches is maintained (Pearson correlation computed with scipy/numpy)
- Normalized matrix retains the raw matrix dimensions and sample_id index after masking values below min_relevant_intensity (1000) with zeros
- Summary CSV in out_path contains one row per method covering all four criteria (cross-correlation, mean batch VC, >=5% VC-inflated fraction, qualitative preservation)

## Limitations

- VC metrics depend on reference samples (e.g. NIST 1950 SRM or pooled samples) being present in every batch; designs with different reference sets per batch need the flexible config supported by RALPS
- Missing values are replaced with min_relevant_intensity before normalization and masked with zeros afterwards, which can distort VCs of low-intensity features
- The >=5% threshold mirrors RALPS's default allowed_vc_increase; applying it to other pipelines imposes this paper's tolerance on unrelated designs

## Evidence

- [abstract] RALPS builds on deep adversarial learning with a three-term loss function that mitigates batch effects while preserving biological identity, spectral properties and coefficients of variation.: "RALPS builds on deep adversarial learning with a three-term loss function that mitigates batch effects while preserving biological identity, spectral properties and coefficients of variation."
- [abstract] Using two large metabolomics datasets, we showcase the superior performance of RALPS as compared with six state-of-the-art methods for batch correction.: "Using two large metabolomics datasets, we showcase the superior performance of RALPS as compared with six state-of-the-art methods for batch correction."
- [readme] RALPS tries to reconstruct the original data while (i) removing interbatch differences on the basis of replicate measurements across batches and (ii) avoiding an expansion of the overall variance: "RALPS tries to reconstruct the original data while (i) removing interbatch differences on the basis of replicate measurements across batches and (ii) avoiding an expansion of the overall variance"
- [readme] fraction of sample's VC increase allowed (not contributing to the variation loss): "fraction of sample's VC increase allowed (not contributing to the variation loss)"
