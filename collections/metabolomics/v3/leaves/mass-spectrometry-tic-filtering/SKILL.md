---
name: mass-spectrometry-tic-filtering
description: Use when raw mass spectra have variable total ion counts and ragged,
  non-uniform m/z grids, and you need a TIC-filtered, uniformly binned feature matrix
  (CSV) for CNN/Transformer classifiers.
when_to_use_negative:
- Input is already a uniform feature table or is already TIC normalized, as the MSMCE
  repo expects ('the input of each dataset is tic normalized in this implementation')
- Data requiring retention-time-aware preprocessing beyond simple m/z binning (the
  rt_binning_window parameter targets RT data, not spectral TIC quality control)
- Spectra from instruments or acquisition modes where low-TIC spectra carry real biological
  signal and a hard 1×10^4 TIC cutoff would discard valid measurements
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0081
tools:
- name: numpy
  role: Compute per-spectrum TIC values, apply the 1×10^4 threshold filter, and bin
    intensities into the 0.1-Da m/z grid
- name: pandas
  role: Assemble the binned spectra as rows of the per-file feature matrix and write
    it to CSV for model training
- name: MSMCE (WoodFY/MSMCE)
  role: Downstream consumer of the TIC-filtered, 0.1-Da-binned feature matrices; its
    exp_multi_channel_embedding.py accepts spectrum_dim and bin_size parameters matching
    this preprocessing
  repo: https://github.com/WoodFY/MSMCE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pone.0321239
    title: 'MSMCE: A novel representation module for classification of raw mass spectrometry
      data'
  source_content_hash: sha256:dd24cbd9984aa9a4fb70ef7ecd5b5159a596d86d051bca1814f6930a95447b98
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WoodFY/MSMCE
  tools:
  - MSMCE (WoodFY/MSMCE)
  - Python (numpy/pandas)
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0081
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# mass-spectrometry-tic-filtering

## Summary

Preprocess raw SpiderMass-style mass spectra (e.g., the Canine Sarcoma dataset) by removing low-quality spectra with total ion count (TIC) below 1×10^4 and binning retained spectra along the m/z axis at 0.1 Da so every spectrum becomes a fixed-length feature vector for downstream deep-learning classifiers like MSMCE-ResNet50.

## When to use

Use when raw mass spectra have variable total ion counts and ragged, non-uniform m/z grids, and you need a TIC-filtered, uniformly binned feature matrix (CSV) for CNN/Transformer classifiers.

## When NOT to use

- Input is already a uniform feature table or is already TIC normalized, as the MSMCE repo expects ('the input of each dataset is tic normalized in this implementation')
- Data requiring retention-time-aware preprocessing beyond simple m/z binning (the rt_binning_window parameter targets RT data, not spectral TIC quality control)
- Spectra from instruments or acquisition modes where low-TIC spectra carry real biological signal and a hard 1×10^4 TIC cutoff would discard valid measurements

## Inputs

- Raw SpiderMass mass spectra files (m/z and intensity pairs), e.g., from the Canine Sarcoma dataset
- Bin width parameter (bin_size, default 0.1 Da)
- Target spectrum dimension (e.g., 15000 for canine_sarcoma_posion, 12000 for nsclc/crlm, 9900 for rcc_posion)

## Outputs

- A 2-D feature matrix per MS file (rows = retained spectra with TIC ≥ 1e4, columns = 0.1-Da m/z bins)
- CSV feature matrix file, e.g., canine_sarcoma_feature_matrix.csv, consumed by downstream model training

## How to apply

Parse each spectrum's m/z–intensity pairs from the raw SpiderMass MS files, compute each spectrum's TIC, and discard spectra with TIC below 1×10^4, keeping only high-quality spectra. Bin the retained spectra along the m/z axis with a 0.1 Da bin width (the pipeline's default bin_size) so that every spectrum is represented by a fixed number of m/z features — e.g., spectrum_dim 15000 for Canine Sarcoma, 12000 for NSCLC/CRLM, 9900 for RCC. Assemble each processed spectrum as one row of a 2-D feature matrix per MS file (rows = retained spectra, columns = 0.1-Da bins) and write it to CSV such as canine_sarcoma_feature_matrix.csv. TIC normalization is applied upstream of model training, matching the MSMCE repo convention that dataset inputs are TIC normalized with the --use_normalization flag.

## Related tools

- **numpy** (Compute per-spectrum TIC values, apply the 1×10^4 threshold filter, and bin intensities into the 0.1-Da m/z grid)
- **pandas** (Assemble the binned spectra as rows of the per-file feature matrix and write it to CSV for model training)
- **MSMCE (WoodFY/MSMCE)** (Downstream consumer of the TIC-filtered, 0.1-Da-binned feature matrices; its exp_multi_channel_embedding.py accepts spectrum_dim and bin_size parameters matching this preprocessing) — https://github.com/WoodFY/MSMCE

## Examples

```
python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset canine_sarcoma_posion --spectrum_dim 15000 --bin_size 0.1 --num_classes 12 --use_normalization
```

## Evaluation signals

- Every retained row corresponds to a spectrum whose TIC is ≥ 1×10^4; recompute TIC per row and assert none falls below the threshold
- All rows of the output matrix have an identical, fixed number of columns equal to the 0.1-Da binned m/z dimension (e.g., 15000 for canine_sarcoma_posion)
- Row count of the CSV equals the number of spectra surviving the TIC filter (raw spectra minus those with TIC < 1e4)
- Binned intensity values are consistent with the original spectra: summing bin intensities should approximate the original (filtered) TIC
- The CSV loads cleanly into the downstream MSMCE pipeline with bin_size 0.1 and --use_normalization, and training runs without dimension-mismatch errors

## Limitations

- Embedding dimensions and parameters (including spectrum_dim) require dataset-specific tuning; there is no adaptive feature selection mechanism
- Downstream MSMCE performance may be limited for severely imbalanced class distributions or very small sample sizes, so aggressive TIC filtering of already-scarce spectra can worsen this
- The fixed 0.1-Da binning and hard TIC cutoff are dataset-level choices; the article does not validate interpretability of which m/z regions survive filtering

## Evidence

- [results] TIC threshold and binning rationale: [evidence span withheld — anchor status: not_found]
- [readme] TIC-normalized input convention in the official repo: "Note that the input of each dataset is tic normalized in this implementation."
- [readme] 0.1 Da default bin size parameter: "bin_size           | Size of bins for spectrum data (default: `0.1`)."
- [results] Canine Sarcoma as an exemplar dataset for this preprocessing: "experiments were conducted on the Canine Sarcoma dataset"
