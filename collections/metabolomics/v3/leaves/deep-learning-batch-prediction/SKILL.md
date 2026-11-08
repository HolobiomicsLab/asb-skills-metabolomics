---
name: deep-learning-batch-prediction
description: Use when you have a pretrained model plus a set of NMR mixture spectra
  (e.g. from a Zenodo deposit) and need to predict compound identification and compute
  ACC/TPR/FPR metrics for the whole dataset in one batch run.
when_to_use_negative:
- You need to train a pSCNN model from scratch or run the data augmentation/superposition
  pipeline — this skill covers prediction only, not training
- The input is not 1D NMR spectra of mixtures of small molecules (e.g. chromatography
  or MS data), since pSCNN was built for NMR chemical shift data
- You need to solve the chemical shift variation/peak overlapping problem itself rather
  than score an already-trained model on new spectra
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3420
- http://edamontology.org/topic_3377
tools:
- name: pSCNN
  role: Pseudo Siamese convolutional neural network whose pretrained model is reloaded
    to predict compound identification in mixture NMR spectra
  repo: https://github.com/yuxuanliao/pSCNN.git
- name: TensorFlow 2.5.0-GPU
  role: Deep learning backend used to load and execute the pSCNN model for batch prediction
- name: Python 3.8.13
  role: Runtime environment (pSCNN conda env) in which prediction and metric computation
    are executed
- name: Anaconda
  role: Environment manager used to create the pSCNN conda environment with the required
    packages
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/molecules27123653
    title: Deep Learning-Based Method for Compound Identification in NMR Spectra of
      Mixtures
  source_content_hash: sha256:0ad26015986122d6a4fe99921297ab23a2c33cfee5446e8930b10deabac0489e
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
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3420
  - http://edamontology.org/topic_3377
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# deep-learning-batch-prediction

## Summary

Reload a pretrained deep learning model (pSCNN, a pseudo Siamese CNN for compound identification in NMR spectra of mixtures) and run batch prediction over NMR mixture spectra datasets, then compute ACC, TPR, and FPR per dataset.

## When to use

Use when you have a pretrained model plus a set of NMR mixture spectra (e.g. from a Zenodo deposit) and need to predict compound identification and compute ACC/TPR/FPR metrics for the whole dataset in one batch run.

## When NOT to use

- You need to train a pSCNN model from scratch or run the data augmentation/superposition pipeline — this skill covers prediction only, not training
- The input is not 1D NMR spectra of mixtures of small molecules (e.g. chromatography or MS data), since pSCNN was built for NMR chemical shift data
- You need to solve the chemical shift variation/peak overlapping problem itself rather than score an already-trained model on new spectra

## Inputs

- Pretrained pSCNN model files from Zenodo deposit 10.5281/zenodo.6504814
- NMR mixture spectra datasets: 2000-pair augmented test set, flavor mixtures dataset, additional flavor mixture dataset
- pSCNN.py script and requirements.txt from the yuxuanliao/pSCNN repository

## Outputs

- Per-dataset prediction results (compound identification calls for each mixture spectrum)
- Metrics table (e.g. pSCNN_prediction_metrics.csv) with columns dataset, ACC, TPR, FPR — one row per dataset

## How to apply

Download the pretrained pSCNN model and the example NMR datasets (2000-pair augmented test set, flavor mixtures, additional flavor mixture) from Zenodo deposit 10.5281/zenodo.6504814. Create the pSCNN conda environment (Anaconda, Python 3.8.13, TensorFlow 2.5.0-GPU) and install the main packages from requirements.txt plus tqdm. Reload the model and execute pSCNN.py to run batch prediction over the mixture spectra. Parse the prediction results per dataset and compute accuracy (ACC), true positive rate (TPR), and false positive rate (FPR), assembling the nine metric values into a table with one row per dataset and columns dataset, ACC, TPR, FPR.

## Related tools

- **pSCNN** (Pseudo Siamese convolutional neural network whose pretrained model is reloaded to predict compound identification in mixture NMR spectra) — https://github.com/yuxuanliao/pSCNN.git
- **TensorFlow 2.5.0-GPU** (Deep learning backend used to load and execute the pSCNN model for batch prediction)
- **Python 3.8.13** (Runtime environment (pSCNN conda env) in which prediction and metric computation are executed)
- **Anaconda** (Environment manager used to create the pSCNN conda environment with the required packages)

## Examples

```
conda activate pSCNN && python pSCNN.py  # reload model from Zenodo (10.5281/zenodo.6504814) and predict the three mixture spectra datasets
```

## Evaluation signals

- Computed metrics match the reported values: augmented test set (ACC = 99.80%, TPR = 99.70%, FPR = 0.10%)
- Flavor mixtures dataset metrics match: ACC = 97.62%, TPR = 96.44%, FPR = 2.29%
- Additional flavor mixture metrics match: ACC = 91.67%, TPR = 100.00%, FPR = 10.53%
- Metrics table contains nine values total (3 metrics × 3 datasets) with the required columns dataset, ACC, TPR, FPR
- Model reload succeeds in the pSCNN conda environment (Python 3.8.13, TensorFlow 2.5.0-GPU) without version errors

## Limitations

- The trained model exceeded GitHub's file-size limit and must be obtained from Zenodo (10.5281/zenodo.6504814) rather than the repository
- The paper's discussion section content (limitations, reproducibility gaps) was not extractable — no changelog found
- Performance degrades on harder experimental data: the additional flavor mixture gives FPR = 10.53% and ACC = 91.67%, well below the augmented test set's 99.80% ACC

## Evidence

- [methods] Run the file 'pSCNN.py'. The model and these data have been uploaded at Zenodo.: "Run the file 'pSCNN.py'. The model and these data have been uploaded at Zenodo. Download the model and these example data, pSCNN can be reload and predict easily."
- [readme] Python 3.8.13 and TensorFlow (version 2.5.0-GPU): "Python 3.8.13 and TensorFlow (version 2.5.0-GPU)"
- [abstract] Two experimental NMR datasets (flavor mixtures and additional flavor mixture) were acquired to benchmark its performance in real applications.: "Two experimental NMR datasets (flavor mixtures and additional flavor mixture) were acquired to benchmark its performance in real applications."
- [readme] Since the model exceeded the limit, we have uploaded the model and all the NMR data to Zenodo.: "Since the model exceeded the limit, we have uploaded the model and all the NMR data to Zenodo."
