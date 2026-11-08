---
name: validation-loss-model-selection
description: Use when retraining the GLEAMS Siamese network from gleams/config.py
  and you must decide whether precursor, fragment, and reference-spectrum feature
  blocks each contribute — or when comparing trained-model variants by validation
  loss.
when_to_use_negative:
- You only need to embed or cluster new spectra with the released pretrained weights
  (gleams_82c0124b.hdf5) rather than retrain and compare model variants
- The goal is tuning clustering parameters such as --distance_threshold 0.3 for gleams
  cluster, not selecting input feature blocks by validation loss
- No CUDA-enabled GPU or Linux environment is available, since GLEAMS explicitly requires
  Python 3.8, Linux, and a CUDA GPU
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0121
tools:
- name: GLEAMS
  role: Siamese neural network whose gleams/config.py feature blocks are toggled to
    produce each ablation variant evaluated by validation loss
  repo: https://github.com/bittremieux/GLEAMS
- name: conda
  role: Creates the 'gleams' environment with compiler tools and GPU runtime needed
    to train the network
- name: pip
  role: Installs GLEAMS from the GitHub repository into the training environment
  repo: https://github.com/bittremieux/GLEAMS.git
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-022-01496-1
    title: A learned embedding for efficient joint analysis of millions of mass spectra
  source_content_hash: sha256:97f2af95ebe54a77aac50ac3805b9fc5fda115797ca1be7bb769cef0d03571e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GLEAMS
  tools:
  - GLEAMS
  - conda
  - pip
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# validation-loss-model-selection

## Summary

Select among GLEAMS encoder configurations (all-features baseline vs. feature-type ablations) by comparing validation loss after convergence, using the 32-dimensional Siamese embedding network trained on MassIVE-KB (v1) Human HCD PSMs.

## When to use

Use when retraining the GLEAMS Siamese network from gleams/config.py and you must decide whether precursor, fragment, and reference-spectrum feature blocks each contribute — or when comparing trained-model variants by validation loss.

## When NOT to use

- You only need to embed or cluster new spectra with the released pretrained weights (gleams_82c0124b.hdf5) rather than retrain and compare model variants
- The goal is tuning clustering parameters such as --distance_threshold 0.3 for gleams cluster, not selecting input feature blocks by validation loss
- No CUDA-enabled GPU or Linux environment is available, since GLEAMS explicitly requires Python 3.8, Linux, and a CUDA GPU

## Inputs

- MassIVE-KB (v1) Human HCD spectral library candidate library spectra metadata TSV (30 million PSMs, task 82c0124b)
- Corresponding peak files retrieved from the MassIVE FTP server, indexed by 'filename' and 'scan' columns
- gleams/config.py specifying the feature blocks (precursor, fragment, reference-spectrum) for each ablation run

## Outputs

- Trained GLEAMS Siamese network weights per configuration (all-features, no-reference, no-fragment, no-precursor)
- Validation loss after convergence for each configuration
- ablation_table.csv collating validation loss across the four feature-type configurations

## How to apply

Train the baseline GLEAMS network with all three feature types by editing gleams/config.py inside the Conda 'gleams' environment (Python 3.8, Linux, CUDA-enabled GPU) on the ~30 million PSMs compiled from the MassIVE-KB (v1) Human HCD spectral library (maximum top 100 PSMs per unique precursor). Record the validation loss after convergence, then retrain the identical architecture three more times with the reference-spectrum, fragment, and precursor feature blocks withheld in turn. The decision rule is comparative validation loss: the paper's ablation shows that removing either the reference-spectrum or fragment features increases validation loss, so the full-features model should win; any ordering that deviates signals a training or data-assembly error. Collate all four runs into ablation_table.csv for inspection against Extended Data Fig. 9.

## Related tools

- **GLEAMS** (Siamese neural network whose gleams/config.py feature blocks are toggled to produce each ablation variant evaluated by validation loss) — https://github.com/bittremieux/GLEAMS
- **conda** (Creates the 'gleams' environment with compiler tools and GPU runtime needed to train the network)
- **pip** (Installs GLEAMS from the GitHub repository into the training environment) — https://github.com/bittremieux/GLEAMS.git

## Examples

```
conda env create -f https://raw.githubusercontent.com/bittremieux/GLEAMS/master/environment.yml && conda activate gleams
```

## Evaluation signals

- Validation loss for the all-features baseline is lower than for both the no-reference-spectrum and no-fragment ablations, matching Extended Data Fig. 9
- ablation_table.csv contains a converged validation-loss entry for all four configurations (baseline plus three held-out feature blocks)
- Each ablation retraining uses an otherwise identical gleams/config.py setup and the same 30-million-PSM training set, so losses are comparable
- Training runs to convergence on a CUDA-enabled GPU without NaN/divergence before losses are compared

## Limitations

- GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU; training cannot proceed on unsupported platforms
- The 30-million-PSM training set is not available as a single download — spectra must be reassembled from the metadata TSV and MassIVE FTP server by filename and scan number
- Git LFS data-quota errors can block installation of model weights; the weights file must be downloaded manually from the v0.3 release if cloning fails

## Evidence

- [other] Ablation tests confirm that removing either the reference-spectrum features or the fragment features leads to decreased GLEAMS performance: [evidence span withheld — anchor status: not_found]
- [readme] Full configuration of GLEAMS, including various configurations to train the neural network, can be modified in the gleams/config.py file.: "Full configuration of GLEAMS, including various configurations to train the neural network, can be modified in the `gleams/config.py` file."
- [intro] GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU.: "GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU."
- [intro] GLEAMS encodes mass spectra as vectors of features and feeds them to a neural network to embed them into a 32-dimensional space: "GLEAMS encodes mass spectra as vectors of features and feeds them to a neural network to embed them into a 32-dimensional space"
- [readme] GLEAMS was trained on 30 million PSMs from the [MassIVE-KB (v1) dataset]: "GLEAMS was trained on 30 million PSMs from the [MassIVE-KB (v1) dataset]"
