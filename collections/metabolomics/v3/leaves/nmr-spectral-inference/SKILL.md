---
name: nmr-spectral-inference
description: Use when given 1H NMR spectral data of a flavor mixture and a standards
  library, and the goal is to predict the mixture's component list and quantify identification
  accuracy (ACC/TPR/FPR).
when_to_use_negative:
- Input is a single pure compound's spectrum rather than a mixture, since FlavorFormer
  was built to identify components within flavor mixtures.
- You need to train the model from scratch or re-run full training rather than reload
  and predict — that is the separate training path also run through demo.ipynb, not
  the inference-only reproduction of unknown-mixture identification.
- Input spectra are not 1H NMR (e.g. 13C or other nuclei), which is outside the article's
  stated scope.
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: FlavorFormer
  role: Hybrid CNN + Transformer model (bi-encoder, cross-encoder, fusion pooling,
    weighted loss) that reloads parameters from model/ and predicts mixture components
    from 1H NMR spectra in demo.ipynb
  repo: https://github.com/yfWang01/FlavorFormer.git
- name: PyTorch
  role: Deep-learning framework (version 2.7.0+cu118) in which FlavorFormer is implemented
    and run for inference
- name: conda
  role: Creates the Python virtual environment required to run FlavorFormer inference
    (environment.yml)
- name: git
  role: Clones the yfWang01/FlavorFormer repository containing model parameters, data,
    and demo.ipynb
  repo: https://github.com/yfWang01/FlavorFormer.git
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.microc.2025.115372
    title: 'FlavorFormer: hybrid deep learning for identifying compounds in flavor
      mixtures based on NMR spectroscopy'
  source_content_hash: sha256:09a88fb2ad094207436d1b9d5283bcb8b6cb2ebe152ce31e723c117f943641c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yfWang01/FlavorFormer.git
  tools:
  - FlavorFormer
  - PyTorch
  - Python
  - conda
  - git
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# nmr-spectral-inference

## Summary

Run FlavorFormer inference on 1H NMR spectra of flavor mixtures using a hybrid CNN + Transformer bi-encoder and cross-encoder with fusion pooling and a weighted loss function, to identify the components of an undisclosed mixture against a standards library and score the prediction with ACC, TPR, and FPR.

## When to use

Use when given 1H NMR spectral data of a flavor mixture and a standards library, and the goal is to predict the mixture's component list and quantify identification accuracy (ACC/TPR/FPR).

## When NOT to use

- Input is a single pure compound's spectrum rather than a mixture, since FlavorFormer was built to identify components within flavor mixtures.
- You need to train the model from scratch or re-run full training rather than reload and predict — that is the separate training path also run through demo.ipynb, not the inference-only reproduction of unknown-mixture identification.
- Input spectra are not 1H NMR (e.g. 13C or other nuclei), which is outside the article's stated scope.

## Inputs

- 1H NMR spectral data of flavor mixtures (from the data/ folder of the FlavorFormer repository)
- Pretrained FlavorFormer model parameters (from the model/ folder)
- Ground-truth component list (disclosed formulation) for each mixture
- Standards library of known component spectra

## Outputs

- Predicted component list for each mixture spectrum
- Per-mixture true-positive / false-positive classification
- Summary metrics: ACC, TPR, FPR (unknown dataset targets: ACC 94.44 ± 2.41 %, TPR 93.33 ± 11.55 %, FPR 5.26 ± 0.00 %)
- prediction_metrics.csv with per-mixture predictions and summary metrics

## How to apply

Clone the yfWang01/FlavorFormer repository and create the Python environment via `conda env create -f environment.yml` (Python 3.13.2, PyTorch 2.7.0+cu118). Load the pretrained model parameters from the model/ folder and the mixture spectra plus ground-truth component lists from the data/ folder, then run inference in demo.ipynb as the article specifies, using the bi-encoder and cross-encoder with fusion pooling to capture local and global spectral features. For each mixture spectrum the model produces a predicted component list; compare it against the disclosed ground-truth formulation to classify true positives and false positives. Compute ACC, TPR, and FPR across the unknown mixture dataset and check them against the paper's reported values (ACC 94.44 ± 2.41 %, TPR 93.33 ± 11.55 %, FPR 5.26 ± 0.00 %). Write per-mixture predictions and summary metrics to prediction_metrics.csv.

## Related tools

- **FlavorFormer** (Hybrid CNN + Transformer model (bi-encoder, cross-encoder, fusion pooling, weighted loss) that reloads parameters from model/ and predicts mixture components from 1H NMR spectra in demo.ipynb) — https://github.com/yfWang01/FlavorFormer.git
- **PyTorch** (Deep-learning framework (version 2.7.0+cu118) in which FlavorFormer is implemented and run for inference)
- **conda** (Creates the Python virtual environment required to run FlavorFormer inference (environment.yml))
- **git** (Clones the yfWang01/FlavorFormer repository containing model parameters, data, and demo.ipynb) — https://github.com/yfWang01/FlavorFormer.git

## Examples

```
git clone https://github.com/yfWang01/FlavorFormer.git; cd FlavorFormer; conda env create -f environment.yml; conda activate FlavorFormer; jupyter execute demo.ipynb
```

## Evaluation signals

- Computed ACC on the unknown flavor mixture dataset must reproduce the reported ACC 94.44 ± 2.41 % within the reported standard deviation.
- Computed TPR must match TPR 93.33 ± 11.55 %, and FPR must match FPR 5.26 ± 0.00 %.
- Each predicted component list must be comparable term-by-term against the disclosed ground-truth formulation, yielding well-defined true positives and false positives.
- prediction_metrics.csv must exist and contain both per-mixture predictions and the summary ACC/TPR/FPR values.
- Model inference must run with parameters loaded from the model/ folder and spectra from the data/ folder of the cloned repository, without retraining.

## Limitations

- Performance on the unknown mixture dataset carries high variance in TPR (± 11.55 %), so per-mixture results can be unstable even though the aggregate matches the paper.
- FlavorFormer was benchmarked against pSCNN and shown to be more robust with higher TPR on complex flavor mixtures, but the skill depends on the standards library and disclosed ground truth being available for evaluation.
- The reported metrics apply to the augmented test set and the two experimental datasets as described in the paper; other spectral conditions or mixture types are not covered by this evidence.

## Evidence

- [abstract] FlavorFormer achieved on an unknown flavor mixture dataset ACC 94.44 ± 2.41 %, TPR 93.33 ± 11.55 %, FPR 5.26 ± 0.00 %.: "an unknown flavor mixture dataset (ACC 94.44 ± 2.41 %, TPR 93.33 ± 11.55 %, FPR 5.26 ± 0.00 %)"
- [methods] Model reload and prediction path via model/ and data/ folders.: "The model parameters is located in the [model](model/) folder and the required data is in the [data](data/) folder, FlavorFormer can be reload and predict easily."
- [readme] Repository clone and environment setup commands.: "git clone https://github.com/yfWang01/FlavorFormer.git"
- [readme] Environment creation via conda environment.yml.: "conda env create -f environment.yml"
- [readme] demo.ipynb is the entry point for training and predicting mixture spectra.: "For training model and predict mixture spectra data, run the file   `demo.ipynb`."
