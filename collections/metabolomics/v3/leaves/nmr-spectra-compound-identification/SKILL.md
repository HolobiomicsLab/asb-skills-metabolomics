---
name: nmr-spectra-compound-identification
description: Use when you need to identify known compounds in flavor/food mixture
  samples from 1H NMR spectra rather than run full spectral deconvolution, or when
  you want to test whether a trained classifier generalizes to unseen compounds by
  retraining on a reduced standard set.
when_to_use_negative:
- When the goal is to quantify concentrations or deconvolve overlapping peaks rather
  than classify which known compounds are present in the spectrum.
- When the compound of interest is outside the closed library of 24 flavor standards
  and no retraining with that standard added is possible.
- When you need general mixture analysis from NMR without a labeled set of reference
  standards to train on — the model must be trained on spectra of known standards
  first.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3391
tools:
- name: FlavorFormer
  role: The hybrid CNN–Transformer model that is trained and used to identify compounds
    in flavor mixtures from 1H NMR spectra; run via demo.ipynb, with weights in model/
    and data in data/.
  repo: https://github.com/yfWang01/FlavorFormer.git
- name: conda
  role: Creates the project virtual environment from environment.yml so the required
    Python and PyTorch versions are reproducibly installed.
  repo: https://github.com/yfWang01/FlavorFormer.git
- name: PyTorch 2.7.0+cu118
  role: Deep learning framework in which the bi-encoder, cross-encoder, fusion pooling
    and weighted loss function of FlavorFormer are implemented and trained.
  repo: https://github.com/yfWang01/FlavorFormer.git
- name: Anaconda
  role: Distribution used to install conda and manage the Python 3.13.2 environment
    for the pipeline.
- name: git
  role: Used to clone the FlavorFormer repository containing code, model parameters
    and data.
  repo: https://github.com/yfWang01/FlavorFormer.git
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.microc.2025.115372
    title: 'FlavorFormer: hybrid deep learning for identifying compounds in flavor
      mixtures based on NMR spectroscopy'
  source_content_hash: sha256:ebad97ee41a11ebbcb0cc1ef57aa49841f198dda370164cfda70e919cece0ed6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yfWang01/FlavorFormer.git
  tools:
  - Anaconda
  - FlavorFormer
  - PyTorch 2.7.0+cu118
  - Python
  - conda
  - git
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 24
---

# nmr-spectra-compound-identification

## Summary

Train and apply FlavorFormer, a hybrid CNN–Transformer model that identifies which of 24 known flavor standard compounds are present in 1H NMR spectra of mixtures using a bi-encoder plus cross-encoder architecture with fusion pooling and a weighted loss function.

## When to use

Use when you need to identify known compounds in flavor/food mixture samples from 1H NMR spectra rather than run full spectral deconvolution, or when you want to test whether a trained classifier generalizes to unseen compounds by retraining on a reduced standard set.

## When NOT to use

- When the goal is to quantify concentrations or deconvolve overlapping peaks rather than classify which known compounds are present in the spectrum.
- When the compound of interest is outside the closed library of 24 flavor standards and no retraining with that standard added is possible.
- When you need general mixture analysis from NMR without a labeled set of reference standards to train on — the model must be trained on spectra of known standards first.

## Inputs

- 1H NMR spectra of flavor standard compounds (training data in the repo data/ folder)
- Augmented 1H NMR training datasets built from the standards
- 1H NMR spectra of experimental flavor mixture samples (known-mixture and unknown-mixture datasets)

## Outputs

- Trained FlavorFormer model parameter files stored in the model/ folder
- Predicted set of compounds present for each input mixture spectrum
- Classification metrics ACC, TPR and FPR (mean ± standard deviation over repeated runs) for the augmented test set and mixture datasets

## How to apply

Set up the environment by cloning the yfWang01/FlavorFormer repository and creating a conda environment from environment.yml (Python 3.13.2, PyTorch 2.7.0+cu118). Load the training data from the data/ folder, apply spectral augmentation, and train the hybrid CNN–Transformer model from demo.ipynb: a bi-encoder produces local features and global dependencies from each 1H NMR spectrum, a cross-encoder fuses pair inputs, and a fusion pooling strategy plus weighted loss function drive identification of compounds in flavor mixtures. Evaluate the trained model on an augmented test set and on experimental flavor mixture datasets, reporting ACC, TPR and FPR as mean ± standard deviation over repeated runs; the reference implementation reports test-set ACC 98.84 ± 0.86 %, TPR 99.90 ± 0.10 % and FPR 2.24 ± 1.77 %. To test generalization or expand the recognition range, remove one or more standards (e.g. Lactic acid, Isopentyl acetate, γ-Nonanoic lactone) from the training data, retrain without altering the architecture, and check the retrained model still meets near-baseline metrics (ACC ≈ 99.27 %, TPR ≈ 99.87 % on the augmented test set; TPR ≈ 94.92 % on known mixtures). Save trained weights in the model/ folder; pretrained parameters there allow FlavorFormer to be reloaded directly for predicting mixture spectra.

