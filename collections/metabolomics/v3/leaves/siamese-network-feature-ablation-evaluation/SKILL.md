---
name: siamese-network-feature-ablation-evaluation
description: Use when a trained Siamese multi-input encoder's feature blocks must
  be ablated to test whether precursor, fragment, and reference-spectrum features
  each contribute complementary information.
when_to_use_negative:
- The goal is inference-only use of the pretrained 32-dimensional embedding (gleams
  embed / gleams cluster) rather than retraining — no ablation training runs are needed.
- Only a single feature configuration is available with no baseline comparison, so
  degradation relative to the all-features model cannot be measured.
- No CUDA-enabled GPU is available and only CPU-scale compute exists — the README
  states GLEAMS requires a CUDA-enabled GPU.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: GLEAMS
  role: Siamese network encoder whose precursor, fragment, and reference-spectrum
    feature blocks are ablated and retrained; configuration edited in gleams/config.py
  repo: https://github.com/bittremieux/GLEAMS
- name: conda
  role: Create the environment with compiler tools and GPU runtime for training runs
- name: pip
  role: Install GLEAMS from the GitHub repository into the Conda environment
  repo: https://github.com/bittremieux/GLEAMS.git
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-022-01496-1
    title: A learned embedding for efficient joint analysis of millions of mass spectra
  source_content_hash: sha256:a2bb341b4c29e5c509462ef0c199746bf1ec10d09a43a86bcb69113539a9f316
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GLEAMS
  tools:
  - GLEAMS
  - conda
  - pip
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# siamese-network-feature-ablation-evaluation

## Summary

Retrain the GLEAMS Siamese neural network with individual feature blocks (precursor, fragment, reference-spectrum) withheld to quantify each feature type's contribution via validation loss, reproducing the Extended Data Fig. 9 ablation.

## When to use

Use when a trained Siamese multi-input encoder's feature blocks must be ablated to test whether precursor, fragment, and reference-spectrum features each contribute complementary information.

## When NOT to use

- The goal is inference-only use of the pretrained 32-dimensional embedding (gleams embed / gleams cluster) rather than retraining — no ablation training runs are needed.
- Only a single feature configuration is available with no baseline comparison, so degradation relative to the all-features model cannot be measured.
- No CUDA-enabled GPU is available and only CPU-scale compute exists — the README states GLEAMS requires a CUDA-enabled GPU.

## Inputs

- MassIVE-KB (v1) Human HCD candidate library spectra metadata TSV (30 million PSMs, columns 'filename' and 'scan')
- Corresponding MS/MS spectra retrieved from the MassIVE FTP server
- gleams/config.py from the GLEAMS repository, edited per ablation configuration
- CUDA-enabled GPU environment (Python 3.8, Linux)

## Outputs

- ablation_table.csv with validation loss for all-features baseline and each held-out configuration
- Trained Siamese network model weights per ablation run

## How to apply

Compile the training corpus from the MassIVE-KB (v1) Human HCD spectral library: download the candidate library spectra metadata TSV (30 million PSMs, maximum top 100 PSMs per unique precursor from 2.1 million precursors) and retrieve the corresponding spectra from the MassIVE FTP server by 'filename' and 'scan' columns. Set up a Conda environment (Python 3.8, Linux, CUDA GPU) and install GLEAMS from the repository. Train the baseline Siamese network with all three feature types by editing gleams/config.py, then retrain three times withholding one feature block at a time (no-reference, no-fragment, no-precursor), recording the validation loss after convergence each run. Collate the validation losses for the all-features baseline and each held-out configuration into ablation_table.csv; correct application yields increased validation loss when either the reference-spectrum or fragment features are removed, matching the Extended Data Fig. 9 ordering.

## Related tools

- **GLEAMS** (Siamese network encoder whose precursor, fragment, and reference-spectrum feature blocks are ablated and retrained; configuration edited in gleams/config.py) — https://github.com/bittremieux/GLEAMS
- **conda** (Create the environment with compiler tools and GPU runtime for training runs)
- **pip** (Install GLEAMS from the GitHub repository into the Conda environment) — https://github.com/bittremieux/GLEAMS.git

## Examples

```
conda env create -f https://raw.githubusercontent.com/bittremieux/GLEAMS/master/environment.yml && conda activate gleams && pip install git+https://github.com/bittremieux/GLEAMS.git
```

## Evaluation signals

- Validation loss increases (performance degrades) when the reference-spectrum feature block is withheld, reproducing the Extended Data Fig. 9 ordering.
- Validation loss increases when the fragment feature block is withheld.
- ablation_table.csv contains validation loss rows for the all-features baseline plus all three held-out configurations.
- Training set matches the paper scale: 30 million PSMs from MassIVE-KB (v1), maximum top 100 PSMs per unique precursor.

## Limitations

- The article reports that removing either the reference-spectrum or fragment features degrades validation performance; the degradation on removing precursor features alone may be subtle to detect.
- Training requires the full 30-million-PSM corpus, which is not available as a single download and must be assembled from per-task MassIVE mzTab / FTP retrieval.
- Git LFS bandwidth limits may block cloning; the model weights (gleams_82c0124b.hdf5) must then be downloaded manually from the v0.3 release and placed in data/.

## Evidence

- [supplementary] Ablation tests confirm that removing either the reference-spectrum features or the fragment features leads to decreased GLEAMS performance: [evidence span withheld — anchor status: not_found]
- [readme] GLEAMS was trained on 30 million PSMs from the MassIVE-KB (v1) dataset: "GLEAMS was trained on 30 million PSMs from the [MassIVE-KB (v1) dataset](https://massive.ucsd.edu/ProteoSAFe/static/massive-kb-libraries.jsp)."
- [readme] These 30 million PSMs are a subset of all spectrum identifications: "These 30 million PSMs are a subset of all spectrum identifications that were obtained during compilation of the MassIVE-KB resource (maximum top 100 PSMs for 2.1 million unique precursors)."
- [readme] Full configuration of GLEAMS, including various configurations to train the neural network: "Full configuration of GLEAMS, including various configurations to train the neural network, can be modified in the `gleams/config.py` file."
- [intro] GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU: "GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU."
