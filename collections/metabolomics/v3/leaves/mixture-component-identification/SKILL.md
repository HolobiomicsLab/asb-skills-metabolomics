---
name: mixture-component-identification
description: Use when given one or more 1H NMR spectra of flavor mixtures (known or
  undisclosed compositions) and a pretrained FlavorFormer model, to produce per-spectrum
  component predictions and compute ACC, TPR, and FPR against a ground-truth formulation
  or standards library.
when_to_use_negative:
- The input consists of spectra of pure, single compounds rather than multi-component
  flavor mixtures — FlavorFormer's fusion pooling and weighted loss are designed for
  mixture identification, and the reported TPR/FPR gains are specific to complex mixtures.
- You need novel model training from scratch rather than reload-and-predict — use
  the training path of demo.ipynb instead of treating this as an inference-only skill.
- The spectral data are not 1H NMR (e.g. 13C, MS, or IR) — the paper's model, metrics,
  and reported accuracies are specific to 1H NMR spectra of flavor mixtures.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3474
tools:
- name: FlavorFormer
  role: The hybrid CNN and Transformer model (bi-encoder + cross-encoder, fusion pooling,
    weighted loss) that performs the actual component identification from 1H NMR spectra.
  repo: https://github.com/yfWang01/FlavorFormer.git
- name: PyTorch
  role: Framework in which FlavorFormer (version 2.7.0+cu118) is implemented and run
    for inference on mixture spectra.
- name: conda
  role: Creates and activates the reproducible Python environment (environment.yml)
    required to run FlavorFormer's demo.ipynb.
- name: git
  role: Clones the FlavorFormer repository containing model parameters, data, and
    demo notebook.
  repo: https://github.com/yfWang01/FlavorFormer.git
- name: Anaconda
  role: Platform providing conda for the virtual environment setup.
  repo: https://www.anaconda.com/
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1016/j.microc.2025.115372
    title: 'FlavorFormer: hybrid deep learning for identifying compounds in flavor
      mixtures based on NMR spectroscopy'
  source_content_hash: sha256:62edd620c111225b3210328e7de5be1507bbbf99926d064f74b78f793c7f2e82
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yfWang01/FlavorFormer.git
  tools:
  - Anaconda
  - FlavorFormer
  - PyTorch
  - Python
  - conda
  - git
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# mixture-component-identification

## Summary

Identify the chemical components of a flavor mixture from its 1H NMR spectrum using FlavorFormer, a hybrid CNN and Transformer model that combines a bi-encoder and cross-encoder with fusion pooling and a weighted loss function to predict the compound list.

## When to use

Use when given one or more 1H NMR spectra of flavor mixtures (known or undisclosed compositions) and a pretrained FlavorFormer model, to produce per-spectrum component predictions and compute ACC, TPR, and FPR against a ground-truth formulation or standards library.

## When NOT to use

- The input consists of spectra of pure, single compounds rather than multi-component flavor mixtures — FlavorFormer's fusion pooling and weighted loss are designed for mixture identification, and the reported TPR/FPR gains are specific to complex mixtures.
- You need novel model training from scratch rather than reload-and-predict — use the training path of demo.ipynb instead of treating this as an inference-only skill.
- The spectral data are not 1H NMR (e.g. 13C, MS, or IR) — the paper's model, metrics, and reported accuracies are specific to 1H NMR spectra of flavor mixtures.

## Inputs

- 1H NMR spectra of flavor mixtures (from the data/ folder of the FlavorFormer repo)
- Ground-truth component list / actual formulation for each mixture (or a standards library of individual compound spectra)
- Pretrained FlavorFormer model parameters (model/ folder)

## Outputs

- Predicted component list for each mixture spectrum
- Per-spectrum true-positive and false-positive classifications against the actual formulation
- Aggregated metrics: ACC, TPR, FPR (e.g. prediction_metrics.csv) and, when benchmarking, mixture-level MA and M-TPR per component-count level (reproduction_metrics.csv)

## How to apply

