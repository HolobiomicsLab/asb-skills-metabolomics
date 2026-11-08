---
name: mz-axis-binning-uniform-dimension
description: Use when raw SpiderMass mass spectra (m/z, intensity pairs) have variable
  lengths and must become a fixed-dimension feature matrix (0.1-Da bins, TIC >= 1e4)
  for deep model input.
when_to_use_negative:
- The input is already a uniform binned feature table (e.g., previously exported CSV
  with fixed spectrum_dim) ready for model training
- Working with MS/MS spectra where learned spectral embeddings (e.g., Spec2Vec, MS2DeepScore-style)
  rather than raw binned profiles are the target representation
- Data requires retention-time-resolved handling beyond simple m/z profile binning
  (the repo exposes a separate rt_binning_window parameter, default 10 s, for that
  purpose)
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
tools:
- name: numpy
  role: Array computation for TIC calculation and m/z binning of spectra into fixed-dimension
    vectors
- name: pandas
  role: Assembling binned spectra as rows of the 2D feature matrix and writing per-file
    CSV outputs
- name: WoodFY/MSMCE
  role: Reference implementation whose data-loading configuration (bin_size=0.1, spectrum_dim,
    TIC-normalized inputs) defines the expected output schema of this preprocessing
    step
  repo: https://github.com/WoodFY/MSMCE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pone.0321239
    title: 'MSMCE: A novel representation module for classification of raw mass spectrometry
      data'
  source_content_hash: sha256:a3fbe9bf939fb9e53f4d04bcebb57750f5b4e75de48c89dd7bb094a7b374d1e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WoodFY/MSMCE
  tools:
  - Python (numpy/pandas)
  - WoodFY/MSMCE
  - numpy
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mz-axis-binning-uniform-dimension

## Summary

Converts heterogeneous-length raw SpiderMass mass spectra into a uniform 2D feature matrix by discarding low-quality spectra with TIC below 1×10^4 and binning the m/z axis at 0.1 Da, producing fixed-dimension inputs (e.g., spectrum_dim=15000 for Canine Sarcoma) for downstream deep learning classifiers.

## When to use

Use when raw SpiderMass mass spectra (m/z, intensity pairs) have variable lengths and must become a fixed-dimension feature matrix (0.1-Da bins, TIC >= 1e4) for deep model input.

## When NOT to use

- The input is already a uniform binned feature table (e.g., previously exported CSV with fixed spectrum_dim) ready for model training
- Working with MS/MS spectra where learned spectral embeddings (e.g., Spec2Vec, MS2DeepScore-style) rather than raw binned profiles are the target representation
- Data requires retention-time-resolved handling beyond simple m/z profile binning (the repo exposes a separate rt_binning_window parameter, default 10 s, for that purpose)

## Inputs

- Raw SpiderMass mass spectra files (m/z, intensity pairs) from public datasets such as Canine Sarcoma, NSCLC, CRLM, or RCC
- TIC filtering threshold (1×10^4) and m/z bin width (0.1 Da) as pipeline parameters

## Outputs

- A per-file 2D feature matrix CSV (e.g., canine_sarcoma_feature_matrix.csv) with one row per retained spectrum and a fixed number of columns equal to the 0.1-Da binned m/z dimension
- Processed datasets placed in the datasets/ folder for MSMCE model training

## How to apply

Load raw SpiderMass spectra from the dataset files (e.g., Canine Sarcoma), parsing each spectrum's m/z and intensity pairs. Compute each spectrum's total ion count (TIC) and discard spectra with TIC below 1×10^4 so only high-quality spectra are retained. Bin the retained spectra along the m/z axis with a 0.1 Da bin width (the MSMCE repo's default bin_size) so every spectrum occupies the same fixed number of m/z features; this choice sets the downstream spectrum_dim (12000 for NSCLC/CRLM, 9900 for RCC, 15000 for Canine Sarcoma). Apply TIC normalization to the input as the implementation expects, then assemble each processed spectrum as one row of a 2D feature matrix (rows = retained spectra, columns = 0.1-Da bins). Write one feature matrix per MS file to CSV (e.g., canine_sarcoma_feature_matrix.csv) into the datasets/ folder, where the MSMCE training script consumes it with matching spectrum_dim.

## Related tools

- **numpy** (Array computation for TIC calculation and m/z binning of spectra into fixed-dimension vectors)
- **pandas** (Assembling binned spectra as rows of the 2D feature matrix and writing per-file CSV outputs)
- **WoodFY/MSMCE** (Reference implementation whose data-loading configuration (bin_size=0.1, spectrum_dim, TIC-normalized inputs) defines the expected output schema of this preprocessing step) — https://github.com/WoodFY/MSMCE

## Examples

```
python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset canine_sarcoma_posion --num_classes 12 --in_channels 1 --spectrum_dim 15000 --bin_size 0.1 --k_folds 6 --epochs 64 --use_normalization
```

## Evaluation signals

- Every row of the output feature matrix corresponds to a retained spectrum with TIC >= 1e4; no below-threshold spectra remain
- All rows have an identical, fixed number of columns equal to the 0.1-Da binned m/z dimension matching the spectrum_dim used in training (e.g., 15000 for Canine Sarcoma, 12000 for NSCLC/CRLM, 9900 for RCC)
- Bin width equals 0.1 Da as in the pipeline described for SpiderMass datasets and the repo's default bin_size
- One CSV feature matrix is produced per MS input file and loads without shape errors into exp_multi_channel_embedding.py with the matching --spectrum_dim
- Downstream training runs converge with input TIC-normalized as the implementation expects

## Limitations

- Embedding dimensions and related parameters require dataset-specific tuning; there is no adaptive feature selection mechanism
- Performance may be limited for severely imbalanced class distributions or when sample size is too small for deep models to learn representations
- Direct biological interpretation of learned m/z features (which regions contribute to classification, biomarker correlation) is not addressed by this preprocessing approach

## Evidence

- [readme] bin width parameter: "bin_size           | Size of bins for spectrum data (default: `0.1`)."
- [readme] TIC-normalized inputs: "The required data files should be put into `datasets/` folder. Note that the input of each dataset is tic normalized in this implementation."
- [results] Canine Sarcoma experiments: "experiments were conducted on the Canine Sarcoma dataset"
- [discussion] dataset-specific tuning limitation: "the choice of embedding dimensions and parameter optimization still require dataset-specific tuning."
