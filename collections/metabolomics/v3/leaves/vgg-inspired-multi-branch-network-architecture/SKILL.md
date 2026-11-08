---
name: vgg-inspired-multi-branch-network-architecture
description: Use when reproducing or adapting the GLEAMS neural architecture that
  embeds MS/MS spectra into a 32-dimensional space, or when training a Siamese contrastive
  model on PSM pairs.
when_to_use_negative:
- Input spectra lack reliable peptide identifications (PSMs) — the pair-construction
  step needs same-peptide positives from MassIVE-KB annotations
- The goal is inference/annotation with pre-trained embeddings rather than (re)training
  — use the released gleams_82c0124b.hdf5 weights directly instead
- No CUDA-enabled GPU or Linux environment is available, which the GLEAMS training
  setup requires
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: GLEAMS
  role: Provides the multi-branch embedder architecture, Siamese pair training with
    contrastive loss, and the gleams embed/cluster commands; training configuration
    lives in gleams/config.py
  repo: https://github.com/bittremieux/GLEAMS
- name: conda
  role: Creates the environment with compiler tools and GPU runtime from environment.yml
- name: pip
  role: Installs GLEAMS from the GitHub repository
- name: MSGF+
  role: Produced the underlying 185 million PSMs at 1% FDR from which the 30 million
    training PSMs derive
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-022-01496-1
    title: A learned embedding for efficient joint analysis of millions of mass spectra
  source_content_hash: sha256:130f5be64cdda109f76fa270bd967113627416ec3e78c5fa033097988950212e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GLEAMS
  tools:
  - GLEAMS
  - MSGF+
  - Python
  - conda
  - pip
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.84
n_claims: 25
---

# vgg-inspired-multi-branch-network-architecture

## Summary

Reconstruct the GLEAMS embedder: parallel VGG-inspired branches for precursor, fragment-peak, and reference-spectrum features, concatenated and mapped through dense layers to a 32-dimensional L2-regularized output, trained as a Siamese pair network with contrastive loss.

## When to use

Use when reproducing or adapting the GLEAMS neural architecture that embeds MS/MS spectra into a 32-dimensional space, or when training a Siamese contrastive model on PSM pairs.

## When NOT to use

- Input spectra lack reliable peptide identifications (PSMs) — the pair-construction step needs same-peptide positives from MassIVE-KB annotations
- The goal is inference/annotation with pre-trained embeddings rather than (re)training — use the released gleams_82c0124b.hdf5 weights directly instead
- No CUDA-enabled GPU or Linux environment is available, which the GLEAMS training setup requires

## Inputs

- MassIVE-KB (v1) PSM metadata TSV (30 million PSMs, columns 'filename' and 'scan') from the Human HCD Spectral Library candidate library spectra
- MS/MS spectra in mzML peak files retrieved from the MassIVE FTP server
- Feature encodings: precursor-mass features, fragment-peak features, reference-spectrum features

## Outputs

- Trained model weights in HDF5 format (e.g. gleams_82c0124b.hdf5)
- 32-dimensional spectral embeddings as an n x 32 NumPy array (GLEAMS_embed.npy) plus metadata parquet
- Per-iteration training and validation contrastive loss history, plotted as a loss-curve PNG

## How to apply

Preprocess the 30 million MassIVE-KB (v1) Human HCD Spectral Library PSMs into three feature encodings: precursor-mass features, fragment-peak features, and reference-spectrum features. Build parallel branches for each encoding, concatenate them, and map through fully-connected layers to a 32-dimensional embedding with L2 regularization (VGG-style stacked dense layers). Form balanced positive pairs (spectra from the same peptide, grouped per precursor charge state) and negative pairs (mismatched spectra), then compile the Siamese network with a contrastive loss and the rectified Adam optimizer. Train for 50 iterations on a CUDA-enabled GPU, monitoring validation contrastive loss, and save weights to HDF5 (e.g. gleams_82c0124b.hdf5).

## Related tools

- **GLEAMS** (Provides the multi-branch embedder architecture, Siamese pair training with contrastive loss, and the gleams embed/cluster commands; training configuration lives in gleams/config.py) — https://github.com/bittremieux/GLEAMS
- **conda** (Creates the environment with compiler tools and GPU runtime from environment.yml)
- **pip** (Installs GLEAMS from the GitHub repository)
- **MSGF+** (Produced the underlying 185 million PSMs at 1% FDR from which the 30 million training PSMs derive)

## Examples

```
conda env create -f https://raw.githubusercontent.com/bittremieux/GLEAMS/master/environment.yml && conda activate gleams && pip install git+https://github.com/bittremieux/GLEAMS.git
```

## Evaluation signals

- The saved weights file loads correctly as HDF5 and matches the released naming (gleams_82c0124b.hdf5)
- Validation contrastive loss decreases over the 50 iterations and converges below the training-loss plateau, as in the published loss-curve figure
- The embedder maps mzML spectra to an n x 32 NumPy array (GLEAMS_embed.npy) with a matching GLEAMS_embed.parquet metadata file
- Embedding-space sanity check: spectra generated by the same peptide cluster close together under gleams cluster with --distance_threshold 0.3
- Positive/negative pair sets are balanced and grouped per precursor charge state

## Limitations

- GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU
- Git LFS bandwidth limits can block downloading model weights during pip install; work around by GIT_LFS_SKIP_SMUDGE=1 cloning and manually fetching the weights release
- The full MassIVE-KB training set is not a single download; spectra must be retrieved per-file from the MassIVE FTP server by filename and scan number
- Training hyperparameters (iterations, layer sizes) are configurable in gleams/config.py and deviating from published settings changes convergence behavior

## Evidence

- [intro] GLEAMS encodes mass spectra as vectors of features and feeds them to a neural network to embed them into a 32-dimensional space in which spectra generated by the same peptide are close together.: "GLEAMS encodes mass spectra as vectors of features and feeds them to a neural network to embed them into a 32-dimensional space in which spectra"
- [readme] GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU.: "GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU."
- [readme] GLEAMS was trained on 30 million PSMs from the [MassIVE-KB (v1) dataset]: "GLEAMS was trained on 30 million PSMs from the [MassIVE-KB (v1) dataset]"
- [readme] Full configuration of GLEAMS, including various configurations to train the neural network, can be modified in the gleams/config.py file.: "Full configuration of GLEAMS, including various configurations to train the neural network, can be modified in the `gleams/config.py` file."
- [readme] GLEAMS provides the `gleams embed` command to convert MS/MS spectra in peak files to 32-dimensional embeddings.: [evidence span withheld — anchor status: external_doc]