Clone yfWang01/FlavorFormer and set up the Python 3.13.2 environment with PyTorch 2.7.0+cu118 via `conda env create -f environment.yml; conda activate FlavorFormer`. Load the pretrained model parameters from the model/ folder and the mixture 1H NMR spectra from the data/ folder, then run the inference portion of demo.ipynb, which uses the bi-encoder and cross-encoder with fusion pooling to capture both local features and global dependencies from the spectra. For each mixture spectrum, FlavorFormer outputs a predicted component list; compare it with the actual formulation (ground-truth component list) to classify true and false positives. Aggregate per-mixture results into accuracy (ACC), true positive rate (TPR), and false positive rate (FPR); on the unknown mixture dataset the expected values are ACC 94.44 ± 2.41 %, TPR 93.33 ± 11.55 %, FPR 5.26 ± 0.00 %, while the known mixture dataset yields ACC 95.44 ± 0.00 %, TPR 95.19 ± 1.24 %, FPR 4.31 ± 0.33 %. When comparing models at different mixture complexities, also compute mixture-level MA and M-TPR per component-count level (2, 3, 4, 5 components), where FlavorFormer reaches average MA 46.03 % and M-TPR 82.54 % (100 % M-TPPR for 2-component mixtures, 46.67 % for 5-component mixtures).

## Related tools

- **FlavorFormer** (The hybrid CNN and Transformer model (bi-encoder + cross-encoder, fusion pooling, weighted loss) that performs the actual component identification from 1H NMR spectra.) — https://github.com/yfWang01/FlavorFormer.git
- **PyTorch** (Framework in which FlavorFormer (version 2.7.0+cu118) is implemented and run for inference on mixture spectra.)
- **conda** (Creates and activates the reproducible Python environment (environment.yml) required to run FlavorFormer's demo.ipynb.)
- **git** (Clones the FlavorFormer repository containing model parameters, data, and demo notebook.) — https://github.com/yfWang01/FlavorFormer.git
- **Anaconda** (Platform providing conda for the virtual environment setup.) — https://www.anaconda.com/

## Examples

```
git clone https://github.com/yfWang01/FlavorFormer.git; cd FlavorFormer; conda env create -f environment.yml; conda activate FlavorFormer; # then run the inference cells of demo.ipynb with model/ and data/
```

## Evaluation signals

- Computed unknown-mixture metrics match the reported values within their standard deviations: ACC 94.44 ± 2.41 %, TPR 93.33 ± 11.55 %, FPR 5.26 ± 0.00 %.
- Computed known-mixture metrics match: ACC 95.44 ± 0.00 %, TPR 95.19 ± 1.24 %, FPR 4.31 ± 0.33 %.
- Mixture-level benchmark on the known dataset: average MA ≈ 46.03 % and average M-TPR ≈ 82.54 % for FlavorFormer, beating pSCNN (MA 28.57 %, M-PTPR 47.62 %) and NMRformer (MA 4.76 %, M-PTPR 14.29 %).
- Per-component-count pattern reproduced: M-TPR = 100 % for 2-component mixtures and 46.67 % for 5-component mixtures (vs. pSCNN 0 % at 5 components).
- Per-mixture predictions and summary metrics are written to the agreed CSV (e.g. prediction_metrics.csv) with each predicted component list traceable to its input spectrum.

## Limitations

- Reported performance is specific to the paper's augmented test set and two experimental (known and unknown flavor mixture) datasets; other mixture types or non-flavor compounds may not transfer.
- Identification quality depends on the standards library / ground-truth formulation used for comparison; TPR/FPR values are only meaningful against a disclosed component list.
- FPR on the unknown dataset has a reported standard deviation of 0.00 % (5.26 ± 0.00 %), suggesting the metric is not well-dispersed and should be interpreted cautiously.

## Evidence

- [methods] identify the components of an undisclosed flavor mixture from its 1H NMR spectrum: [evidence span withheld — anchor status: not_found]
- [abstract] ACC 94.44 ± 2.41 %, TPR 93.33 ± 11.55 %, FPR 5.26 ± 0.00 %: "an unknown flavor mixture dataset (ACC 94.44 ± 2.41 %, TPR 93.33 ± 11.55 %, FPR 5.26 ± 0.00 %)"
- [intro] hybrid CNN and Transformer architecture: "Here, FlavorFormer is proposed by incorporating a hybrid CNN and Transformer architecture to capture both local features and global dependencies from 1H NMR spectra."
- [intro] bi-encoder and cross-encoder, a fusion pooling strategy, and a weighted loss function: "it leverages a combination of a bi-encoder and cross-encoder, a fusion pooling strategy, and a weighted loss function to identify compounds correctly."
- [readme] The model parameters is located in the model folder and the required data is in the data folder: "The model parameters is located in the [model](model/) folder and the required data is in the [data](data/) folder, FlavorFormer can be reload and predict easily."
- [methods] conda env create -f environment.yml: "Use this command to create python virtual environment.
```shell
cd FlavorFormer
conda env create -f environment.yml
conda activate FlavorFormer
```"
