---
name: pscnn-inclusion-probability-prediction
description: Use when a single NMR mixture spectrum must be decomposed into its constituent
  compounds against a library of known compound NMR spectra, especially when chemical
  shift variation and peak overlapping complicate matching.
when_to_use_negative:
- You need to identify compounds in a pure (single-compound) spectrum rather than
  deconvolve a mixture — pSCNN was built for compound identification in NMR spectra
  of mixtures
- No pre-trained model or training data are available and you cannot train pSCNN —
  the method requires the model uploaded to Zenodo or the data-augmentation training
  pipeline
- The target spectra are not NMR (e.g. MS or IR) — the translational-invariance argument
  and training data are specific to NMR spectral representations
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3474
tools:
- name: pSCNN
  role: Pseudo-Siamese CNN model that scores each mixture-compound spectral pair with
    an inclusion probability; run via 'pSCNN.py'
  repo: https://github.com/yuxuanliao/pSCNN.git
- name: TensorFlow 2.5.0-GPU
  role: Deep-learning backend used to reload and run the pre-trained pSCNN model for
    prediction
- name: Python 3.8.13
  role: Runtime environment (Anaconda env 'pSCNN') in which the prediction pipeline
    is executed
- name: tqdm
  role: Progress-bar dependency installed alongside the prediction pipeline requirements
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/molecules27123653
    title: Deep Learning-Based Method for Compound Identification in NMR Spectra of
      Mixtures
  source_content_hash: sha256:c9ba0ae1b34645d12953f94108065906b64070fb7a3724debc754385ce1d138d
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
yes_rate: 0.5833333333333334
n_claims: 12
---

# pscnn-inclusion-probability-prediction

## Summary

Predicts whether each compound in an NMR spectral database is present in a mixture by feeding pseudo-Siamese CNN (pSCNN) spectral pairs of the mixture spectrum and each database compound spectrum, then thresholding the predicted inclusion probabilities at 0.5 to rank candidate components.

## When to use

Use when a single NMR mixture spectrum must be decomposed into its constituent compounds against a library of known compound NMR spectra, especially when chemical shift variation and peak overlapping complicate matching.

## When NOT to use

- You need to identify compounds in a pure (single-compound) spectrum rather than deconvolve a mixture — pSCNN was built for compound identification in NMR spectra of mixtures
- No pre-trained model or training data are available and you cannot train pSCNN — the method requires the model uploaded to Zenodo or the data-augmentation training pipeline
- The target spectra are not NMR (e.g. MS or IR) — the translational-invariance argument and training data are specific to NMR spectral representations

## Inputs

- Unknown mixture NMR spectrum (e.g. the U1 flavor mixture spectrum)
- Library of N reference compound NMR spectra from the spectral database (Zenodo 10.5281/zenodo.6504814)
- Pre-trained pSCNN model checkpoint from Zenodo 10.5281/zenodo.6504814

## Outputs

- Inclusion probability for each of the N mixture-database compound spectral pairs
- Ranked candidate component list with predicted probabilities (candidate_components.csv)

## How to apply

Download the pre-trained pSCNN model and example NMR data (mixture spectra plus N database compound spectra) from Zenodo deposit 10.5281/zenodo.6504814, then in Python 3.8.13 with TensorFlow 2.5.0-GPU (conda env 'pSCNN', requirements.txt, tqdm) reload the model. Form N spectral pairs, each pairing the unknown mixture spectrum with one database compound spectrum, in the pSCNN input format. Run the prediction step via the 'pSCNN.py' entry point so the pseudo-Siamese CNN outputs an inclusion probability for each pair. Apply a 0.5 probability threshold to flag candidate components and sort them by descending predicted probability. Validate by checking the ranked list reproduces the reported U1 mixture ranking (beta-Ionone, gamma-Decalactone, gamma-Nonanoic lactone, Citral, Leaf alcohol, Isovaleric acid, 2-Methylbutyric acid), consistent with ACC = 97.62% on the flavor mixtures dataset.

## Related tools

- **pSCNN** (Pseudo-Siamese CNN model that scores each mixture-compound spectral pair with an inclusion probability; run via 'pSCNN.py') — https://github.com/yuxuanliao/pSCNN.git
- **TensorFlow 2.5.0-GPU** (Deep-learning backend used to reload and run the pre-trained pSCNN model for prediction)
- **Python 3.8.13** (Runtime environment (Anaconda env 'pSCNN') in which the prediction pipeline is executed)
- **tqdm** (Progress-bar dependency installed alongside the prediction pipeline requirements)

## Examples

```
conda create --name pSCNN python=3.8.13; conda activate pSCNN; python -m pip install -r requirements.txt; python pSCNN.py
```

## Evaluation signals

- Every one of the N spectral pairs yields exactly one inclusion probability in [0, 1]
- Thresholding at 0.5 yields a flagged candidate set whose high-to-low ranking for U1 matches beta-Ionone, gamma-Decalactone, gamma-Nonanoic lactone, Citral, Leaf alcohol, Isovaleric acid, 2-Methylbutyric acid
- Reported reference performance on real data: ACC = 97.62%, TPR = 96.44%, FPR = 2.29% on the flavor mixtures dataset; results deviating grossly from this regime suggest pipeline errors
- candidate_components.csv contains one row per flagged compound with a descending-sorted probability column
- Model reloads successfully under Python 3.8.13 / TensorFlow 2.5.0-GPU as specified in requirements.txt

## Limitations

- Performance degrades on the additional flavor mixture dataset (ACC = 91.67%, TPR = 100.00%, FPR = 10.53%), so false-positive rates can rise on novel real mixtures
- The pre-trained model exceeded repository size limits and must be fetched from Zenodo, an external dependency for reproducibility
- Requires a GPU-capable TensorFlow 2.5.0 environment; exact version pinning (Python 3.8.13) may be needed to reload the checkpoint
- No discussion-section content was extracted, so reproducibility gaps and limitations beyond the reported metrics are undocumented

## Evidence

- [intro] We presented a pseudo Siamese convolutional neural network method (pSCNN) to solve the problems of compound identification in NMR spectra of mixtures: "We presented a pseudo Siamese convolutional neural network method (pSCNN) to solve the problems of compound identification in NMR spectra of mixtures"
- [abstract] the flavor mixtures dataset (ACC = 97.62%, TPR = 96.44% and FPR = 2.29%): "the flavor mixtures dataset (ACC = 97.62%, TPR = 96.44% and FPR = 2.29%)"
- [methods] Run the file 'pSCNN.py'.: "Run the file 'pSCNN.py'. The model and these data have been uploaded at Zenodo."
- [methods] Download the model and these example data, pSCNN can be reload and predict easily.: "Download the model and these example data, pSCNN can be reload and predict easily."
- [readme] Python 3.8.13 and TensorFlow (version 2.5.0-GPU): "Python 3.8.13 and TensorFlow (version 2.5.0-GPU)"
