---
name: probability-thresholding-and-component-ranking
description: Use when a trained pSCNN model has produced inclusion probabilities for
  N mixture–compound spectral pairs and you must convert these probabilities into
  a thresholded, descending-rank list of candidate components of an NMR mixture.
when_to_use_negative:
- You have raw NMR spectra but no trained pSCNN model or database compound spectra
  to form the mixture–compound spectral pairs — training and data augmentation come
  first
- The task is training or validating the pSCNN model itself (e.g., splitting the augmented
  dataset into training, validation and test sets) rather than predicting components
  of an unknown mixture
- Compounds in the mixture are not represented in the spectral database, since identification
  is restricted to database compounds scored by the model
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3474
tools:
- name: pSCNN
  role: Pseudo-Siamese CNN that outputs an inclusion probability for each mixture–database
    compound spectral pair; run via 'pSCNN.py' for prediction
  repo: https://github.com/yuxuanliao/pSCNN.git
- name: TensorFlow 2.5.0-GPU
  role: Deep learning framework in which the pre-trained pSCNN model is reloaded and
    the prediction step is executed
- name: Python 3.8.13
  role: Runtime environment (installed via conda environment 'pSCNN') for executing
    the prediction and ranking workflow
- name: tqdm
  role: Progress-bar dependency installed with pip for the pSCNN package requirements
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/molecules27123653
    title: Deep Learning-Based Method for Compound Identification in NMR Spectra of
      Mixtures
  source_content_hash: sha256:5775762c3be7d042812238ea3f3973a008446a4e5eec98df268e28fc251885f0
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
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# probability-thresholding-and-component-ranking

## Summary

Ranks candidate mixture components by thresholding the per-pair inclusion probabilities that the pSCNN pseudo-Siamese CNN outputs for each mixture–database compound spectral pair, converting soft predictions into a ranked compound list (e.g., the U1 mixture's β-Ionone → 2-Methylbutyric acid ordering).

## When to use

Use when a trained pSCNN model has produced inclusion probabilities for N mixture–compound spectral pairs and you must convert these probabilities into a thresholded, descending-rank list of candidate components of an NMR mixture.

## When NOT to use

- You have raw NMR spectra but no trained pSCNN model or database compound spectra to form the mixture–compound spectral pairs — training and data augmentation come first
- The task is training or validating the pSCNN model itself (e.g., splitting the augmented dataset into training, validation and test sets) rather than predicting components of an unknown mixture
- Compounds in the mixture are not represented in the spectral database, since identification is restricted to database compounds scored by the model

## Inputs

- Trained pSCNN model (TensorFlow/Keras) reloaded from Zenodo deposit 10.5281/zenodo.6504814
- Unknown NMR mixture spectrum (e.g., U1) from an experimental dataset such as the flavor mixtures dataset
- N database compound NMR spectra paired one-to-one with the mixture spectrum as pSCNN inputs

## Outputs

- Inclusion probability per mixture–database compound spectral pair
- Thresholded (probability ≥ 0.5) candidate component list sorted by descending predicted probability, written to candidate_components.csv

## How to apply

Form N spectral pairs, each pairing the unknown mixture spectrum with one database compound spectrum, and run the prediction step of the pre-trained pSCNN model (the 'pSCNN.py' entry point under Python 3.8.13 with TensorFlow 2.5.0-GPU) so the network outputs one inclusion probability per pair. Apply a probability threshold of 0.5 to flag candidate components, then sort the flagged compounds by descending predicted probability and write the ranked list with probabilities to candidate_components.csv. The ranking rationale is that higher probabilities indicate stronger evidence that the compound's spectrum is contained in the mixture, and the CNN's translational invariance absorbs chemical shift variation between mixtures. Validate the output ordering against known ground truth where available — for the U1 mixture the expected high-to-low order is β-Ionone, γ-Decalactone, γ-Nonanoic lactone, Citral, Leaf alcohol, Isovaleric acid and 2-Methylbutyric acid.

## Related tools

- **pSCNN** (Pseudo-Siamese CNN that outputs an inclusion probability for each mixture–database compound spectral pair; run via 'pSCNN.py' for prediction) — https://github.com/yuxuanliao/pSCNN.git
- **TensorFlow 2.5.0-GPU** (Deep learning framework in which the pre-trained pSCNN model is reloaded and the prediction step is executed)
- **Python 3.8.13** (Runtime environment (installed via conda environment 'pSCNN') for executing the prediction and ranking workflow)
- **tqdm** (Progress-bar dependency installed with pip for the pSCNN package requirements)

## Examples

```
python pSCNN.py  # reload the Zenodo-trained model, predict inclusion probabilities for the U1 mixture vs N database spectra, threshold at 0.5, and rank candidates
```

## Evaluation signals

- The high-to-low probability ranking for the U1 mixture matches the reported order: β-Ionone, γ-Decalactone, γ-Nonanoic lactone, Citral, Leaf alcohol, Isovaleric acid and 2-Methylbutyric acid
- Every predicted inclusion probability lies in [0, 1] and there is exactly one probability per formed spectral pair (N pairs → N predictions)
- Only compounds with predicted probability above the 0.5 threshold appear in the ranked candidate list
- Benchmark agreement matches reported metrics, e.g., ACC = 97.62%, TPR = 96.44% and FPR = 2.29% on the flavor mixtures dataset
- candidate_components.csv contains both the compound names and their predicted probabilities and is sorted by descending probability

## Limitations

- Real-data performance degrades relative to the augmented test set (ACC = 97.62% on flavor mixtures vs 99.80% on the augmented test set; ACC = 91.67%, FPR = 10.53% on the additional flavor mixture dataset), so rankings on experimental spectra may include false positives
- Model and data exceed the GitHub repo limits and must be downloaded from Zenodo (10.5281/zenodo.6504814), adding a setup dependency
- No discussion-section content on limitations and reproducibility gaps was available in the extracted document

## Evidence

- [abstract] the proposed method can achieve good performances in the augmented test set (ACC = 99.80%, TPR = 99.70% and FPR = 0.10%): "the proposed method can achieve good performances in the augmented test set (ACC = 99.80%, TPR = 99.70% and FPR = 0.10%)"
- [abstract] the flavor mixtures dataset (ACC = 97.62%, TPR = 96.44% and FPR = 2.29%): "the flavor mixtures dataset (ACC = 97.62%, TPR = 96.44% and FPR = 2.29%)"
- [methods] Run the file 'pSCNN.py'. The model and these data have been uploaded at Zenodo.: "Run the file 'pSCNN.py'. The model and these data have been uploaded at Zenodo."
- [methods] Download the model and these example data, pSCNN can be reload and predict easily.: "Download the model and these example data, pSCNN can be reload and predict easily."
- [readme] Python 3.8.13 and TensorFlow (version 2.5.0-GPU): "Python 3.8.13 and TensorFlow (version 2.5.0-GPU)"
