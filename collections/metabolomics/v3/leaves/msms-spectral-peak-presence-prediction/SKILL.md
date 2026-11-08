---
name: msms-spectral-peak-presence-prediction
description: Use when you need in silico MS/MS spectra for metabolites lacking empirical
  library entries, or when predicting binary peak presence at specific m/z bins (2-decimal
  resolution) is preferred over intensity regression across collision energies.
when_to_use_negative:
- When you need calibrated peak intensities rather than binary presence/absence —
  SingleFrag deliberately predicts presence, not intensity
- When the molecule exceeds the representable chemistry of the bundled Mol2vec/threshold
  assets — only the most frequented 1,000 MZ positions in the training set have thresholds
  and models
- When you require curation-verified multi-collision-energy reference spectra — the
  paper notes GNPS, MSDial, and MassBank typically contain spectra at only one or
  a few collision energies per compound
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: SingleFrag
  role: Per-MZ-position ANN/GNN/COM classifiers, prediction notebook (spectraPrediction.ipynb)
    and threshold files used to predict binary peak presence and reconstruct spectra
  repo: https://github.com/MaribelPR/SingleFrag
- name: mol2vec
  role: Embeds input SMILES into the 300-dimensional vector that serves as the ANN
    model input
- name: 3DMolMS
  role: State-of-the-art 3D-structure-aware MS/MS predictor used as the performance
    comparator; its MolNet encoder can also supply molecular embeddings/features
  repo: josiehong/3DMolMS
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.11.04.621329
    title: 'SingleFrag: A deep learning tool for MS/MS fragment and spectral prediction
      and metabolite annotation'
  source_content_hash: sha256:c719b8d1be142267f4dedb07b01afde12ca08da90f498772bbb1d03a2bce5a19
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/MaribelPR/SingleFrag
  tools:
  - 3DMolMS
  - SingleFrag
  - mol2vec
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# msms-spectral-peak-presence-prediction

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Predict the presence or absence of each MS/MS peak at individual MZ positions using SingleFrag's per-bin deep learning classifiers (ANN/GNN/COM), which train a separate model per fragment ion rather than one model for the whole spectrum.

## When to use

Use when you need in silico MS/MS spectra for metabolites lacking empirical library entries, or when predicting binary peak presence at specific m/z bins (2-decimal resolution) is preferred over intensity regression across collision energies.

## When NOT to use

- When you need calibrated peak intensities rather than binary presence/absence — SingleFrag deliberately predicts presence, not intensity
- When the molecule exceeds the representable chemistry of the bundled Mol2vec/threshold assets — only the most frequented 1,000 MZ positions in the training set have thresholds and models
- When you require curation-verified multi-collision-energy reference spectra — the paper notes GNPS, MSDial, and MassBank typically contain spectra at only one or a few collision energies per compound

## Inputs

- Molecule SMILES strings (e.g. inputMols_git.xlsx with columns smiles, name, spectrum, link)
- SingleFrag Figshare dataset (trained ModelVal*.pickle per MZ bin, Mol2vec .model, thresholds*.pickle, AllMostFreqMolGeneral_rep_dades1.pickle)
- Optional experimental MS/MS spectra (e.g. HMDB spectra, positive mode) for prediction-vs-experimental comparison

## Outputs

- Per-MZ-position prediction pickles (results/dfPredictionPeak_***.pickle)
- Reconstructed predicted MS/MS spectra (results/predSpectra.json)
- Optional graphical comparison of predicted vs experimental spectra
- Optional retrained per-bin models via ANNmodel.py/GNNmodel.py/COMmodel.py

## How to apply

Download the SingleFrag dataset from Figshare, which contains per-bin trained models (ModelVal*.pickle, named with the m/z position written as mz*100), a Mol2vec model (Mol2vecModelGeneral_rep_dades1.model) that converts input SMILES into a 300-dimensional embedding vector, and per-position threshold files (thresholdsANN/GNN/COM.pickle) for the most frequented 1,000 MZ positions in the training set. Run spectraPrediction.ipynb to generate per-MZ-position prediction pickles (results/dfPredictionPeak_***.pickle); a peak is predicted at a position when the model output equals or exceeds that position's stored threshold. Reconstruct the full spectrum by aggregating all per-bin predictions into results/predSpectra.json, considering two-decimal m/z resolution rather than integer masses. Prefer the ANN models: they obtained better results than the GNN and COM variants, and the notebook uses ANN by default. For retraining, prepare training/validation/test splits and use ANNmodel.py, GNNmodel.py, or COMmodel.py per MZ position.

## Related tools

- **SingleFrag** (Per-MZ-position ANN/GNN/COM classifiers, prediction notebook (spectraPrediction.ipynb) and threshold files used to predict binary peak presence and reconstruct spectra) — https://github.com/MaribelPR/SingleFrag
- **mol2vec** (Embeds input SMILES into the 300-dimensional vector that serves as the ANN model input)
- **3DMolMS** (State-of-the-art 3D-structure-aware MS/MS predictor used as the performance comparator; its MolNet encoder can also supply molecular embeddings/features) — josiehong/3DMolMS

## Examples

```
python ANNmodel.py --mz-position <bin>  # after: pip install -r requirements.txt (Python 3.8.19) and downloading the SingleFrag dataset from figshare.com/articles/dataset/SingleFrag/27266787; then run spectraPrediction.ipynb to emit results/dfPredictionPeak_***.pickle and results/predSpectra.json
```

## Evaluation signals

- Predicted peak at a given MZ position must satisfy output >= that position's threshold from thresholdsANN.pickle for the most frequented 1,000 training-set MZ positions
- Reconstructed spectra (predSpectra.json) should be present for every input molecule, with m/z values at 2-decimal resolution rather than integers
- Held-out test-set mean per-bin F1 should be reported; the mol2vec-embedding ANN reference is 0.411, and all SingleFrag models should outperform CFM-ID and 3DMolMS
- Per-position prediction pickles (dfPredictionPeak_***.pickle) must exist before spectrum reconstruction, one per predicted MZ bin
- Models must be run under Python 3.8.19 with the requirements.txt environment since all .pickle assets were created with that version

## Limitations

- Only the most frequented 1,000 MZ positions in the training set have trained models and thresholds; peaks outside these bins cannot be predicted
- The toy input spectrum format contains only integers while the model predicts fragments at 2-decimal resolution, so comparisons with coarse experimental spectra are limited
- GNN performance is slightly lower than ANN, attributed to deliberately simple, unoptimized architectures rather than a fundamental limitation of graph neural networks
- Predictions are binary presence/absence; intensity information across collision energies is not modeled

## Evidence
