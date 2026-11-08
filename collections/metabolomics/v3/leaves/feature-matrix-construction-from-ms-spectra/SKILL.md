---
name: feature-matrix-construction-from-ms-spectra
description: Use when raw mass spectra stored as variable-length m/z/intensity pairs
  must be reduced to a fixed-dimensional matrix (TIC filtering + 0.1-Da m/z binning)
  before feeding CNN/LSTM/Transformer classifiers.
when_to_use_negative:
- Input is already a uniform feature table / binned CSV matrix (the repo notes 'the
  input of each dataset is tic normalized in this implementation'), so re-binning
  would be redundant.
- The task concerns the MSMCE representation module or classifier training itself
  — this skill only covers the upstream preprocessing into a fixed-dimension matrix,
  not the multi-channel embedding or model fitting.
tools:
- name: numpy
  role: Computes per-spectrum total ion count and performs the 0.1-Da m/z binning
    into fixed-length vectors
  repo: https://github.com/WoodFY/MSMCE
- name: pandas
  role: Assembles binned spectra as rows of a 2D matrix and writes each file's feature
    matrix to CSV for downstream training
  repo: https://github.com/WoodFY/MSMCE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pone.0321239
    title: 'MSMCE: A novel representation module for classification of raw mass spectrometry
      data'
  source_content_hash: sha256:bfae9eb5ce0244f74ceff6812317aa766eb93cb343158eabde798ffd0b869d82
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WoodFY/MSMCE
  tools:
  - Python (numpy/pandas)
  - numpy
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# feature-matrix-construction-from-ms-spectra

## Summary

Converts raw SpiderMass-style mass spectra into a uniform 2D feature matrix by discarding low-quality spectra (TIC < 1e4) and binning the m/z axis at 0.1 Da, producing fixed-width rows suitable as input to deep learning classifiers such as MSMCE-ResNet50 on the Canine Sarcoma, NSCLC, CRLM, and RCC datasets.

## When to use

Use when raw mass spectra stored as variable-length m/z/intensity pairs must be reduced to a fixed-dimensional matrix (TIC filtering + 0.1-Da m/z binning) before feeding CNN/LSTM/Transformer classifiers.

## When NOT to use

- Input is already a uniform feature table / binned CSV matrix (the repo notes 'the input of each dataset is tic normalized in this implementation'), so re-binning would be redundant.
- The task concerns the MSMCE representation module or classifier training itself — this skill only covers the upstream preprocessing into a fixed-dimension matrix, not the multi-channel embedding or model fitting.

## Inputs

- Raw SpiderMass mass spectra (m/z and intensity pairs) from datasets such as Canine Sarcoma, NSCLC, CRLM, RCC
- TIC threshold parameter (1x10^4) and m/z bin width parameter (bin_size, default 0.1 Da)

## Outputs

- Per-file 2D feature matrix (rows = TIC-retained spectra, columns = 0.1-Da binned m/z features) written to CSV, e.g. canine_sarcoma_feature_matrix.csv
- TIC-normalized matrix consumed by downstream model training (exp_multi_channel_embedding.py)

## How to apply

Load each raw SpiderMass spectrum file and parse its m/z and intensity pairs; compute each spectrum's total ion count (TIC) and discard any spectrum with TIC below 1x10^4, retaining only high-quality spectra. Bin the retained spectra along the m/z axis with a 0.1-Da bin width (the repo's bin_size parameter defaults to 0.1) so every spectrum has the same fixed number of features — e.g. spectrum_dim=12000 for NSCLC/CRLM, 9900 for RCC, and 15000 for Canine Sarcoma. Assemble each processed spectrum as one row of a per-file 2D matrix (rows = retained spectra, columns = 0.1-Da bins) and write it to CSV such as canine_sarcoma_feature_matrix.csv. Because the MSMCE implementation expects inputs that are 'tic normalized', apply TIC normalization to the matrix (e.g. the use_normalization flag) before model training.

## Related tools

- **numpy** (Computes per-spectrum total ion count and performs the 0.1-Da m/z binning into fixed-length vectors) — https://github.com/WoodFY/MSMCE
- **pandas** (Assembles binned spectra as rows of a 2D matrix and writes each file's feature matrix to CSV for downstream training) — https://github.com/WoodFY/MSMCE

## Examples

```
python exp_multi_channel_embedding.py --dataset canine_sarcoma_posion --num_classes 12 --in_channels 1 --spectrum_dim 15000 --bin_size 0.1 --use_normalization
```

## Evaluation signals

- Every output row corresponds to a retained spectrum whose TIC >= 1e4; no row derived from a filtered spectrum remains.
- All rows have an identical column count equal to the 0.1-Da binned m/z dimension (e.g. 12000 for NSCLC/CRLM, 9900 for RCC, 15000 for Canine Sarcoma), matching the spectrum_dim used downstream.
- Output CSVs exist per MS file (e.g. canine_sarcoma_feature_matrix.csv) with rows = spectra and columns = bins, matching the paper's data-processing workflow.
- The matrix fed to training is TIC normalized, consistent with the repository statement that dataset inputs are TIC normalized.
- Bin width check: the bin_size parameter equals 0.1 (the documented default) so feature counts reproduce the published spectrum_dim values.

## Limitations

- The hard TIC cutoff (< 1e4) discards low-intensity spectra, which may remove marginal but informative samples.
- Fixed 0.1-Da binning is dataset-specific: spectrum_dim (12000, 9900, 15000) must be tuned per dataset, as the paper notes embedding dimensions and parameters 'still require dataset-specific tuning'.
- Downstream performance may be limited for severely imbalanced class distributions or very small sample sizes, where deep models may fail to learn embeddings.

## Evidence

- [other] TIC filter and 0.1-Da binning: [evidence span withheld — anchor status: not_found]
- [other] TIC threshold detail: [evidence span withheld — anchor status: not_found]
- [readme] TIC-normalized inputs: "Note that the input of each dataset is tic normalized in this implementation."
- [readme] bin_size default: "bin_size           | Size of bins for spectrum data (default: `0.1`)."
