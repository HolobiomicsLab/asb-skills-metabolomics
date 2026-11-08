---
name: nmr-mixture-spectra-cnn-training
description: Use when the goal is compound identification in NMR spectra of mixtures
  and you have (or can generate) superposed, noise-augmented NMR spectral pairs to
  train a CNN classifier.
when_to_use_negative:
- Input is a single, pure-compound NMR spectrum rather than mixture spectra — the
  method targets compound identification in mixtures.
- No spectral database is available to generate the superposed augmented training
  pairs required for training.
- The environment cannot provide TensorFlow 2.5.0-GPU / Python 3.8.13 and the model
  must be trained from scratch without GPU support.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3474
tools:
- name: pSCNN
  role: Pseudo Siamese CNN model trained via pSCNN.py to classify compounds in NMR
    mixture spectra
  repo: https://github.com/yuxuanliao/pSCNN.git
- name: TensorFlow 2.5.0-GPU
  role: Deep learning framework used to instantiate and train pSCNN variants
- name: Anaconda
  role: Environment manager to create the pinned Python 3.8.13 environment from requirements.txt
  repo: https://www.anaconda.com/
- name: tqdm
  role: Progress monitoring during pSCNN.py training runs
- name: Python 3.8.13
  role: Runtime interpreter for the training script
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/molecules27123653
    title: Deep Learning-Based Method for Compound Identification in NMR Spectra of
      Mixtures
  source_content_hash: sha256:44a6b16d1840e871269fa8eb27aff58403f34bec33a5dc59434881bf4db3b9c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yuxuanliao/pSCNN.git
  tools:
  - Anaconda
  - Python
  - Python 3.8.13
  - TensorFlow 2.5.0-GPU
  - pSCNN
  - tqdm
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# nmr-mixture-spectra-cnn-training

## Summary

Train the pSCNN (pseudo Siamese convolutional neural network) on augmented NMR mixture spectra to identify compounds despite chemical shift variation and peak overlap, using the article's pinned Python 3.8.13 / TensorFlow 2.5.0-GPU stack.

## When to use

Use when the goal is compound identification in NMR spectra of mixtures and you have (or can generate) superposed, noise-augmented NMR spectral pairs to train a CNN classifier.

## When NOT to use

- Input is a single, pure-compound NMR spectrum rather than mixture spectra — the method targets compound identification in mixtures.
- No spectral database is available to generate the superposed augmented training pairs required for training.
- The environment cannot provide TensorFlow 2.5.0-GPU / Python 3.8.13 and the model must be trained from scratch without GPU support.

## Inputs

- Augmented NMR mixture spectra pairs (e.g., 18,000 training / 2,000 validation superposed spectra with random noises)
- Spectral database of individual compound NMR spectra used for data augmentation
- Zenodo archive (10.5281/zenodo.6504814) containing the pSCNN model and example NMR data
- pSCNN.py training script from github.com/yuxuanliao/pSCNN

## Outputs

- Trained pSCNN model checkpoint (e.g., pscnn_model_M3.h5)
- Per-epoch validation accuracy logs / hyperparameter grid results table
- Compound identification predictions on mixture spectra (with ACC, TPR, FPR metrics)
- Experimental benchmark results on flavor mixtures datasets

## How to apply

Set up an Anaconda environment with Python 3.8.13 and TensorFlow 2.5.0-GPU, installing the packages in requirements.txt plus tqdm. Clone github.com/yuxuanliao/pSCNN and download the model and example NMR data from Zenodo (DOI 10.5281/zenodo.6504814). Build the augmented dataset by superposing NMR spectra sampled from a spectral database with random noises, split it into training, validation and test sets (the released data comprise 18,000 training and 2,000 validation mixture pairs), then run 'pSCNN.py' to train, monitoring progress with tqdm. Sweep the reported hyperparameter grid — learning rates 10^-2 to 10^-5 crossed with 5–10 convolutional layers at 100 epochs — and select the top configuration (model M3: learning rate 10^-4, 6 convolutional layers, validation accuracy 0.9990). Validate by checking test-set performance (ACC = 99.80%, TPR = 99.70%, FPR = 0.10%) and by benchmarking on experimental flavor mixture datasets.

## Related tools

- **pSCNN** (Pseudo Siamese CNN model trained via pSCNN.py to classify compounds in NMR mixture spectra) — https://github.com/yuxuanliao/pSCNN.git
- **TensorFlow 2.5.0-GPU** (Deep learning framework used to instantiate and train pSCNN variants)
- **Anaconda** (Environment manager to create the pinned Python 3.8.13 environment from requirements.txt) — https://www.anaconda.com/
- **tqdm** (Progress monitoring during pSCNN.py training runs)
- **Python 3.8.13** (Runtime interpreter for the training script)

## Examples

```
conda create --name pSCNN python=3.8.13; conda activate pSCNN; python -m pip install -r requirements.txt; pip install tqdm; git clone https://github.com/yuxuanliao/pSCNN.git; python pSCNN.py
```

## Evaluation signals

- Selected hyperparameters match epoch = 100, learning rate = 10^-4, 6 convolutional layers (model M3).
- Final validation accuracy of the selected model reaches 0.9990.
- Augmented test set performance reproduces ACC = 99.80%, TPR = 99.70%, FPR = 0.10%.
- Experimental benchmark on flavor mixtures dataset yields ACC = 97.62%, TPR = 96.44%, FPR = 2.29%.
- Training logs show tqdm-monitored per-epoch validation accuracy converging as in the paper's Figure 4b.

## Limitations

- Model size exceeded repository limits, so the trained model and all NMR data must be fetched from Zenodo separately.
- Chemical shift variation and peak overlapping among molecules make identification difficult — the augmentation strategy is designed specifically to address these, so performance on unaugmented or out-of-domain spectra is untested here.
- Benchmark on real experimental data degrades relative to augmented data (e.g., additional flavor mixture dataset: ACC = 91.67%, FPR = 10.53%).
- No discussion/limitations content was extractable for this article, so further reproducibility gaps are undocumented.

## Evidence

- [intro] pinned environment: "Python 3.8.13 and TensorFlow (version 2.5.0-GPU)"
- [abstract] data augmentation: "A data augmentation method was implemented for the superposition of several NMR spectra sampled from a spectral database with random noises."
- [methods] training script: "Run the file 'pSCNN.py'. The model and these data have been uploaded at Zenodo."
- [methods] Zenodo archive: "zenodo.org/badge/DOI/10.5281/zenodo.6504814.svg)](https://doi.org/10.5281/zenodo.6504814)"
- [abstract] rationale: "We have demonstrated that the translational invariance of convolutional neural networks can solve the chemical shift variation problem in NMR spectra."
- [readme] installation: "conda create --name pSCNN python=3.8.13
conda activate pSCNN
python -m pip install -r requirements.txt
pip install tqdm"
- [readme] Zenodo note: "Since the model exceeded the limit, we have uploaded the model and all the NMR data to Zenodo."
