---
name: mol2vec-molecule-embedding
description: Use when you have molecule SMILES and need fixed-length (300-dim) molecular
  embeddings to feed SingleFrag's per-m/z-bin ANN classifiers, or when reproducing
  SingleFrag's ANN-based spectra prediction or retraining (ANNmodel.py) pipeline.
when_to_use_negative:
- You are working with the GNN variant of SingleFrag, which operates directly on the
  molecular graph rather than on a mol2vec embedding
- Your input is raw experimental spectra without molecule structures (SMILES) — mol2vec
  embedding requires a molecular structure as input
- You need atom-level or geometry-aware representations (e.g. 3D-conformation-based
  prediction as in 3DMolMS) rather than a whole-molecule fixed vector
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: mol2vec
  role: Pre-trained model (Mol2vecModelGeneral_rep_dades1.model) that converts input
    SMILES into the 300-dimensional vectors consumed by the SingleFrag ANN models
- name: SingleFrag
  role: 'Framework that consumes the mol2vec embeddings: spectraPrediction.ipynb applies
    the embeddings to per-bin ANN classifiers; ANNmodel.py retrains them'
  repo: https://github.com/MaribelPR/SingleFrag
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.11.04.621329
    title: 'SingleFrag: A deep learning tool for MS/MS fragment and spectral prediction
      and metabolite annotation'
  source_content_hash: sha256:3642182728265b8467941fc6ff2258e7f96d160cefcb8e15f6512bd98164fb52
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/MaribelPR/SingleFrag
  tools:
  - SingleFrag
  - mol2vec
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# mol2vec-molecule-embedding

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Generates a fixed 300-dimensional vector embedding for each molecule from its SMILES using the mol2vec model shipped with SingleFrag; this embedding is the sole input to the per-bin ANN peak classifiers in the SingleFrag MS/MS prediction pipeline.

## When to use

Use when you have molecule SMILES and need fixed-length (300-dim) molecular embeddings to feed SingleFrag's per-m/z-bin ANN classifiers, or when reproducing SingleFrag's ANN-based spectra prediction or retraining (ANNmodel.py) pipeline.

## When NOT to use

- You are working with the GNN variant of SingleFrag, which operates directly on the molecular graph rather than on a mol2vec embedding
- Your input is raw experimental spectra without molecule structures (SMILES) — mol2vec embedding requires a molecular structure as input
- You need atom-level or geometry-aware representations (e.g. 3D-conformation-based prediction as in 3DMolMS) rather than a whole-molecule fixed vector

## Inputs

- Molecule SMILES strings (e.g. the smiles column of inputMols_git.xlsx)
- Pre-trained mol2vec model file data/Mol2vecModel/Mol2vecModelGeneral_rep_dades1.model (from the SingleFrag Figshare deposit)
- Per-bin ANN models (data/ANN/ModelVal*.pickle) and per-bin thresholds (data/modelData/thresholdsANN.pickle) when embeddings are used for prediction

## Outputs

- One 300-dimensional mol2vec embedding vector per input molecule
- Per-MZ-position peak predictions (results/dfPredictionPeak_***.pickle) when used inside the ANN prediction workflow
- Reconstructed predicted MS/MS spectra (results/predSpectra.json)

## How to apply

Set up a Python 3.8.19 environment with the packages in SingleFrag's requirements.txt, then download the SingleFrag Figshare dataset, which contains the pre-trained Mol2vec model file Mol2vecModelGeneral_rep_dades1.model under data/Mol2vecModel/. Apply this mol2vec model to each input SMILES from inputMols_git.xlsx to obtain a 300-dimensional vector per molecule — this vector, not the molecular graph, is the input representation for the ANN models. For spectra prediction, drive the workflow through spectraPrediction.ipynb, which computes predictions per MZ position for the 1,000 most frequented m/z bins and then reconstructs spectra (results/predSpectra.json); a peak is called only if the ANN's probability meets or exceeds the per-bin threshold in thresholdsANN.pickle. For retraining (ANNmodel.py), the same mol2vec embeddings feed one binary classifier per m/z position, which was the best-performing SingleFrag variant (mean F1 ≈ 0.411).

## Related tools

- **mol2vec** (Pre-trained model (Mol2vecModelGeneral_rep_dades1.model) that converts input SMILES into the 300-dimensional vectors consumed by the SingleFrag ANN models)
- **SingleFrag** (Framework that consumes the mol2vec embeddings: spectraPrediction.ipynb applies the embeddings to per-bin ANN classifiers; ANNmodel.py retrains them) — https://github.com/MaribelPR/SingleFrag

## Examples

```
jupyter notebook spectraPrediction.ipynb
```

## Evaluation signals

- Each molecule yields a vector of exactly 300 dimensions (the README states the input to ANN models is a 300-dim vector generated by applying Mol2vec to the SMILES)
- Embedding generation and downstream pickles require Python 3.8.19, as every .pickle file in the SingleFrag deposit was created with that version
- Predicted peaks must satisfy the per-bin decision threshold from thresholdsANN.pickle for the corresponding m/z bin (1,000 most frequented training-set positions)
- On the 10% test split, the ANN pipeline built on these embeddings should reach a mean F1 of about 0.411 across the 1,000 bins, matching the reported reproduction target

## Limitations

- mol2vec embeddings power only the ANN variant; the GNN variant works directly on the molecular graph and the combined (COM) variant merges both representations
- The ANN/GNN architectures in SingleFrag were deliberately kept simple and are not optimized, so embedding-based performance may under-represent what graph models could achieve
- The toy input spectra in inputMols_git.xlsx contain integer m/z values, while the models predict fragments with 2-decimal resolution, so experimental-vs-predicted comparisons in the toy example are coarse

## Evidence
