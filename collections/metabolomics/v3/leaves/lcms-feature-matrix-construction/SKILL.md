---
name: lcms-feature-matrix-construction
description: Use when raw LC-MS mass spectra must be converted into a fixed-dimension
  2D feature matrix (m/z bins × RT windows) for feeding CNN/LSTM/Transformer classifiers,
  and spectrum_dim differs per dataset (e.g. 12000 NSCLC/CRLM, 9900 RCC, 15000 Canine
  Sarcoma).
when_to_use_negative:
- Input is already a processed, TIC-normalized feature table with fixed spectrum_dim
  — re-binning would discard the existing binning.
- Data are MS/MS tandem spectra where Spec2Vec/MS2DeepScore-style embedded representations
  of MS2 vectors are the target, rather than 1D LC-MS profile binning.
- Class distributions are severely imbalanced or sample size is too small — the paper
  warns deep models may fail to learn embedding representations in these extreme cases
  regardless of preprocessing.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: numpy
  role: Binning and averaging mass-spectrum intensities per m/z bin and RT window
    to build the 2D feature matrix
- name: pandas
  role: Loading raw spectra and stacking representative RT-window spectra into the
    feature table
- name: pytorch
  role: Runtime dependency of the MSMCE repository in which this preprocessing (bin_size,
    rt_binning_window, use_normalization flags) is implemented
  repo: https://github.com/WoodFY/MSMCE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pone.0321239
    title: 'MSMCE: A novel representation module for classification of raw mass spectrometry
      data'
  source_content_hash: sha256:f6e36b6678ec670d53dad0e1c09fc8f39b2d90d0d76d924cab0965f13962b8bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WoodFY/MSMCE
  tools:
  - numpy
  - pandas
  - pytorch
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# lcms-feature-matrix-construction

## Summary

Constructs a consistent 2D LC-MS input feature matrix for deep learning by binning raw mass spectra along the m/z axis (0.1 Da bins) and aggregating spectra within 10-second retention-time windows, followed by TIC normalization. This reproduces the raw-data preprocessing used for the NSCLC, CRLM, RCC and Canine Sarcoma datasets in the MSMCE pipeline.

## When to use

Use when raw LC-MS mass spectra must be converted into a fixed-dimension 2D feature matrix (m/z bins × RT windows) for feeding CNN/LSTM/Transformer classifiers, and spectrum_dim differs per dataset (e.g. 12000 NSCLC/CRLM, 9900 RCC, 15000 Canine Sarcoma).

## When NOT to use

- Input is already a processed, TIC-normalized feature table with fixed spectrum_dim — re-binning would discard the existing binning.
- Data are MS/MS tandem spectra where Spec2Vec/MS2DeepScore-style embedded representations of MS2 vectors are the target, rather than 1D LC-MS profile binning.
- Class distributions are severely imbalanced or sample size is too small — the paper warns deep models may fail to learn embedding representations in these extreme cases regardless of preprocessing.

## Inputs

- Raw LC-MS mass spectra files (NSCLC, CRLM, RCC, or Canine Sarcoma datasets)
- CLI parameters: bin_size=0.1 (Da), rt_binning_window=10 (seconds), spectrum_dim per dataset (12000/9900/15000)

## Outputs

- Two-dimensional feature matrix: one representative mass spectrum per 10-second RT window (rows) × 0.1-Da m/z bins (columns)
- TIC-normalized input arrays ready for model input (e.g. .npy/.csv per dataset)

## How to apply

Load raw LC-MS mass spectra and bin intensities along the m/z axis with a bin width of 0.1 Da (the `bin_size` default in the repository CLI). Aggregate spectra within 10-second RT binning windows by summing and then averaging intensity values per m/z bin across all spectra in each window, yielding one representative spectrum sample per RT window. Stack representative samples as rows into a 2D matrix whose columns correspond to the 0.1-Da m/z bins — the resulting dimensionality must match the dataset's `spectrum_dim` (12000 for NSCLC/CRLM, 9900 for RCC, 15000 for Canine Sarcoma). Finally apply TIC (total ion count) normalization, as the repository states 'the input of each dataset is tic normalized in this implementation', controlled by the `--use_normalization` flag. Rationale: a consistent, fixed-length representation without aggressive feature selection lets the downstream MSMCE multi-channel embedding module learn directly from raw spectral data.

## Related tools

- **numpy** (Binning and averaging mass-spectrum intensities per m/z bin and RT window to build the 2D feature matrix)
- **pandas** (Loading raw spectra and stacking representative RT-window spectra into the feature table)
- **pytorch** (Runtime dependency of the MSMCE repository in which this preprocessing (bin_size, rt_binning_window, use_normalization flags) is implemented) — https://github.com/WoodFY/MSMCE

## Examples

```
python exp_mass_spectra_multi_channel_embedding.py --root_dir ../ --save_dir ckpt --model_name ResNet50 --dataset nsclc --in_channels 1 --spectrum_dim 12000 --bin_size 0.1 --rt_binning_window 10 --use_normalization
```

## Evaluation signals

- Row count equals the number of retained 10-second RT windows per file; column count equals the dataset's 0.1-Da m/z bin count (spectrum_dim 12000 for NSCLC/CRLM, 9900 for RCC, 15000 for Canine Sarcoma).
- All normalized intensity values are finite and non-negative after TIC normalization.
- Each row's intensities sum to a comparable total ion count (TIC normalization reduces dynamic range across samples).
- Downstream classifier accuracy on the preprocessed matrix reaches the reported baselines (e.g. MSMCE-ResNet50 accuracy 0.9785 on NSCLC) when trained with the repository's 6-fold cross-validation settings.
- Re-running with identical random_seed (default 3407) and the same bin_size/rt_binning_window reproduces an identical feature matrix.

## Limitations

- Embedding dimensions and parameter optimization (including spectrum_dim and binning choices) require dataset-specific tuning; no adaptive feature selection mechanism exists.
- The m/z binning discards fine mass resolution information; the paper notes it did not investigate biological interpretability of learned features (e.g. which m/z regions contribute to classification).
- Performance may be limited for severely imbalanced class distributions or too-small sample sizes; the raw download provides processed data rather than vendor files, so exact raw-file handling varies at the source.

## Evidence

- [readme] m/z binning with 0.1 Da and 10-second RT window aggregation: "bin_size           | Size of bins for spectrum data (default: `0.1`).
rt_binning_window  | Retention time binning window size (default: `10`second)."
- [readme] TIC normalization of dataset input: "The required data files should be put into `datasets/` folder. Note that the input of each dataset is tic normalized in this implementation."
- [readme] spectrum_dim per dataset in CLI: "python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset rcc_posion --num_classes 2 --k_folds 6 --in_channels 1 --spectrum_dim 9900"
- [intro] learning directly from raw MS data motivation: "This persistent need for effectively learning discriminative representations from data motivates the exploration of alternative methods that can learn representations more directly from raw MS data."
- [discussion] dataset-specific tuning limitation: "the choice of embedding dimensions and parameter optimization still require dataset-specific tuning."
