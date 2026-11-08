---
name: gpu-based-deep-learning-training-curve-visualization
description: 'Use when reproducing or fine-tuning GLEAMS''s spectral embedding model:
  you have MassIVE-KB PSM pairs, need the 32-dimensional Siamese embedder trained
  with contrastive loss, and must verify convergence via loss curves.'
when_to_use_negative:
- You only need to apply the pre-trained embedder — download the released weights
  directly instead of retraining (the README FAQ documents manual weight download
  to avoid Git LFS bandwidth errors)
- Your spectra are not HCD-MS/MS data with known peptide identifications suitable
  for pair construction — contrastive training requires labeled positive/negative
  spectra pairs grouped per precursor charge state
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_3520
tools:
- name: GLEAMS
  role: Siamese contrastive-loss embedder trained on MassIVE-KB pairs; network architecture
    and training configuration defined in gleams/config.py
  repo: https://github.com/bittremieux/GLEAMS
- name: conda
  role: Creates the environment with compiler tools and GPU runtime required for training
  repo: https://raw.githubusercontent.com/bittremieux/GLEAMS/master/environment.yml
- name: pip
  role: Installs GLEAMS from the GitHub repository or a local clone
  repo: https://github.com/bittremieux/GLEAMS.git
- name: Python 3.8
  role: Runtime for the training script, loss history logging, and matplotlib-style
    loss-curve PNG export
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-022-01496-1
    title: A learned embedding for efficient joint analysis of millions of mass spectra
  source_content_hash: sha256:41b0ac573f60b6472ce73b697860756509676b1683e8edf5ae319f4a8a3ba538
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GLEAMS
  tools:
  - GLEAMS
  - Python
  - Python 3.8
  - conda
  - pip
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# gpu-based-deep-learning-training-curve-visualization

## Summary

Trains GLEAMS's Siamese neural network embedder with a contrastive loss on positive/negative MS/MS spectra pairs from MassIVE-KB on a CUDA-enabled GPU, then records and visualizes the per-iteration training and validation loss curves to document convergence.

## When to use

Use when reproducing or fine-tuning GLEAMS's spectral embedding model: you have MassIVE-KB PSM pairs, need the 32-dimensional Siamese embedder trained with contrastive loss, and must verify convergence via loss curves.

## When NOT to use

- You only need to apply the pre-trained embedder — download the released weights directly instead of retraining (the README FAQ documents manual weight download to avoid Git LFS bandwidth errors)
- Your spectra are not HCD-MS/MS data with known peptide identifications suitable for pair construction — contrastive training requires labeled positive/negative spectra pairs grouped per precursor charge state

## Inputs

- MassIVE-KB (v1) Human HCD Spectral Library candidate library spectra (30 million PSMs, zipped TSV metadata with 'filename' and 'scan' columns)
- mzML peak files retrieved from the MassIVE FTP server for the PSM metadata rows
- CUDA-enabled Linux GPU and conda environment with GLEAMS dependencies (Python 3.8)

## Outputs

- Trained Siamese embedder weights in HDF5 format (e.g. gleams_82c0124b.hdf5 placed in the GLEAMS data/ directory)
- Per-iteration training and validation contrastive-loss history
- PNG figure of training and validation loss curves showing convergence
- 32-dimensional embeddings of MS/MS spectra (n x 32 NumPy array, e.g. GLEAMS_embed.npy)

## How to apply

Load the 30 million MassIVE-KB (v1) PSMs from the Human HCD Spectral Library candidate library spectra and preprocess them into the precursor-mass, fragment-peak, and reference-spectrum feature encodings consumed by the network. Construct the VGG-inspired GLEAMS embedder — parallel branches for precursor, fragment, and reference-spectrum features concatenated through dense layers to a 32-dimensional, L2-regularized output — and compile it as a Siamese pair network with a contrastive loss and the rectified Adam optimizer. Form balanced positive pairs (spectra from the same peptide) and negative pairs (mismatched spectra) grouped per precursor charge state, then train on the GPU for 50 iterations, monitoring contrastive loss on the held-out validation split. Save the trained weights to HDF5 (e.g. gleams_82c0124b.hdf5, matching the released v0.3 weights), record per-iteration training/validation loss history, and plot both curves as a PNG to document convergence. Training configuration lives in `gleams/config.py`; correctness is judged by the validation loss decreasing and converging below the training loss plateau over the 50 iterations, as in the published loss-curve figure.

## Related tools

- **GLEAMS** (Siamese contrastive-loss embedder trained on MassIVE-KB pairs; network architecture and training configuration defined in gleams/config.py) — https://github.com/bittremieux/GLEAMS
- **conda** (Creates the environment with compiler tools and GPU runtime required for training) — https://raw.githubusercontent.com/bittremieux/GLEAMS/master/environment.yml
- **pip** (Installs GLEAMS from the GitHub repository or a local clone) — https://github.com/bittremieux/GLEAMS.git
- **Python 3.8** (Runtime for the training script, loss history logging, and matplotlib-style loss-curve PNG export)

## Examples

```
conda env create -f https://raw.githubusercontent.com/bittremieux/GLEAMS/master/environment.yml && conda activate gleams && pip install git+https://github.com/bittremieux/GLEAMS.git
```

## Evaluation signals

- Validation contrastive loss decreases monotonically and converges below the training loss plateau over the 50 iterations, matching the published loss-curve figure
- The trained weights HDF5 file (e.g. gleams_82c0124b.hdf5) loads successfully into the GLEAMS embedder
- Embeddings produced by the trained model are n x 32 NumPy arrays consistent with `gleams embed` output
- Pair construction yields balanced positive/negative sets grouped per precursor charge state from the same MassIVE-KB PSM pool

## Limitations

- Requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU
- The 30-million-PSM training dataset must be assembled piecemeal: MassIVE-KB metadata TSV plus spectra fetched individually by scan number from the MassIVE FTP server; no single download exists
- Git LFS bandwidth limits can block cloning with model weights; clone with GIT_LFS_SKIP_SMUDGE=1 and download weights manually from the v0.3 release

## Evidence

- [intro] GLEAMS encodes mass spectra as feature vectors and uses a neural network to embed them into a 32-dimensional space in which spectra from the same peptide are close together: [evidence span withheld — anchor status: not_found]
- [methods] Contrastive loss trains the Siamese embedder on pairs: [evidence span withheld — anchor status: not_found]
- [methods] Form balanced positive and negative pairs per charge state: [evidence span withheld — anchor status: not_found]
- [methods] Validation criterion: loss curves converge: [evidence span withheld — anchor status: not_found]
- [readme] Manual weights download for LFS quota errors: "Manually download the [GLEAMS model weights from the latest release](https://github.com/bittremieux/GLEAMS/releases/download/v0.3/gleams_82c0124b.hdf5)."
- [readme] Full training configuration in config.py: "Full configuration of GLEAMS, including various configurations to train the neural network, can be modified in the `gleams/config.py` file."
