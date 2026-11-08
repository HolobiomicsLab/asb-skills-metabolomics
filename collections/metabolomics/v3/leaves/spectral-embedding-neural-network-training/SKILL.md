---
name: spectral-embedding-neural-network-training
description: Use when retraining or reproducing the GLEAMS Siamese spectral embedder
  — e.g. ablating precursor/fragment/reference-spectrum feature blocks — from MassIVE-KB
  PSMs rather than running the pretrained model.
when_to_use_negative:
- You only need 32-dimensional embeddings or cluster labels from new spectra — use
  the released pretrained weights with 'gleams embed' and 'gleams cluster' instead
  of retraining.
- The goal is clustering already-computed embeddings — that is the 'gleams cluster
  --distance_threshold 0.3' workflow, not network training.
- No CUDA-enabled GPU or Linux environment is available — GLEAMS requires Python 3.8,
  Linux, and a CUDA GPU for training.
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3474
tools:
- name: GLEAMS
  role: Provides the Siamese network architecture (precursor, fragment, and reference-spectrum
    branches), contrastive-loss pair construction, and training loop configured via
    gleams/config.py
  repo: https://github.com/bittremieux/GLEAMS
- name: conda
  role: Creates the environment with compiler tools and GPU runtime needed to train
    the network
- name: pip
  role: Installs GLEAMS from the repository into the Conda environment
- name: Python 3.8
  role: Runtime for training the embedder and recording loss histories
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1038/s41592-022-01496-1
    title: A learned embedding for efficient joint analysis of millions of mass spectra
  source_content_hash: sha256:66e6cc961a73e12dccbd303aaf75933ad4219af0406ed1df9a8dd2ddf3993a24
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
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# spectral-embedding-neural-network-training

## Summary

Trains the GLEAMS Siamese neural network that embeds MS/MS spectra into a 32-dimensional L2-regularized space using a contrastive loss on positive/negative spectra pairs drawn from 30 million MassIVE-KB PSMs. This is the training/reproduction skill (including feature-ablation retraining), distinct from applying the released pretrained weights.

## When to use

Use when retraining or reproducing the GLEAMS Siamese spectral embedder — e.g. ablating precursor/fragment/reference-spectrum feature blocks — from MassIVE-KB PSMs rather than running the pretrained model.

## When NOT to use

- You only need 32-dimensional embeddings or cluster labels from new spectra — use the released pretrained weights with 'gleams embed' and 'gleams cluster' instead of retraining.
- The goal is clustering already-computed embeddings — that is the 'gleams cluster --distance_threshold 0.3' workflow, not network training.
- No CUDA-enabled GPU or Linux environment is available — GLEAMS requires Python 3.8, Linux, and a CUDA GPU for training.

## Inputs

- MassIVE-KB (v1) Human HCD candidate library spectra metadata TSV (30 million PSMs)
- Corresponding MS/MS peak files retrieved from the MassIVE FTP server via 'filename' and 'scan' columns
- gleams/config.py neural-network configuration (feature blocks, optimizer, iterations)

## Outputs

- Trained Siamese embedder weights saved as an HDF5 file (e.g. gleams_82c0124b.hdf5)
- Per-iteration training and validation contrastive-loss history
- Training/validation loss-curve figure (PNG) documenting convergence

## How to apply

Compile training data from the MassIVE-KB (v1) Human HCD Spectral Library: download the candidate library spectra metadata TSV (30 million PSMs, maximum top 100 PSMs per unique precursor, 2.1 million unique precursors), then retrieve the matching spectra from the MassIVE FTP server by 'filename' and 'scan' columns. Preprocess into three feature encodings — precursor-mass, fragment-peak, and reference-spectrum features — consumed by parallel VGG-inspired branches that concatenate into a final fully-connected layer of dimension 32 with L2 regularization. Form balanced positive pairs (spectra from the same peptide) and negative pairs (mismatched spectra), grouped per precursor charge state, and compile the Siamese pair network with the contrastive loss and the rectified Adam (RAdam-style) optimizer. Configure the network by editing gleams/config.py, train for 50 iterations on a CUDA-enabled GPU while monitoring contrastive loss on the held-out validation split, save weights to an HDF5 file (matching the released gleams_82c0124b.hdf5 naming), and record per-iteration training/validation loss history for a loss-curve figure.

## Related tools

- **GLEAMS** (Provides the Siamese network architecture (precursor, fragment, and reference-spectrum branches), contrastive-loss pair construction, and training loop configured via gleams/config.py) — https://github.com/bittremieux/GLEAMS
- **conda** (Creates the environment with compiler tools and GPU runtime needed to train the network)
- **pip** (Installs GLEAMS from the repository into the Conda environment)
- **Python 3.8** (Runtime for training the embedder and recording loss histories)

## Evaluation signals

- The trained weights file (e.g. gleams_82c0124b.hdf5) loads successfully from disk.
- Validation contrastive loss decreases and converges below the training-loss plateau over the 50 training iterations, reproducing the published loss-curve figure.
- Feature-ablation check: validation loss increases when either the fragment or the reference-spectrum feature block is withheld, matching the Extended Data Fig. 9 ordering (all-features baseline is optimal).
- The embedding output is 32-dimensional (an n x 32 NumPy array, as produced downstream by 'gleams embed').
- Training pairs are balanced positives (same peptide) and negatives (mismatched spectra), grouped per precursor charge state.

## Limitations

- Requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU.
- Training data is not a single download: the 30 million PSMs must be assembled from the MassIVE-KB metadata TSV plus spectra fetched per-file from the MassIVE FTP server.
- Git LFS bandwidth quotas can block installation of the model weights; work around by cloning with GIT_LFS_SKIP_SMUDGE=1 and downloading gleams_82c0124b.hdf5 manually into data/.
- The 30 million PSMs are capped at the maximum top 100 PSMs for 2.1 million unique precursors, a subset of the full 185 million MSGF+ PSMs at 1% FDR.

## Evidence

- [intro] Neural architecture and embedding dimension: [evidence span withheld — anchor status: not_found]
- [readme] Training dataset size and source: "These 30 million PSMs are a subset of all spectrum identifications that were obtained during compilation of the MassIVE-KB resource (maximum top 100 PSMs for 2.1 million unique precursors)."
- [readme] Runtime requirements for training: "GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU."
- [readme] How to configure network training: "Full configuration of GLEAMS, including various configurations to train the neural network, can be modified in the `gleams/config.py` file."
- [readme] Embedding output format downstream of training: "export the results to a two-dimensional NumPy array of dimension _n_ x 32 in file `GLEAMS_embed.npy`, with _n_ the number of MS/MS spectra read from the mzML files."
