---
name: graph-neural-network-training
description: Use when reproducing or adapting per-fragment binary classifiers for
  MS/MS peak prediction where molecules are available as molecular graphs and each
  of the 1,000 most frequented m/z positions needs a separately trained GNN.
when_to_use_negative:
- The goal is overall spectrum prediction for end users — the authors use only the
  ANN models in spectraPrediction.ipynb because they got better results than the GNN
  and COM
- A single whole-spectrum model is preferred over per-fragment binary classifiers
- Peak intensity prediction is required — SingleFrag deliberately predicts only presence/absence
  of each peak
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3474
tools:
- name: SingleFrag
  role: Provides GNNmodel.py for training per-m/z-position GNN binary classifiers
    and the Figshare dataset with splits and model files
  repo: https://github.com/MaribelPR/SingleFrag
- name: mol2vec
  role: Baseline embedding used by the ANN/COM variants against which the GNN's direct
    molecular-graph input is compared
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.11.04.621329
    title: 'SingleFrag: A deep learning tool for MS/MS fragment and spectral prediction
      and metabolite annotation'
  source_content_hash: sha256:7296b04609e1b32e67dfc89a08f8f5b8b4b338249ad6c5b919199eecfa2739b7
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/MaribelPR/SingleFrag
  tools:
  - SingleFrag
  - mol2vec
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# graph-neural-network-training

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Train per-bin graph neural network binary classifiers (GNN variant of SingleFrag) that operate directly on the molecular graph to predict the presence or absence of each fragment peak in an MS/MS spectrum, and calibrate per-bin decision thresholds.

## When to use

Use when reproducing or adapting per-fragment binary classifiers for MS/MS peak prediction where molecules are available as molecular graphs and each of the 1,000 most frequented m/z positions needs a separately trained GNN.

## When NOT to use

- The goal is overall spectrum prediction for end users — the authors use only the ANN models in spectraPrediction.ipynb because they got better results than the GNN and COM
- A single whole-spectrum model is preferred over per-fragment binary classifiers
- Peak intensity prediction is required — SingleFrag deliberately predicts only presence/absence of each peak

## Inputs

- Binarized MS/MS dataset from the SingleFrag Figshare deposit (training/validation/test splits)
- Molecular graphs derived from molecule SMILES
- Most-frequented m/z bin list (AllMostFreqMolGeneral_rep_dades1.pickle)
- Per-bin threshold calibration set (validation spectra)

## Outputs

- Trained per-m/z-position GNN models (data/GNN/ModelVal*.pickle, mz encoded as mz*100)
- Per-bin decision thresholds (thresholdsGNN.pickle)
- Mean F1 score on the 10% test split (~0.290 expected)
- Per-position peak prediction files (results/dfPredictionPeak_***.pickle) for spectra reconstruction

## How to apply

Download the binarized SingleFrag dataset from Figshare and split it into the authors' training, validation, and test sets. For each of the 1,000 most frequented m/z positions in the training set spectra, run GNNmodel.py to train a graph neural network that takes the molecular graph (rather than a mol2vec embedding) as input and outputs the binary presence/absence of a peak at that m/z position. Calibrate a decision threshold per m/z bin using validation spectra, saved as thresholdsGNN.pickle; a prediction at or above its bin's threshold indicates a predicted peak. Evaluate on the 10% test split and report the mean F1 across all 1,000 bins, expecting ~0.290. Note the article reports the GNN underperforms the ANN (0.411) and combined (0.399) variants, attributed to deliberately simple, unoptimized graph features and layer architectures.

## Related tools

- **SingleFrag** (Provides GNNmodel.py for training per-m/z-position GNN binary classifiers and the Figshare dataset with splits and model files) — https://github.com/MaribelPR/SingleFrag
- **mol2vec** (Baseline embedding used by the ANN/COM variants against which the GNN's direct molecular-graph input is compared)

## Examples

```
python GNNmodel.py --mz_position <mz*100>  # one binary classifier per m/z bin, after preparing train/val/test sets and the SingleFrag Figshare data
```

## Evaluation signals

- Mean F1 across the 1,000 bins on the 10% test split reproduces ~0.290
- Threshold file thresholdsGNN.pickle contains one threshold per most-frequented m/z position, and predictions >= the bin threshold are called as peaks
- Trained model files follow the ModelVal*.pickle naming convention with the m/z position encoded as mz*100
- GNN mean F1 falls below the ANN (~0.411) and combined (~0.399) variants trained identically per bin

## Limitations

- GNN performance is slightly lower than the ANN model, attributed to unoptimized graph features and layer architectures rather than a fundamental limitation of graph neural networks
- Authors deliberately kept neural network architectures simple, so architectures are not optimized
- Pickles were created with Python 3.8.19 and the environment pins all packages in requirements.txt

## Evidence
