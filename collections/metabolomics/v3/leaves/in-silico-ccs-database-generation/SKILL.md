---
name: in-silico-ccs-database-generation
description: Use when building a searchable CCS reference library for compound identification
  from a SMILES/compound list, by predicting CCS values for multiple adduct types
  with the pretrained SigmaCCS model.
when_to_use_negative:
- Molecules containing elements other than C, H, O, N, P, S, F, Cl, Br, I, Co, As,
  and Se — SigmaCCS cannot predict them
- Compounds with molecular weight at or above 1500 Da, which are excluded by the paper's
  database filtering
- Adduct ion types beyond the three encoded ([M+H]+, [M+Na]+, [M-H]-); the paper notes
  encoding more adduct types as future work
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SigmaCCS
  role: Pretrained graph neural network (model.h5) used to predict CCS values for
    [M+H]+, [M+Na]+, and [M-H]- adducts from molecular graphs
  repo: https://github.com/zmzhang/SigmaCCS
- name: RDKit
  role: Loads SMILES, filters by molecular weight, generates ETKDG conformers, and
    minimizes them with MMFF94 before graph conversion
- name: TensorFlow
  role: Runtime for loading and executing the pretrained SigmaCCS model
- name: Spektral
  role: Graph neural network library used by the SigmaCCS model architecture for graph
    convolution layers
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42004-023-00939-w
    title: Highly accurate and large-scale collision cross sections prediction with
      graph neural networks
  source_content_hash: sha256:1064f812aa63a9c150be7fc0d008681cd98b988634a0755b155910be7fe4dfd3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmzhang/SigmaCCS
  tools:
  - RDKit
  - SigmaCCS
  - Spektral
  - TensorFlow
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# in-silico-ccs-database-generation

## Summary

Reconstructs the SigmaCCS pipeline that converts PubChem compounds (MW < 1500 Da) into 3D conformers, predicts CCS values for the [M+H]+, [M+Na]+, and [M-H]- adducts with a pretrained graph neural network, and writes the results to a CSV in-silico database like the Zenodo record 5501674 deposit.

## When to use

Use when building a searchable CCS reference library for compound identification from a SMILES/compound list, by predicting CCS values for multiple adduct types with the pretrained SigmaCCS model.

## When NOT to use

- Molecules containing elements other than C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se — SigmaCCS cannot predict them
- Compounds with molecular weight at or above 1500 Da, which are excluded by the paper's database filtering
- Adduct ion types beyond the three encoded ([M+H]+, [M+Na]+, [M-H]-); the paper notes encoding more adduct types as future work

## Inputs

- SMILES list of PubChem compounds (filtered to MW < 1500 Da and supported elements)
- parameter.pkl (adduct set, atoms set, coordinate min/max)
- model.h5 (pretrained SigmaCCS TensorFlow/Spektral model)

## Outputs

- CSV in-silico CCS database with fields Pubchem ID, SMILES, Molecular Weight, Formula, and predicted CCS values for [M+H]+, [M+Na]+, and [M-H]-
- Per-molecule predicted CCS values for three adduct ion types

## How to apply

Load a bounded sample of PubChem compounds from a SMILES list with RDKit, filtering to molecular weight below 1500 Dalton and to elements supported by the model (C, H, O, N, P, S, F, Cl, Br, I, Co, As, Se). Generate one 3D conformer per molecule via RDKit ETKDGv3 (EmbedMultipleConfs with numConfs=1, useRandomCoords=True) and minimize it with MMFF (MMFFOptimizeMoleculeConfs, numThreads=0). Load the saved parameters (adduct set, atoms set, coordinate min/max) from parameter.pkl, then convert each conformer into node feature, adjacency, and edge feature matrices with the convertToGraph routine in GraphData.py. Feed the graph plus adduct one-hot encoding into the pretrained model (model.h5, TensorFlow/Spektral) with the Model_prediction function to get CCS values for [M+H]+, [M+Na]+, and [M-H]-. Write one output row per compound containing Pubchem ID, SMILES, Molecular Weight, Formula, and the three predicted CCS values, mirroring the Zenodo (record 5501674) dataset format.

## Related tools

- **SigmaCCS** (Pretrained graph neural network (model.h5) used to predict CCS values for [M+H]+, [M+Na]+, and [M-H]- adducts from molecular graphs) — https://github.com/zmzhang/SigmaCCS
- **RDKit** (Loads SMILES, filters by molecular weight, generates ETKDG conformers, and minimizes them with MMFF94 before graph conversion)
- **TensorFlow** (Runtime for loading and executing the pretrained SigmaCCS model)
- **Spektral** (Graph neural network library used by the SigmaCCS model architecture for graph convolution layers)

## Examples

```
from sigma.SigmaCCS import Model_prediction; ccs = Model_prediction(smiles, adduct='[M+H]+')
```

## Evaluation signals

- Output CSV exists with all expected fields (Pubchem ID, SMILES, Molecular Weight, Formula, three CCS columns) populated for every sampled compound
- Predicted CCS values on compounds with known experimental CCS fall within the reported accuracy: median relative error ~1.1751% and R2 ~0.9945 on the held-out test set
- Every input compound passes the MW < 1500 Da filter and contains only the 13 supported elements before conformer generation
- Compounds whose ETKDG conformer generation or MMFF94 minimization fails are tracked and excluded, as in the paper (33 unrecoverable PubChem molecules; 662 non-minimizable conformers)

## Limitations

- SigmaCCS cannot predict molecules containing elements other than C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se
- Only three adduct ion types are encoded; additional adduct types are noted as a future improvement
- ETKG could not generate conformers for 33 of 94,161,896 PubChem molecules, and 662 ETKDG conformers could not be minimized by MMFF94

## Evidence

- [abstract] MW filter and three-adduct prediction: "An in-silico database with 282 million CCS values was generated for three different adduct types of 94 million compounds"
- [readme] adduct set and output content: [evidence span withheld — anchor status: not_found]
- [methods] conformer generation and minimization code: "re = AllChem.EmbedMultipleConfs(mol, numConfs = 1, params = ps)"
- [methods] MMFF minimization step: "re = AllChem.MMFFOptimizeMoleculeConfs(mol, numThreads = 0)"
- [abstract] model accuracy benchmark: "It achieved a coefficient of determination of 0.9945 and a median relative error of 1.1751% on the test set"
- [discussion] element coverage limitation: "Although SigmaCCS cannot predict molecules of elements other than C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se"
