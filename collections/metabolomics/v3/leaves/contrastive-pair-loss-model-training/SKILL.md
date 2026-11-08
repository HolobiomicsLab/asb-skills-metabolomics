---
name: contrastive-pair-loss-model-training
description: 'Use when reproducing the GLEAMS embedder training procedure: you have
  MassIVE-KB PSM pairs (grouped per precursor charge state) and need to train a Siamese
  network with contrastive loss to a 32-dimensional embedding space.'
when_to_use_negative:
- 'You only need to apply an already-trained embedding: the released gleams_82c0124b.hdf5
  weights exist for this and no retraining is required.'
- Your input is an unlabeled spectra collection with no PSM annotations — pair construction
  requires peptide-identified spectra to define positives and negatives.
- You are using non-Linux or non-CUDA hardware; GLEAMS explicitly requires Python
  3.8, Linux, and a CUDA-enabled GPU.
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0121
tools:
- name: GLEAMS
  role: Provides the Siamese embedder architecture (precursor/fragment/reference-spectrum
    branches, 32-dim output, L2 regularization) and its contrastive-loss pair network;
    training configuration is set in gleams/config.py
  repo: https://github.com/bittremieux/GLEAMS
- name: conda
  role: Creates the environment with compiler tools and GPU runtime needed to train
    the network (environment.yml)
  repo: https://raw.githubusercontent.com/bittremieux/GLEAMS/master/environment.yml
- name: pip
  role: Installs the GLEAMS package from the GitHub repository for training runs
  repo: https://github.com/bittremieux/GLEAMS.git
- name: Python 3.8
  role: Required runtime version for training the GLEAMS network
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-022-01496-1
    title: A learned embedding for efficient joint analysis of millions of mass spectra
  source_content_hash: sha256:869c116934bdd356877e591f4e6f2bf0692aba916d929f48a576ee4066bedccb
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# contrastive-pair-loss-model-training

## Summary

Trains GLEAMS's VGG-inspired Siamese neural network with a contrastive loss on balanced positive/negative MS/MS spectra pairs to produce a 32-dimensional learned embedding in which spectra from the same peptide lie close together.

## When to use

Use when reproducing the GLEAMS embedder training procedure: you have MassIVE-KB PSM pairs (grouped per precursor charge state) and need to train a Siamese network with contrastive loss to a 32-dimensional embedding space.

## When NOT to use

- You only need to apply an already-trained embedding: the released gleams_82c0124b.hdf5 weights exist for this and no retraining is required.
- Your input is an unlabeled spectra collection with no PSM annotations — pair construction requires peptide-identified spectra to define positives and negatives.
- You are using non-Linux or non-CUDA hardware; GLEAMS explicitly requires Python 3.8, Linux, and a CUDA-enabled GPU.

## Inputs

- MassIVE-KB (v1) Human HCD Spectral Library PSM metadata TSV with 'filename' and 'scan' columns (30 million PSMs, top 100 PSMs for 2.1 million unique precursors)
- Raw MS/MS spectra retrieved per filename/scan from the MassIVE FTP server
- Feature encodings: precursor-mass, fragment-peak, and reference-spectrum vectors per spectrum
- Balanced positive/negative spectra pairs grouped per precursor charge state

## Outputs

- Trained GLEAMS model weights in HDF5 format (e.g. gleams_82c0124b.hdf5)
- Per-iteration training and validation contrastive-loss history
- PNG figure of training and validation loss curves documenting convergence
- 32-dimensional embedding network usable via `gleams embed` to convert mzML spectra to n x 32 NumPy arrays

## How to apply

Load the ~30 million PSMs from the MassIVE-KB (v1) Human HCD Spectral Library candidate library spectra (task 82c0124b) and preprocess each spectrum into the precursor-mass, fragment-peak, and reference-spectrum feature encodings. Build the GLEAMS embedder with three parallel branches (precursor, fragment, reference spectrum) whose outputs are concatenated and mapped through dense layers to a 32-dimensional embedding with L2 regularization, configuration controlled in gleams/config.py. Form balanced positive pairs (spectra from the same peptide) and negative pairs (mismatched spectra), grouped per precursor charge state, and feed them to a Siamese pair network. Compile the Siamese network with the contrastive loss and the rectified Adam (RAdam-style) optimizer and train for 50 iterations on a CUDA-enabled GPU. Save the trained weights to an HDF5 file matching the released naming (e.g. gleams_82c0124b.hdf5), record per-iteration train/validation loss history, and plot both loss curves as a PNG to document convergence.

## Related tools

- **GLEAMS** (Provides the Siamese embedder architecture (precursor/fragment/reference-spectrum branches, 32-dim output, L2 regularization) and its contrastive-loss pair network; training configuration is set in gleams/config.py) — https://github.com/bittremieux/GLEAMS
- **conda** (Creates the environment with compiler tools and GPU runtime needed to train the network (environment.yml)) — https://raw.githubusercontent.com/bittremieux/GLEAMS/master/environment.yml
- **pip** (Installs the GLEAMS package from the GitHub repository for training runs) — https://github.com/bittremieux/GLEAMS.git
- **Python 3.8** (Required runtime version for training the GLEAMS network)

## Evaluation signals

- The trained weights file (e.g. gleams_82c0124b.hdf5) loads successfully and matches the released-weights naming convention.
- The held-out validation contrastive loss decreases over the 50 training iterations and converges at or below the training-loss plateau.
- The training/validation loss curves are exported as a PNG and visually match the published convergence figure.
- Positive and negative pairs are balanced and grouped per precursor charge state (same-peptide spectra as positives, mismatched spectra as negatives).
- The final embedding layer outputs exactly 32 dimensions with L2 regularization.

## Limitations

- GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU.
- The 30 million MassIVE-KB training PSMs are not a single download: spectra must be fetched individually from the MassIVE FTP server using the 'filename' and 'scan' metadata columns.
- Model weights are distributed via Git LFS, which can exceed data-quota bandwidth; clone with GIT_LFS_SKIP_SMUDGE=1 and download the weights manually to data/.
- Only spectra generated by the same peptide serve as positives, and pairing is per precursor charge state — cross-charge pairing is outside the training scheme described.

## Evidence

- [intro] GLEAMS encodes mass spectra as vectors of features and feeds them to a neural network to embed them into a 32-dimensional space in which spectra generated by the same peptide are close together.: "GLEAMS encodes mass spectra as vectors of features and feeds them to a neural network to embed them into a 32-dimensional space in which spectra generated by the same peptide are close together."
- [readme] GLEAMS was trained on 30 million PSMs from the [MassIVE-KB (v1) dataset]: "GLEAMS was trained on 30 million PSMs from the [MassIVE-KB (v1) dataset]"
- [readme] Full configuration of GLEAMS, including various configurations to train the neural network, can be modified in the `gleams/config.py` file.: "Full configuration of GLEAMS, including various configurations to train the neural network, can be modified in the `gleams/config.py` file."
- [readme] GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU.: "GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU."
- [readme] Manually download the GLEAMS model weights from the latest release ... Move the model weights file to the `data/` directory in the cloned GLEAMS repository.: "Manually download the [GLEAMS model weights from the latest release](https://github.com/bittremieux/GLEAMS/releases/download/v0.3/gleams_82c0124b.hdf5)."
