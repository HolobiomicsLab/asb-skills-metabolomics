---
name: mass-spectrometry-spectrum-preprocessing
description: Use when raw MS/MS peak files (e.g. mzML) must be converted into fixed-dimensional
  feature encodings/embeddings for downstream clustering or comparison of millions
  of spectra, following the GLEAMS learned-embedding approach.
when_to_use_negative:
- Spectra are already embedded and the goal is grouping — use `gleams cluster` (hierarchical
  clustering with --distance_threshold 0.3) instead of re-embedding.
- The compute environment is not Linux with a CUDA-enabled GPU running Python 3.8
  — GLEAMS requires all three and will not run.
- The task is peptide identification/searching (e.g. MSGF+ PSM assignment) rather
  than embedding-based joint analysis or clustering of already-identified spectra.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: GLEAMS
  role: Provides the `gleams embed` command that converts MS/MS spectra in peak files
    to 32-dimensional embeddings; its network configuration (feature encodings, training)
    is set in gleams/config.py.
  repo: https://github.com/bittremieux/GLEAMS
- name: conda
  role: Creates the gleams environment with compiler tools and GPU runtime from environment.yml.
- name: pip
  role: Installs GLEAMS from the GitHub repository (pip install git+... or pip install
    . after manually placing weights in data/).
  repo: https://github.com/bittremieux/GLEAMS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-022-01496-1
    title: A learned embedding for efficient joint analysis of millions of mass spectra
  source_content_hash: sha256:ccb8171efd5e5546d2e58b312a898aba2da29e3b411aa8d1d6367d46c439a85c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GLEAMS
  tools:
  - GLEAMS
  - Python
  - conda
  - pip
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# mass-spectrometry-spectrum-preprocessing

## Summary

Preprocess and encode MS/MS spectra into GLEAMS's precursor-mass, fragment-peak, and reference-spectrum feature encodings, then embed them into a 32-dimensional space in which spectra from the same peptide are close together, using the GLEAMS neural-network embedder.

## When to use

Use when raw MS/MS peak files (e.g. mzML) must be converted into fixed-dimensional feature encodings/embeddings for downstream clustering or comparison of millions of spectra, following the GLEAMS learned-embedding approach.

## When NOT to use

- Spectra are already embedded and the goal is grouping — use `gleams cluster` (hierarchical clustering with --distance_threshold 0.3) instead of re-embedding.
- The compute environment is not Linux with a CUDA-enabled GPU running Python 3.8 — GLEAMS requires all three and will not run.
- The task is peptide identification/searching (e.g. MSGF+ PSM assignment) rather than embedding-based joint analysis or clustering of already-identified spectra.

## Inputs

- MS/MS spectra in peak files (e.g. mzML) to be embedded
- MassIVE-KB (v1) metadata TSV with 'filename' and 'scan' columns for the 30 million Human HCD training PSMs
- Pretrained GLEAMS model weights (HDF5, e.g. gleams_82c0124b.hdf5) or a training pair set with positive/negative spectra pairs per charge state

## Outputs

- GLEAMS_embed.npy: two-dimensional NumPy array of n x 32 embeddings, one row per MS/MS spectrum
- GLEAMS_embed.parquet: tabular metadata corresponding to the embedded spectra
- (when training) model weights saved to HDF5 (e.g. gleams_82c0124b.hdf5) plus per-iteration training/validation contrastive loss history and loss-curve PNG

## How to apply

First retrieve PSM metadata as a zipped TSV from the MassIVE-KB (v1) Human HCD Spectral Library (30 million PSMs, maximum top 100 PSMs for 2.1 million unique precursors), using the 'filename' column to fetch corresponding spectra from the MassIVE FTP server and the 'scan' column to extract the required scans. Preprocess each spectrum into the three feature encodings consumed by the network: precursor-mass features, fragment-peak features, and reference-spectrum features. For inference, run `gleams embed` on the mzML peak files, which exports a two-dimensional NumPy array of dimension n x 32 (GLEAMS_embed.npy) plus per-spectrum metadata (GLEAMS_embed.parquet). If retraining rather than using released weights (gleams_82c0124b.hdf5), form balanced positive pairs (spectra from the same peptide) and negative pairs grouped per precursor charge state, compile the Siamese pair network with contrastive loss and the RAdam optimizer, and train 50 iterations on a GPU; network configuration lives in gleams/config.py.

## Related tools

- **GLEAMS** (Provides the `gleams embed` command that converts MS/MS spectra in peak files to 32-dimensional embeddings; its network configuration (feature encodings, training) is set in gleams/config.py.) — https://github.com/bittremieux/GLEAMS
- **conda** (Creates the gleams environment with compiler tools and GPU runtime from environment.yml.)
- **pip** (Installs GLEAMS from the GitHub repository (pip install git+... or pip install . after manually placing weights in data/).) — https://github.com/bittremieux/GLEAMS

## Examples

```
gleams embed *.mzML --embed_name GLEAMS_embed
```

## Evaluation signals

- GLEAMS_embed.npy has shape (n, 32), where n equals the number of MS/MS spectra read from the input mzML files, and GLEAMS_embed.parquet contains one metadata row per embedded spectrum.
- The trained weights file loads correctly from HDF5 (e.g. gleams_82c0124b.hdf5 moved into the data/ directory).
- When retraining, the validation contrastive loss decreases and converges over the 50 iterations, ideally at or below the training-loss plateau, matching the published loss-curve figure.
- Embeddings place spectra generated by the same peptide close together in the 32-dimensional space, as verified by downstream clustering into per-peptide spectrum clusters.

## Limitations

- GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU.
- The 30-million-PSM training data are not available as a single download; they must be assembled from the MassIVE-KB Human HCD Spectral Library metadata TSV plus per-task mzTab files and MassIVE FTP spectra retrieval.
- Installation can fail with a Git LFS 'data quota' error when downloading model weights; work around with GIT_LFS_SKIP_SMUDGE=1 clone and manual weights download from the v0.3 release.

## Evidence

- [readme] GLEAMS encodes mass spectra as vectors of features and feeds them to a neural network to embed them into a 32-dimensional space in which spectra generated by the same peptide are close together.: "GLEAMS encodes mass spectra as vectors of features and feeds them to a neural network to embed them into a 32-dimensional space in which spectra"
- [readme] Embedding output format is an n x 32 NumPy array with parquet metadata.: "This will read the MS/MS spectra from all matched mzML files and export the results to a two-dimensional NumPy array of dimension _n_ x 32 in file `GLEAMS_embed.npy`"
- [readme] Training data provenance.: "GLEAMS was trained on 30 million PSMs from the [MassIVE-KB (v1) dataset](https://massive.ucsd.edu/ProteoSAFe/static/massive-kb-libraries.jsp)."
- [methods] The embed command converts peak-file spectra to 32-dimensional embeddings.: "command to convert MS/MS spectra in peak files to 32-dimensional embeddings"
- [readme] Training configuration is editable in gleams/config.py.: "Full configuration of GLEAMS, including various configurations to train the neural network, can be modified in the `gleams/config.py` file."
