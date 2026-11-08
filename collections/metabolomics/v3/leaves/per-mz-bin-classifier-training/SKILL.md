---
name: per-mz-bin-classifier-training
description: Use when building or reproducing MS/MS spectral prediction models that
  predict peak presence per m/z bin (F1-scored binary classification) instead of full-spectrum
  intensity regression, as in the SingleFrag approach.
when_to_use_negative:
- The goal is predicting peak intensities across collision energies rather than presence/absence
  of each peak — SingleFrag deliberately predicts binary peak presence
- A single whole-spectrum model is preferred over per-fragment models — the article's
  core premise is training a separate model for each fragment ion
- Only inference on already-trained models is needed — use the spectra prediction
  skill (spectraPrediction.ipynb) instead of retraining
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: SingleFrag
  role: Provides ANNmodel.py, GNNmodel.py, and COMmodel.py scripts for training per-bin
    binary classifiers, plus the Figshare dataset with splits and threshold files
  repo: https://github.com/MaribelPR/SingleFrag
- name: mol2vec
  role: Generates the 300-dimensional molecular embedding used as input to the ANN
    and combined models from molecule SMILES
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.11.04.621329
    title: 'SingleFrag: A deep learning tool for MS/MS fragment and spectral prediction
      and metabolite annotation'
  source_content_hash: sha256:af553376ffa565d8cfa368ca4f42323e02e926770208bb20a5f0985dc1925f48
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/MaribelPR/SingleFrag
  tools:
  - SingleFrag
  - mol2vec
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# per-mz-bin-classifier-training

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Train separate binary classifiers (ANN, GNN, or combined ANN+GNN) for each of the 1,000 most frequented m/z bins in a tandem MS training set, then calibrate per-bin decision thresholds to predict peak presence/absence rather than intensities.

## When to use

Use when building or reproducing MS/MS spectral prediction models that predict peak presence per m/z bin (F1-scored binary classification) instead of full-spectrum intensity regression, as in the SingleFrag approach.

## When NOT to use

- The goal is predicting peak intensities across collision energies rather than presence/absence of each peak — SingleFrag deliberately predicts binary peak presence
- A single whole-spectrum model is preferred over per-fragment models — the article's core premise is training a separate model for each fragment ion
- Only inference on already-trained models is needed — use the spectra prediction skill (spectraPrediction.ipynb) instead of retraining

## Inputs

- Binarized MS/MS dataset from the SingleFrag Figshare deposit (figshare.com/articles/dataset/SingleFrag/27266787) with author-prepared training, validation, and test splits
- Molecule SMILES strings converted to 300-dimensional mol2vec embeddings (Mol2vecModelGeneral_rep_dades1.model) for ANN/COM variants
- Molecular graphs for GNN variant
- Target m/z position (mz*100 naming convention) supplied as input to each model script

## Outputs

- Trained per-bin model pickles (ModelVal*.pickle) for ANN, GNN, and COM variants under data/ANN, data/GNN, data/COM
- Per-bin threshold files thresholdsANN.pickle, thresholdsGNN.pickle, thresholdsCOM.pickle
- Mean F1 score across the 1,000 m/z bins on the test split
- Reconstructed predicted spectra per molecule

## How to apply

Load the binarized MS/MS dataset from the SingleFrag Figshare deposit and use the author-prepared training/validation/test splits. Using ANNmodel.py, train one multilayer feedforward network per m/z bin taking a 300-dimensional mol2vec embedding of the molecule SMILES as input; with GNNmodel.py train a graph neural network directly on the molecular graph for the same bins; with COMmodel.py train combined models merging both representations. Each script predicts whether a molecule presents a peak at a specific MZ position supplied as input, so the full spectrum is reconstructed by aggregating predictions across the 1,000 most frequented positions. Calibrate per-bin decision thresholds on validation spectra using thresholdsANN.pickle, thresholdsGNN.pickle, and thresholdsCOM.pickle (a prediction >= its bin's threshold means a peak is predicted). Evaluate on the 10% test split and report mean F1 across all 1,000 bins.

## Related tools

- **SingleFrag** (Provides ANNmodel.py, GNNmodel.py, and COMmodel.py scripts for training per-bin binary classifiers, plus the Figshare dataset with splits and threshold files) — https://github.com/MaribelPR/SingleFrag
- **mol2vec** (Generates the 300-dimensional molecular embedding used as input to the ANN and combined models from molecule SMILES)

## Examples

```
python ANNmodel.py --mz_position <mz*100>  # repeat per bin; then python GNNmodel.py / COMmodel.py for the same bins
```

## Evaluation signals

- Mean F1 on the test split should reproduce the reported values of 0.411 (ANN), 0.399 (combined), and 0.290 (GNN) across the 1,000 bins
- Model pickles must be named with the MZ position encoded as mz*100 (e.g. ModelVal<mz*100>.pickle)
- Threshold files must contain one threshold per each of the 1,000 most frequented MZ positions in the training set
- Environment must use Python 3.8.19 with packages from requirements.txt for pickle compatibility
- All three SingleFrag variants should outperform CFM-ID and 3DMolMS baselines when compared on the same task

## Limitations

- The GNN model shows slightly lower performance than the ANN; the authors attribute this to deliberately simple, unoptimized graph features and layer architectures rather than a fundamental limitation of GNNs
- Thresholds are tied to the 1,000 most frequented training-set MZ bins, so rare fragments outside these bins are not predicted
- Training requires preparing your own training, validation, and test sets when retraining beyond the deposited splits

## Evidence