## Related tools

- **FlavorFormer** (The hybrid CNN–Transformer model that is trained and used to identify compounds in flavor mixtures from 1H NMR spectra; run via demo.ipynb, with weights in model/ and data in data/.) — https://github.com/yfWang01/FlavorFormer.git
- **conda** (Creates the project virtual environment from environment.yml so the required Python and PyTorch versions are reproducibly installed.) — https://github.com/yfWang01/FlavorFormer.git
- **PyTorch 2.7.0+cu118** (Deep learning framework in which the bi-encoder, cross-encoder, fusion pooling and weighted loss function of FlavorFormer are implemented and trained.) — https://github.com/yfWang01/FlavorFormer.git
- **Anaconda** (Distribution used to install conda and manage the Python 3.13.2 environment for the pipeline.)
- **git** (Used to clone the FlavorFormer repository containing code, model parameters and data.) — https://github.com/yfWang01/FlavorFormer.git

## Examples

```
git clone https://github.com/yfWang01/FlavorFormer.git; cd FlavorFormer; conda env create -f environment.yml; conda activate FlavorFormer; jupyter notebook demo.ipynb
```

## Evaluation signals

- On the augmented test set, reported performance should match ACC 98.84 ± 0.86 %, TPR 99.90 ± 0.10 %, FPR 2.24 ± 1.77 %.
- On the known flavor mixture dataset, ACC ≈ 95.44 ± 0.00 %, TPR ≈ 95.19 ± 1.24 %, FPR ≈ 4.31 ± 0.33 %.
- On the unknown flavor mixture dataset, ACC ≈ 94.44 ± 2.41 %, TPR ≈ 93.33 ± 11.55 %, FPR ≈ 5.26 ± 0.00 %.
- After retraining on 21 of 24 standards (Lactic acid, Isopentyl acetate, γ-Nonanoic lactone removed), test-set metrics should stay near baseline: ACC ≈ 99.27 ± 0.33 % and TPR ≈ 99.87 ± 0.11 %; known-mixture TPR ≈ 94.92 ± 0.55 %.
- Model weights are written to the model/ folder and can be reloaded with the data/ folder to reproduce predictions via demo.ipynb.

## Limitations

- Identification is confined to the compounds represented in the training standards (24 flavor standards in the original model); compounds unseen during training require retraining on a reduced-plus-new standard set rather than zero-shot prediction.
- Performance varies across mixture datasets: TPR on the unknown flavor mixture dataset has a large spread (93.33 ± 11.55 %), so results on genuinely novel mixtures are less stable.
- FlavorFormer was benchmarked against pSCNN for flavor mixtures; behavior on non-flavor or non-1H-NMR spectra is not established by this paper.
- The specific PyTorch (2.7.0+cu118) and Python (3.13.2) versions are required, so other toolchains may not reproduce results.

## Evidence

- [abstract] Hybrid architecture identifies compounds in flavor mixtures from 1H NMR spectra: "incorporating a hybrid CNN and Transformer architecture to capture both local features and global dependencies from 1H NMR spectra"
- [abstract] Reported test-set performance metrics for FlavorFormer: "The model achieved consistently high performance metrics on the test set (ACC 98.84 ± 0.86 %, TPR 99.90 ± 0.10 %, FPR 2.24 ± 1.77 %)"
- [abstract] Training and evaluation protocol: "After training on augmented datasets, FlavorFormer was tested on an augmented test set and two experimental datasets."
- [readme] Environment setup via conda: "Use this command to create python virtual environment.
```shell
cd FlavorFormer
conda env create -f environment.yml
conda activate FlavorFormer
```"
- [readme] Running training and prediction via demo notebook: "For training model and predict mixture spectra data, run the file   `demo.ipynb`."
- [readme] Reloading pretrained model for prediction: "The model parameters is located in the [model](model/) folder and the required data is in the [data](data/) folder, FlavorFormer can be reload and predict easily."
- [results] Retrained model on 21 standards keeps near-baseline performance: [evidence span withheld — anchor status: not_found]
- [results] Three compounds removed for the retraining experiment: [evidence span withheld — anchor status: not_found]
