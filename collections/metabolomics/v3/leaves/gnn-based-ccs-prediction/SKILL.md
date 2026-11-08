---
name: gnn-based-ccs-prediction
description: Use when LC-MS compound identification needs CCS values predicted from
  SMILES/molecular structures, or when building an in-silico CCS database for molecules
  in PubChem under the three adduct types.
when_to_use_negative:
- Molecules containing elements other than C, H, O, N, P, S, F, Cl, Br, I, Co, As,
  and Se — SigmaCCS cannot predict these
- Adduct ion types beyond [M+H]+, [M+Na]+, and [M-H]- — only these three adducts are
  encoded
- When a descriptor-based or ensemble method is required instead — alternatives like
  AllCCS (SVR on descriptors) or CCSbase (K-Means clustering) follow different assumptions
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: SigmaCCS
  role: Graph neural network model (ECC layers + fully connected layers) that predicts
    CCS from 3D molecular graphs and adduct encoding
  repo: https://github.com/zmzhang/SigmaCCS
- name: RDKit
  role: Generates 3D conformers with ETKDG (EmbedMultipleConfs, numConfs=1) and minimizes
    them with MMFF (MMFFOptimizeMoleculeConfs)
  repo: https://rdkit.org/
- name: TensorFlow
  role: Backend framework used to load and run the pretrained model.h5 for CCS prediction
  repo: https://www.tensorflow.org
- name: Spektral
  role: Graph neural network library used to build and evaluate the ECC-based molecular
    graph model
  repo: https://graphneural.network/
- name: GNN-RT
  role: Related GNN approach from the same group used for LC retention time prediction,
    cited alongside SigmaCCS
  repo: https://github.com/Qiong-Yang/GNN-RT
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42004-023-00939-w
    title: Highly accurate and large-scale collision cross sections prediction with
      graph neural networks
  source_content_hash: sha256:f21302631fe3dad642421c8edfd84c38254587cf2075ea6d7e758ea59a88466a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmzhang/SigmaCCS
  tools:
  - GNN-RT
  - RDKit
  - SigmaCCS
  - Spektral
  - TensorFlow
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# gnn-based-ccs-prediction

## Summary

Predict collision cross section (CCS) values of small molecules end-to-end with the SigmaCCS graph neural network, which consumes RDKit-generated 3D conformers converted to molecular graphs plus one-hot adduct encoding. It supports [M+H]+, [M+Na]+ and [M-H]- adducts and enables large-scale in-silico CCS database construction (Zenodo record 5501674).

## When to use

Use when LC-MS compound identification needs CCS values predicted from SMILES/molecular structures, or when building an in-silico CCS database for molecules in PubChem under the three adduct types.

## When NOT to use

- Molecules containing elements other than C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se — SigmaCCS cannot predict these
- Adduct ion types beyond [M+H]+, [M+Na]+, and [M-H]- — only these three adducts are encoded
- When a descriptor-based or ensemble method is required instead — alternatives like AllCCS (SVR on descriptors) or CCSbase (K-Means clustering) follow different assumptions

## Inputs

- SMILES strings or a SMILES list of query compounds (element-restricted)
- Pretrained SigmaCCS model file model.h5
- parameter.pkl with saved adduct set, atoms set, and coordinate min/max
- TrainData.csv / TestData.csv for retraining or evaluation

## Outputs

- Predicted CCS values (Å²) for each molecule under [M+H]+, [M+Na]+, and [M-H]-
- CSV file (e.g. sigma_ccs_predictions.csv) with Pubchem ID, SMILES, Molecular Weight, Formula, and predicted CCS columns

## How to apply

