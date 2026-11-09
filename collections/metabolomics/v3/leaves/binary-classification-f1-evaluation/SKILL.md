---
name: binary-classification-f1-evaluation
description: Use when the goal is to compare classifier variants that predict peak
  presence/absence per m/z bin from molecular structure (SMILES), and benchmark them
  with mean F1 on a test split rather than predicting full spectra or peak intensities.
when_to_use_negative:
- The task is full-spectrum prediction or peak-intensity regression rather than binary
  peak presence/absence classification — SingleFrag deliberately focuses on presence/absence,
  which it argues is more robust across collision energies.
- The molecules fall outside the model's training domain or lack a usable SMILES,
  so no mol2vec embedding or molecular graph can be computed.
- Only a small number of m/z bins are of interest and no per-bin calibration data
  (validation spectra) are available, making threshold-based peak calling unreliable.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: SingleFrag
  role: Provides the ANNmodel.py, GNNmodel.py, and COMmodel.py training scripts and
    the binarized MS/MS dataset with author-prepared splits for per-bin binary classification
    and F1 evaluation.
  repo: https://github.com/MaribelPR/SingleFrag
- name: mol2vec
  role: Embeds each input SMILES into a 300-dimensional vector used as ANN model input;
    a custom-trained mol2vec model is distributed in the Figshare deposit.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.11.04.621329
    title: 'SingleFrag: A deep learning tool for MS/MS fragment and spectral prediction
      and metabolite annotation'
  source_content_hash: sha256:1e21c916b2ab1ebdb9f640247d28f3abec35f74cd5dfb550aef6aae0ad0eaa59
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/MaribelPR/SingleFrag
  tools:
  - SingleFrag
  - mol2vec
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# binary-classification-f1-evaluation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Trains and evaluates per-m/z-bin binary classifiers (ANN, GNN, and combined variants from SingleFrag) that predict peak presence/absence, and reports mean F1 across the 1,000 most frequented m/z positions on a held-out test split.

## When to use

Use when the goal is to compare classifier variants that predict peak presence/absence per m/z bin from molecular structure (SMILES), and benchmark them with mean F1 on a test split rather than predicting full spectra or peak intensities.

## When NOT to use

- The task is full-spectrum prediction or peak-intensity regression rather than binary peak presence/absence classification — SingleFrag deliberately focuses on presence/absence, which it argues is more robust across collision energies.
- The molecules fall outside the model's training domain or lack a usable SMILES, so no mol2vec embedding or molecular graph can be computed.
- Only a small number of m/z bins are of interest and no per-bin calibration data (validation spectra) are available, making threshold-based peak calling unreliable.

## Inputs

- Binarized MS/MS dataset from the SingleFrag Figshare deposit (training/validation/test split as prepared by the authors)
- Molecule SMILES strings (inputMols_git.xlsx format)
- Mol2vec model file (Mol2vecModelGeneral_rep_dades1.model) producing 300-dimensional molecular embeddings
- Per-bin threshold files: thresholdsANN.pickle, thresholdsGNN.pickle, thresholdsCOM.pickle
- AllMostFreqMolGeneral_rep_dades1.pickle (peak counts per m/z bin identifying the 1,000 most frequented positions)

## Outputs

- Trained per-bin models (ModelVal*.pickle) for ANN, GNN, and COM variants
- Per-MZ-position prediction files (results/dfPredictionPeak_***.pickle)
- Mean F1 score across the 1,000 m/z bins per variant, computed on the 10% test split (expected ~0.411 ANN, ~0.399 COM, ~0.290 GNN)

## How to apply

Load the binarized MS/MS dataset from the SingleFrag Figshare deposit (https://figshare.com/articles/dataset/SingleFrag/27266787) and use the training, validation, and test split as prepared by the authors. For each of the 1,000 most frequented m/z positions, train a separate binary classifier: an ANN fed a 300-dimensional mol2vec embedding of the input SMILES (ANNmodel.py), a graph neural network operating directly on the molecular graph (GNNmodel.py), and a combined model merging both representations (COMmodel.py). Calibrate per-bin decision thresholds using validation spectra (thresholdsANN.pickle, thresholdsGNN.pickle, thresholdsCOM.pickle); a prediction at or above its bin-specific threshold means a peak is called for that bin. Evaluate on the 10% test split and report the mean F1 across all 1,000 bins for each variant. The rationale is that per-fragment models capture features specific to each peak, and the article reports the GNN's lower mean F1 as a consequence of deliberately simple, unoptimized architectures rather than a fundamental limitation of graph neural networks.

## Related tools

- **SingleFrag** (Provides the ANNmodel.py, GNNmodel.py, and COMmodel.py training scripts and the binarized MS/MS dataset with author-prepared splits for per-bin binary classification and F1 evaluation.) — https://github.com/MaribelPR/SingleFrag
- **mol2vec** (Embeds each input SMILES into a 300-dimensional vector used as ANN model input; a custom-trained mol2vec model is distributed in the Figshare deposit.)

## Examples

```
python ANNmodel.py  # train the per-bin binary classifier for a specified MZ position, then evaluate per-bin F1 on the 10% test split
```

## Evaluation signals

- Mean F1 on the 10% test split reproduces the reported values: 0.411 (ANN), 0.399 (combined), 0.290 (GNN).
- ANN and combined variants outperform the GNN; a reversal of this ordering indicates a split, threshold-calibration, or implementation error.
- Predictions cover exactly the 1,000 most frequented m/z positions identified in AllMostFreqMolGeneral_rep_dades1.pickle.
- Per-bin threshold files contain one threshold per m/z bin (file names encode mz*100), and a peak is called only when the prediction is >= the bin-specific threshold.
- Per-MZ-position prediction pickles (results/dfPredictionPeak_***.pickle) exist before spectra reconstruction, confirming the two-stage prediction flow was followed.

## Limitations

- The GNN's lower mean F1 (0.290 vs 0.411 for ANN) reflects deliberately simple, unoptimized graph features and layer architectures, not a fundamental limitation of graph neural networks for spectral prediction.
- Only the ANN models are used in the released spectra-prediction notebook because they achieved better results than GNN and COM.
- Threshold files and all .pickle artifacts were created with Python 3.8.19; the environment (requirements.txt, Python 3.8.19) must match to load them correctly.

## Evidence
