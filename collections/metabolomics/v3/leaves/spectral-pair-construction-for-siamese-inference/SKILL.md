---
name: spectral-pair-construction-for-siamese-inference
description: Use when an unknown mixture NMR spectrum must be decomposed into candidate
  components by scoring it pairwise against a library of reference compound spectra
  with a trained pSCNN model.
when_to_use_negative:
- You only have pure-compound spectra to classify without a mixture spectrum — pSCNN
  is designed for compound identification in mixtures, not single-spectrum classification
- You are training a model from scratch rather than predicting with the pre-trained
  model — then use the data-augmentation and training workflow instead of pair-construction
  inference
- No reference spectral database is available to pair against the mixture spectrum
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3474
tools:
- name: pSCNN
  role: Pseudo-Siamese CNN that consumes each mixture-database compound spectral pair
    and outputs the inclusion probability; prediction is run via 'pSCNN.py'
  repo: https://github.com/yuxuanliao/pSCNN.git
- name: TensorFlow 2.5.0-GPU
  role: Deep-learning backend used to reload the pre-trained pSCNN model and run prediction
    on the constructed spectral pairs
- name: Python 3.8.13
  role: Runtime environment (Anaconda env 'pSCNN') in which the model is reloaded
    and pair-wise prediction executed
- name: tqdm
  role: Progress reporting while looping over the N mixture-database compound pairs
    during prediction
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/molecules27123653
    title: Deep Learning-Based Method for Compound Identification in NMR Spectra of
      Mixtures
  source_content_hash: sha256:e88d5e5f5e4690c79f1d62b775b8503c50e110632c32e51af3f3a903282f73d5
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# spectral-pair-construction-for-siamese-inference

## Summary

Constructs pairwise spectral inputs — a mixture NMR spectrum paired with each candidate database compound spectrum — so a pseudo-Siamese convolutional neural network (pSCNN) can output a compound-inclusion probability per pair, enabling compound identification in NMR spectra of mixtures despite chemical shift variation and peak overlap.

## When to use

Use when an unknown mixture NMR spectrum must be decomposed into candidate components by scoring it pairwise against a library of reference compound spectra with a trained pSCNN model.

## When NOT to use

- You only have pure-compound spectra to classify without a mixture spectrum — pSCNN is designed for compound identification in mixtures, not single-spectrum classification
- You are training a model from scratch rather than predicting with the pre-trained model — then use the data-augmentation and training workflow instead of pair-construction inference
- No reference spectral database is available to pair against the mixture spectrum

## Inputs

- Unknown mixture 1D NMR spectrum (e.g. U1 from the flavor mixtures dataset)
- Spectral database of N reference compound 1D NMR spectra
- Pre-trained pSCNN model weights from Zenodo deposit 10.5281/zenodo.6504814

## Outputs

- N inclusion probabilities, one per mixture-compound spectral pair
- Ranked candidate component list with predicted probabilities (e.g. candidate_components.csv), thresholded at 0.5

## How to apply

Download the pre-trained pSCNN model and example NMR data (mixture spectra plus database compound spectra) from Zenodo (10.5281/zenodo.6504814) or clone github.com/yuxuanliao/pSCNN, and set up Python 3.8.13 with TensorFlow 2.5.0-GPU in an Anaconda environment (pip install tqdm for progress bars). Load the unknown mixture spectrum (e.g. the U1 flavor mixture) together with the N reference compound spectra from the spectral database. Form N spectral pairs — each pairing the mixture spectrum with exactly one database compound spectrum — in the pSCNN input format, then run the prediction step via the 'pSCNN.py' entry point so the pseudo-Siamese CNN outputs an inclusion probability for each pair. Apply a probability threshold of 0.5 to flag candidate components and rank the flagged compounds by descending predicted probability. The rationale is that the Siamese architecture compares mixture-vs-reference spectra pairwise, and CNN translational invariance absorbs chemical shift variation between the same compound in different mixtures.

## Related tools

- **pSCNN** (Pseudo-Siamese CNN that consumes each mixture-database compound spectral pair and outputs the inclusion probability; prediction is run via 'pSCNN.py') — https://github.com/yuxuanliao/pSCNN.git
- **TensorFlow 2.5.0-GPU** (Deep-learning backend used to reload the pre-trained pSCNN model and run prediction on the constructed spectral pairs)
- **Python 3.8.13** (Runtime environment (Anaconda env 'pSCNN') in which the model is reloaded and pair-wise prediction executed)
- **tqdm** (Progress reporting while looping over the N mixture-database compound pairs during prediction)

## Examples

```
conda activate pSCNN; python pSCNN.py  # reload Zenodo model + U1 mixture & database spectra, predict N pair probabilities, threshold at 0.5
```

## Evaluation signals

- Each of the N mixture-compound pairs yields exactly one inclusion probability in [0, 1]
- Ranking flagged compounds (probability > 0.5) by descending probability for mixture U1 reproduces the reported order: beta-Ionone, gamma-Decalactone, gamma-Nonanoic lactone, Citral, Leaf alcohol, Isovaleric acid, 2-Methylbutyric acid
- Reported real-data performance is matched: ACC = 97.62%, TPR = 96.44%, FPR = 2.29% on the flavor mixtures dataset (and ACC = 99.80% on the augmented test set)
- The environment resolves as Python 3.8.13 with TensorFlow 2.5.0-GPU per requirements.txt, and 'pSCNN.py' runs without errors on the Zenodo example data

## Limitations

- The pre-trained model exceeded repository size limits and must be downloaded from Zenodo, adding a dependency on that deposit
- Performance degrades on additional/unseen experimental mixtures: ACC = 91.67%, TPR = 100.00%, FPR = 10.53% on the additional flavor mixture dataset
- Chemical shift variation of the same compound across mixtures and peak overlapping among molecules make identification difficult, which the CNN's translational invariance mitigates but does not guarantee

## Evidence

- [intro] We presented a pseudo Siamese convolutional neural network method (pSCNN) to solve the problems of compound identification in NMR spectra of mixtures: "We presented a pseudo Siamese convolutional neural network method (pSCNN) to solve the problems of compound identification in NMR spectra of mixtures"
- [methods] Run the file 'pSCNN.py'. The model and these data have been uploaded at Zenodo.: "Run the file 'pSCNN.py'. The model and these data have been uploaded at Zenodo."
- [methods] Download the model and these example data, pSCNN can be reload and predict easily.: "Download the model and these example data, pSCNN can be reload and predict easily."
- [abstract] the flavor mixtures dataset (ACC = 97.62%, TPR = 96.44% and FPR = 2.29%): "the flavor mixtures dataset (ACC = 97.62%, TPR = 96.44% and FPR = 2.29%)"
- [abstract] We have demonstrated that the translational invariance of convolutional neural networks can solve the chemical shift variation problem in NMR spectra.: "We have demonstrated that the translational invariance of convolutional neural networks can solve the chemical shift variation problem in NMR spectra."
