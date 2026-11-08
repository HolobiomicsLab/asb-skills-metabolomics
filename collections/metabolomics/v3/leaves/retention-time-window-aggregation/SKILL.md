---
name: retention-time-window-aggregation
description: Use when raw LC-MS profile spectra must be converted to a fixed-size
  2D intensity matrix (m/z x RT windows) as input to CNN/LSTM/Transformer classifiers,
  and the m/z and RT dimensions vary between acquisitions.
when_to_use_negative:
- The data is already a processed 2D feature table (e.g., the preprocessed files distributed
  via the repository's Google Drive link) — re-binning would double-process it
- The spectra are MS/MS tandem spectra intended for embedding methods like Spec2Vec
  or MS2DeepScore rather than 1D LC-MS profile spectra
- The acquisition has no meaningful retention-time dimension (e.g., direct infusion),
  so 10 s RT window aggregation is not applicable
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: numpy
  role: array operations for m/z binning, per-window intensity summing/averaging,
    and matrix stacking
- name: pandas
  role: reading raw spectra tables and writing the 2D binned/normalized feature matrix
- name: pytorch
  role: consumes the normalized 2D matrix as model input tensors in the MSMCE training
    pipeline (exp_multi_channel_embedding.py)
  repo: https://github.com/WoodFY/MSMCE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pone.0321239
    title: 'MSMCE: A novel representation module for classification of raw mass spectrometry
      data'
  source_content_hash: sha256:f0a9dc79081ddaf532940d06ff44b2b7ca97785658300b62e2c8d7b7db730323
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WoodFY/MSMCE
  tools:
  - numpy
  - pandas
  - pytorch
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# retention-time-window-aggregation

## Summary

Converts raw LC-MS mass spectra (e.g., NSCLC, CRLM, RCC datasets) into a fixed 2D feature matrix by binning the m/z axis at 0.1 Da and aggregating spectra within 10-second retention-time windows, with TIC normalization prior to deep-learning model input in the MSMCE pipeline.

## When to use

Use when raw LC-MS profile spectra must be converted to a fixed-size 2D intensity matrix (m/z x RT windows) as input to CNN/LSTM/Transformer classifiers, and the m/z and RT dimensions vary between acquisitions.

## When NOT to use

- The data is already a processed 2D feature table (e.g., the preprocessed files distributed via the repository's Google Drive link) — re-binning would double-process it
- The spectra are MS/MS tandem spectra intended for embedding methods like Spec2Vec or MS2DeepScore rather than 1D LC-MS profile spectra
- The acquisition has no meaningful retention-time dimension (e.g., direct infusion), so 10 s RT window aggregation is not applicable

## Inputs

- Raw LC-MS mass spectra files from NSCLC, CRLM, RCC, or Canine Sarcoma datasets
- CLI parameters: bin_size (default 0.1 Da), rt_binning_window (default 10 s), spectrum_dim (12000 NSCLC/CRLM, 9900 RCC, 15000 Canine Sarcoma)

## Outputs

- 2D feature matrix (CSV) with rows = representative mass spectra per 10 s RT window and columns = 0.1-Da m/z bins
- TIC-normalized input matrix ready for deep-learning model training

## How to apply

Load the raw mass spectra from the input MS file, then bin along the m/z axis with a bin width of 0.1 Da (the repository's `bin_size` default) to fix the feature dimension (e.g., spectrum_dim 12000 for NSCLC/CRLM, 9900 for RCC, 15000 for Canine Sarcoma). Aggregate all spectra falling inside a 10-second retention-time binning window by summing and then averaging the intensities of each 0.1-Da m/z bin across those spectra, yielding one representative mass-spectrum sample per RT window. Stack the per-RT-window representative spectra as rows into a 2D feature matrix whose columns are the binned m/z features. Apply TIC (total ion count) normalization to the matrix to reduce dynamic range and improve cross-sample comparability — the README states 'the input of each dataset is tic normalized in this implementation' — before feeding the matrix to the model via the `--use_normalization` flag.

## Related tools

- **numpy** (array operations for m/z binning, per-window intensity summing/averaging, and matrix stacking)
- **pandas** (reading raw spectra tables and writing the 2D binned/normalized feature matrix)
- **pytorch** (consumes the normalized 2D matrix as model input tensors in the MSMCE training pipeline (exp_multi_channel_embedding.py)) — https://github.com/WoodFY/MSMCE

## Examples

```
python exp_mass_spectra_multi_channel_embedding.py --dataset nsclc --spectrum_dim 12000 --bin_size 0.1 --rt_binning_window 10 --use_normalization
```

## Evaluation signals

- Output CSV has rows equal to the number of RT windows retained per file
- Output CSV has columns equal to the dataset's spectrum_dim (12000 for NSCLC/CRLM, 9900 for RCC, 15000 for Canine Sarcoma per the repository CLI)
- All normalized intensity values are finite after TIC normalization
- Exactly one representative mass-spectrum sample is produced per 10-second RT binning window
- The `--use_normalization` flag is set in the training command, matching the README note that input is TIC normalized

## Limitations

- spectrum_dim and other parameters are dataset-specific (12000/9900/15000 bins) and must be tuned per dataset
- Downstream classification performance may fail on severely imbalanced class distributions or when sample size is too small, per the article's discussion
- The pipeline yields a matrix representation without biological/clinical interpretability of the aggregated features (e.g., which m/z regions contribute to classification), a stated limitation of the study

## Evidence

- [methods] raw mass spectra are binned along the m/z axis with a bin width of 0.1 Da, and mass spectra are aggregated within 10-second RT binning windows: [evidence span withheld — anchor status: not_found]
- [readme] bin_size | Size of bins for spectrum data (default: `0.1`).: "bin_size           | Size of bins for spectrum data (default: `0.1`)."
- [readme] rt_binning_window | Retention time binning window size (default: `10`second).: "rt_binning_window  | Retention time binning window size (default: `10`second)."
- [readme] Note that the input of each dataset is tic normalized in this implementation.: "Note that the input of each dataset is tic normalized in this implementation."
