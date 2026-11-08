---
name: mol2vec-molecular-embedding
description: Use when a pipeline needs to convert candidate molecule SMILES (e.g.,
  the inputMols_git.xlsx SMILES column) into numerical feature vectors before running
  SingleFrag's ANN, GNN, or COM peak-prediction models.
when_to_use_negative:
- You plan to use the GNN models, which work directly on the molecular graph rather
  than a mol2vec embedding
- You need spectra of molecules with non-standard elements outside those supported
  by the trained models, or spectra at collision energies/stages the SingleFrag models
  were not trained on
- You want raw molecular descriptors or fingerprints for general cheminformatics tasks
  unrelated to SingleFrag's fixed 1,000-MZ-position peak-presence setup
tools:
- name: mol2vec
  role: Pretrained embedding model that converts molecule SMILES into the 300-dimensional
    input vectors for SingleFrag's ANN peak-prediction models
  repo: https://github.com/MaribelPR/SingleFrag
- name: SingleFrag
  role: Downstream deep learning tool whose spectraPrediction.ipynb consumes the mol2vec
    embeddings to predict peak presence at the most frequented MZ positions
  repo: https://github.com/MaribelPR/SingleFrag
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1101/2024.11.04.621329
    title: 'SingleFrag: A deep learning tool for MS/MS fragment and spectral prediction
      and metabolite annotation'
  source_content_hash: sha256:0a6d426114a0b94db28d43d4812eade07438707dabd3b6b287da250736fd0bbe
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/MaribelPR/SingleFrag
  tools:
  - 3DMolMS
  - CFM-ID
  - SingleFrag
  - mol2vec
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# mol2vec-molecular-embedding

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Embeds input molecule SMILES into fixed 300-dimensional vectors using the pretrained Mol2vec model (Mol2vecModelGeneral_rep_dades1.model) distributed with the SingleFrag dataset, producing the feature representation required by SingleFrag's per-MZ-position ANN models.

## When to use

Use when a pipeline needs to convert candidate molecule SMILES (e.g., the inputMols_git.xlsx SMILES column) into numerical feature vectors before running SingleFrag's ANN, GNN, or COM peak-prediction models.

## When NOT to use

- You plan to use the GNN models, which work directly on the molecular graph rather than a mol2vec embedding
- You need spectra of molecules with non-standard elements outside those supported by the trained models, or spectra at collision energies/stages the SingleFrag models were not trained on
- You want raw molecular descriptors or fingerprints for general cheminformatics tasks unrelated to SingleFrag's fixed 1,000-MZ-position peak-presence setup

## Inputs

- Molecule SMILES strings (e.g., SMILES column of inputMols_git.xlsx)
- Pretrained Mol2vec model file Mol2vecModelGeneral_rep_dades1.model from the SingleFrag Figshare dataset

## Outputs

- One 300-dimensional embedding vector per input molecule, used as input to the trained ANN/ModelVal*.pickle models
- Per-MZ-position predictions written to results/dfPredictionPeak_*.pickle and reconstructed spectra in results/predSpectra.json (downstream of the embeddings)

## How to apply

Download the SingleFrag dataset from Figshare (https://figshare.com/articles/dataset/SingleFrag/27266787) and place the Mol2vec model at data/Mol2vecModel/Mol2vecModelGeneral_rep_dades1.model, working under Python 3.8.19 with the packages in requirements.txt. Read the SMILES column from the input file (inputMols_git.xlsx) and apply the Mol2vec model to each SMILES to obtain a 300-dimensional vector per molecule; this vector is the required input format for the trained ANN models (ANN/ModelVal*.pickle), where file names encode the m/z position as mz*100 for the 1,000 most frequented MZ positions. Embedding is a per-molecule preprocessing step done once and reused across all MZ-position models, and peak presence is subsequently called only when a model's prediction is greater than or equal to the position-specific threshold in thresholdsANN.pickle. As the SingleFrag README notes, only the ANN models are used in spectraPrediction.ipynb because they outperformed GNN and COM, so the mol2vec embedding is the canonical input representation for the recommended workflow.

## Related tools

- **mol2vec** (Pretrained embedding model that converts molecule SMILES into the 300-dimensional input vectors for SingleFrag's ANN peak-prediction models) — https://github.com/MaribelPR/SingleFrag
- **SingleFrag** (Downstream deep learning tool whose spectraPrediction.ipynb consumes the mol2vec embeddings to predict peak presence at the most frequented MZ positions) — https://github.com/MaribelPR/SingleFrag

## Examples

```
python -m jupyter nbconvert --to notebook --execute spectraPrediction.ipynb
```

## Evaluation signals

- Each input SMILES yields exactly one 300-dimensional vector (dimensionality check against the expected 300 features)
- The embedding is computed with Mol2vecModelGeneral_rep_dades1.model under Python 3.8.19, matching the environment used to create the .pickle model files
- Downstream ANN predictions reproduce peak calls only where prediction >= the position-specific threshold in thresholdsANN.pickle
- Running the full pipeline (embeddings -> ANN -> reconstruction) regenerates results/dfPredictionPeak_*.pickle and results/predSpectra.json consistent with spectraPrediction.ipynb

## Limitations

- The ANN model (and thus the mol2vec embedding path) is only wired to the 1,000 most frequented MZ positions in the SingleFrag training set; peaks outside these positions cannot be predicted
- The GNN variant does not use mol2vec embeddings, so an embedding-only pipeline cannot reproduce GNN or COM model behavior
- Embeddings are only as good as the Mol2vec model shipped with the dataset; substituting a different mol2vec model would break compatibility with the trained ModelVal*.pickle files

## Evidence
