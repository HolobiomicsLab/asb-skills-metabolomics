---
name: molecular-structure-feature-augmentation
description: Use when a per-MZ-position peak classifier built on 2D-only molecular
  embeddings (mol2vec or graph features) underperforms and you want to test whether
  3D conformer-derived structure features lift per-bin F1.
when_to_use_negative:
- Your goal is full-spectrum intensity regression rather than per-fragment peak-presence
  classification — SingleFrag deliberately focuses on presence/absence per peak, not
  intensities.
- Your molecules cannot be embedded into 3D conformations (e.g., unsupported atom
  types or >300 atoms), which is outside 3DMolMS's supported input space.
- You only need annotation of experimental spectra against an existing library, not
  improved in silico fragment prediction models.
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3332
tools:
- name: SingleFrag
  role: Provides the per-MZ-position ANN/GNN/COM classifier framework, the preprocessed
    dataset splits, and per-bin F1 evaluation against which the 3D-augmented model
    is benchmarked.
  repo: https://github.com/MaribelPR/SingleFrag
- name: 3DMolMS
  role: 'Source of the 3D-structure feature concept: a deep network that predicts
    MS/MS from 3D molecular conformations and whose learned representation (features)
    can be extracted via save_features().'
  repo: https://github.com/JosieHong/3DMolMS
- name: mol2vec
  role: Baseline 2D molecular embedding (300-dim) that is concatenated with 3D features
    as input to the per-bin classifier.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.11.04.621329
    title: 'SingleFrag: A deep learning tool for MS/MS fragment and spectral prediction
      and metabolite annotation'
  source_content_hash: sha256:adfab43395aebb7b32efc36d055c8cdcc1206bdbcdb8f1e455f4fd4b4eb3038b
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/MaribelPR/SingleFrag
  tools:
  - 3DMolMS
  - SingleFrag
  - mol2vec
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# molecular-structure-feature-augmentation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Augments existing molecular input representations (e.g., 300-dim mol2vec embeddings or molecular graphs) with 3D atom-location structural features derived from computed conformers, as used by 3DMolMS, to improve per-fragment MS/MS peak-presence classification.

## When to use

Use when a per-MZ-position peak classifier built on 2D-only molecular embeddings (mol2vec or graph features) underperforms and you want to test whether 3D conformer-derived structure features lift per-bin F1.

## When NOT to use

- Your goal is full-spectrum intensity regression rather than per-fragment peak-presence classification — SingleFrag deliberately focuses on presence/absence per peak, not intensities.
- Your molecules cannot be embedded into 3D conformations (e.g., unsupported atom types or >300 atoms), which is outside 3DMolMS's supported input space.
- You only need annotation of experimental spectra against an existing library, not improved in silico fragment prediction models.

## Inputs

- SingleFrag dataset from Figshare (https://figshare.com/articles/dataset/SingleFrag/27266787) with training/validation/test splits
- Molecule SMILES strings (inputMols_git.xlsx format)
- mol2vec 300-dimensional embedding model (Mol2vecModelGeneral_rep_dades1.model)
- 3D conformers / atom-coordinate descriptors for each molecule

## Outputs

- Per-MZ-position binary peak-presence predictions over the most frequented 1,000 MZ positions
- Reconstructed predicted MS/MS spectra (e.g., predSpectra.json)
- Per-bin F1 scores on the held-out test set, compared against the 0.411 mol2vec-ANN baseline (CSV)

## How to apply

Download the SingleFrag dataset from Figshare (figshare.com/articles/dataset/SingleFrag/27266787) and reuse the training, validation and test splits already used by ANNmodel.py. Generate 3D molecular conformations for each input SMILES and extract atom-location/geometry descriptors in the style of 3DMolMS, which predicts MS/MS spectra 'from their 3D conformations' and learns molecular representations that transfer across tasks. Concatenate these 3D features with the existing 300-dimensional mol2vec embedding (produced by Mol2vecModelGeneral_rep_dades1.model) or with the molecular graph features used by GNNmodel.py, then train a per-MZ-position binary classifier (one model per bin, mirroring ANNmodel.py) on the most frequented MZ positions in the training set. Apply the per-MZ-position prediction thresholds (thresholdsANN.pickle) to binarize peak presence, reconstruct spectra, and compute per-bin F1 on the held-out test set against the reference mean F1 of 0.411 from the mol2vec-only ANN.

## Related tools

- **SingleFrag** (Provides the per-MZ-position ANN/GNN/COM classifier framework, the preprocessed dataset splits, and per-bin F1 evaluation against which the 3D-augmented model is benchmarked.) — https://github.com/MaribelPR/SingleFrag
- **3DMolMS** (Source of the 3D-structure feature concept: a deep network that predicts MS/MS from 3D molecular conformations and whose learned representation (features) can be extracted via save_features().) — https://github.com/JosieHong/3DMolMS
- **mol2vec** (Baseline 2D molecular embedding (300-dim) that is concatenated with 3D features as input to the per-bin classifier.)

## Examples

```
molnet_engine = MolNet(device); ids, features = molnet_engine.save_features()  # 3D embeddings to concatenate with mol2vec input for per-bin ANN training
```

## Evaluation signals

- Mean per-bin F1 on the held-out test set exceeds the 0.411 reference score of the mol2vec-embedding ANN per-bin classifier.
- Predictions are made for all of the most frequented MZ positions (the 1,000 most frequent bins in the training set) and thresholded with the corresponding per-position threshold before peak calling.
- Reconstructed predicted spectrum contains peaks only at MZ bins where the thresholded classifier output crossed its per-bin threshold.
- Evaluation CSV contains a per-bin F1 entry for every MZ position modeled, enabling direct comparison with the ANN/GNN/COM baselines.

## Limitations

- The GNN baseline already underperforms the ANN, so adding 3D features does not guarantee improvement — the authors note architectures were deliberately kept simple and are not optimized.
- 3DMolMS supports only specific atom types (C, O, N, H, P, S, F, Cl, B, Br, I), at most 300 atoms, and specific precursor types ([M+H]+, [M-H]-, [M+H-H2O]+, [M+Na]+, [M+2H]2+); molecules outside this space cannot receive 3D features.
- Peak presence is threshold-dependent; per-MZ thresholds must be recomputed or reused consistently to keep F1 comparable to the baseline.

## Evidence

- [readme] 3DMolMS conformer input: [evidence span withheld — anchor status: external_doc]