Load molecules from a SMILES list with RDKit (restricting to elements C, H, O, N, P, S, F, Cl, Br, I, Co, As, Se and MW < 1500 Da for database generation), then generate one 3D conformer per molecule with ETKDGv3 (AllChem.EmbedMultipleConfs, numConfs=1, useRandomCoords=True) and minimize it with MMFF (MMFFOptimizeMoleculeConfs, numThreads=0). Convert each conformer into node feature, adjacency, and edge feature matrices via the convertToGraph routine in GraphData.py, using the saved parameters (adduct set, atoms set, coordinate min/max) from parameter.pkl. Feed the graph plus a one-hot adduct vector ([M+H]+, [M+Na]+, or [M-H]-) into the pretrained SigmaCCS model (model.h5, TensorFlow/Spektral) with the Model_prediction function to obtain CCS values. Write results to CSV with fields Pubchem ID, SMILES, Molecular Weight, Formula, and predicted CCS values, mirroring the Zenodo (record 5501674) dataset format.

## Related tools

- **SigmaCCS** (Graph neural network model (ECC layers + fully connected layers) that predicts CCS from 3D molecular graphs and adduct encoding) — https://github.com/zmzhang/SigmaCCS
- **RDKit** (Generates 3D conformers with ETKDG (EmbedMultipleConfs, numConfs=1) and minimizes them with MMFF (MMFFOptimizeMoleculeConfs)) — https://rdkit.org/
- **TensorFlow** (Backend framework used to load and run the pretrained model.h5 for CCS prediction) — https://www.tensorflow.org
- **Spektral** (Graph neural network library used to build and evaluate the ECC-based molecular graph model) — https://graphneural.network/
- **GNN-RT** (Related GNN approach from the same group used for LC retention time prediction, cited alongside SigmaCCS) — https://github.com/Qiong-Yang/GNN-RT

## Examples

```
from SigmaCCS import Model_prediction; ccs = Model_prediction(model, convertToGraph(smiles, coord, All_Atoms), adduct='[M+H]+')
```

## Evaluation signals

- Predicted CCS on a small set of compounds with known experimental CCS falls within the reported model accuracy (median relative error ~1.1751%, R2 ~0.9945 on the held-out test set of >5,000 experimental CCS values)
- Output CSV contains all expected fields (Pubchem ID, SMILES, Molecular Weight, Formula, three adduct CCS values) populated for every sampled compound
- Conformer generation succeeds via ETKDG for essentially all inputs — only 33 of 94,161,896 PubChem molecules and 5 of 5,645 CCSbase entries failed, so systematic failures indicate malformed SMILES or unsupported elements
- MMFF minimization completes for nearly all conformers (only 662 of ~94M PubChem conformers failed minimization); recurring minimization failures suggest problematic structures
- Predicted CCS values are in a physically plausible range for the molecular weight and adduct type (CCS grows with molecular size)

## Limitations

- SigmaCCS cannot predict molecules containing elements other than C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se
- Only three adduct ion types ([M+H]+, [M+Na]+, [M-H]-) are encoded; encoding more adduct types is noted as future improvement
- ETKDGV conformer generation failed for 33 of 94,161,896 PubChem molecules and 5 of 5,645 CCSbase entries, and 662 ETKDG conformers could not be minimized by MMFF94
- Preprocessing requires element-restricted filtering (MW < 1500 Da for the database pipeline) and correct parameter.pkl settings; incorrect atom sets silently drop unsupported elements

## Evidence

- [abstract] GNN with 3D conformers: "we have developed the Structure included graph merging with adduct method for CCS prediction (SigmaCCS) based on graph neural networks using 3D conformers as inputs"
- [abstract] test-set accuracy: "It achieved a coefficient of determination of 0.9945 and a median relative error of 1.1751% on the test set"
- [intro] in-silico database scale: "An in-silico database with 282 million CCS values was generated for three different adduct types of 94 million compounds"
- [discussion] ETKDG failures: "the conformers of some molecules (33 out 94,161,896) in PubChem and (5 out 5645 entries) in CCSbase cannot be generated by the ETKDG method"
- [methods] prediction workflow: "The CCS prediction of the molecule is obtained by inputting Graph and Adduct into the already trained SigmaCCS model."
- [readme] dataset fields: "For each molecule, there are "Pubchem ID", "SMILES", "InChi", "Inchikey", "Molecular Weight", "Exact Mass", "Formula" and predicted CCS values of three adduct ion types."
- [readme] RDKit parameters: "re = AllChem.EmbedMultipleConfs(mol, numConfs = 1, params = ps)"
