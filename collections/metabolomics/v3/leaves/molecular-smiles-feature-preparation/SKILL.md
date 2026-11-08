---
name: molecular-smiles-feature-preparation
description: Use when raw SMILES strings and adduct labels ([M+H]+, [M+Na]+, [M-H]-)
  must be transformed into graph representations (node, adjacency, edge matrices or
  adduct graphs) with 3D coordinates before training or evaluating GNN CCS prediction
  models.
when_to_use_negative:
- Input is already a featurized graph dataset (MyDataset/DGL graphs) — re-running
  conformer generation wastes compute and may change predictions via different random
  conformers
- The goal is cross-database generalization testing on already-trained models — only
  the prepared test parquet is needed, not re-featurization logic
- Molecules cannot be embedded or lack valid SMILES; ETKDGv3 embedding failures will
  silently degrade coverage
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_2258
tools:
- name: RDKit
  role: Parses SMILES, adds hydrogens, generates 3D conformers (ETKDGv3) and MMFF-optimizes
    them; supplies node/edge featurization inputs
- name: SigmaCCS
  role: Defines the SMILES-to-graph conversion (GraphData.py, convertToGraph) producing
    node/adjacency/edge matrices and the parameter files for coordinate normalization
  repo: https://github.com/zmzhang/SigmaCCS
- name: GraphCCS
  role: Provides edit_adduct_mol and DGL featurization to build adduct-aware graphs
    with explicit hydrogens from SMILES + adduct type
  repo: https://github.com/tingxiecsu/GraphCCS
- name: Mol2CCS
  role: Consumes csv/excel formatted per its input spec, extending SigmaCCS features
    with fingerprints, descriptors, and molecule/instrument type
  repo: https://github.com/enveda/ccs-prediction
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-024-00899-w
    title: Evaluating the generalizability of graph neural networks for predicting
      collision cross section
  source_content_hash: sha256:cb68c380e8a531624f9bfcedc78825e13c81d3d9baa27d0c5971c8883039c234
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmzhang/SigmaCCS
  tools:
  - GraphCCS
  - Mol2CCS
  - Python
  - RDKit
  - SigmaCCS
  - poetry
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# molecular-smiles-feature-preparation

## Summary

Converts SMILES strings plus adduct annotations into GNN-ready molecular graph inputs — 3D conformers (ETKDGv3/MMFF via RDKit v2023.09.5), node/adjacency/edge feature matrices or DGL adduct graphs — for CCS prediction models such as SigmaCCS, GraphCCS, and Mol2CCS.

## When to use

Use when raw SMILES strings and adduct labels ([M+H]+, [M+Na]+, [M-H]-) must be transformed into graph representations (node, adjacency, edge matrices or adduct graphs) with 3D coordinates before training or evaluating GNN CCS prediction models.

## When NOT to use

- Input is already a featurized graph dataset (MyDataset/DGL graphs) — re-running conformer generation wastes compute and may change predictions via different random conformers
- The goal is cross-database generalization testing on already-trained models — only the prepared test parquet is needed, not re-featurization logic
- Molecules cannot be embedded or lack valid SMILES; ETKDGv3 embedding failures will silently degrade coverage

## Inputs

- SMILES strings of small molecules (csv/excel or parquet)
- Adduct type labels per molecule ([M+H]+, [M+Na]+, [M-H]-)
- Experimental CCS values column (ccs) for supervised training
- 80/20 train/test split parquet files (e.g. metlin_train_3d.parquet, ccsbase_3d.parquet)
- Optional precomputed 3D coordinates column (coordinates) with coordinates-present flag

## Outputs

- Node feature matrix, adjacency matrix, and edge feature matrix per molecule (SigmaCCS MyDataset)
- DGL adduct graphs with explicit hydrogens (GraphCCS graph_calculation/DataLoader)
- 3D coordinate arrays normalized against saved min/max bounds
- Parameter file storing atom set, adduct set, and coordinate min/max (parameter-train-metlin-test-metlin.json)

## How to apply

Read each SMILES with RDKit (v2023.09.5), add hydrogens via Chem.AddHs, embed a single 3D conformer with AllChem.ETKDGv3 (useRandomCoords=True, maxAttempts=1) and optimize it with MMFFOptimizeMoleculeConfs. Persist the atom set, adduct set, and the min/max coordinate bounds (as in SigmaCCS's sigma/parameter.py) so coordinates can be normalized consistently across train/test splits. Then build the three graph tensors — node feature matrix, adjacency matrix, and edge feature matrix via convertToGraph — or, for GraphCCS, build an 'Adduct Graph' by applying edit_adduct_mol(mol, add) before DGL featurization with explicit hydrogens. For Mol2CCS, additionally format the csv/excel input to carry molecular fingerprints, descriptors, and instrument type alongside smiles/adduct/ccs columns, matching the output format of notebooks/data_processing/2_data_splits.ipynb parquet splits (e.g. metlin_train_3d.parquet, ccsbase_3d.parquet).

## Related tools

- **RDKit** (Parses SMILES, adds hydrogens, generates 3D conformers (ETKDGv3) and MMFF-optimizes them; supplies node/edge featurization inputs)
- **SigmaCCS** (Defines the SMILES-to-graph conversion (GraphData.py, convertToGraph) producing node/adjacency/edge matrices and the parameter files for coordinate normalization) — https://github.com/zmzhang/SigmaCCS
- **GraphCCS** (Provides edit_adduct_mol and DGL featurization to build adduct-aware graphs with explicit hydrogens from SMILES + adduct type) — https://github.com/tingxiecsu/GraphCCS
- **Mol2CCS** (Consumes csv/excel formatted per its input spec, extending SigmaCCS features with fingerprints, descriptors, and molecule/instrument type) — https://github.com/enveda/ccs-prediction

## Examples

```
mol = Chem.MolFromSmiles(smiles); mol = Chem.AddHs(mol); ps = AllChem.ETKDGv3(); ps.useRandomCoords = True; AllChem.EmbedMultipleConfs(mol, numConfs=1, params=ps); AllChem.MMFFOptimizeMoleculeConfs(mol)
```

## Evaluation signals

- Each molecule yields exactly three aligned tensors — node feature matrix, adjacency matrix, edge feature matrix — with atom counts matching the hydrogen-added RDKit Mol object
- 3D conformer embedding via ETKDGv3 succeeds (EmbedMultipleConfs returns a result) and MMFF optimization is applied before coordinate extraction
- Coordinates are min/max normalized using the persisted parameter file (atom set, adduct set, coordinate min/max) so train and test splits share the same scaling
- Adduct labels fall within the defined adduct set ([M+H]+, [M+Na]+, [M-H]-) and are encoded as an input feature before prediction
- Featurized datasets reproduce the schema expected by 2_data_splits.ipynb output parquet files (smiles, adduct, ccs, coordinates columns)

## Limitations

- Conformer generation depends on ETKDG random seed and force field choice — performance differs between ETKDG/MMFF94-generated and randomly rotated coordinates (per SigmaCCS's 'Performance on the test set with large rotations' notebooks)
- Variations in experimental conditions such as instrument types likely introduced inaccuracies in the CCS values used to train models, affecting downstream feature quality
- Evaluation scope is restricted mainly to small molecules and specific regions of chemical space (METLIN-CCS is based on synthetic structures), so featurization behavior on larger or novel scaffolds is untested

## Evidence

- [readme] SigmaCCS SMILES-to-graph conversion: [evidence span withheld — anchor status: external_doc]
- [readme] Graph tensor construction: [evidence span withheld — anchor status: external_doc]
- [readme] GraphCCS adduct graph featurization: [evidence span withheld — anchor status: external_doc]
- [readme] RDKit conformer pipeline: [evidence span withheld — anchor status: external_doc]
- [methods] Mol2CCS input formatting: "Each notebook reads the csv/excel and formats it according to the input of Mol2CCS."
- [intro] RDKit version used in study: "we first read the SMILES representation of each molecule using RDKit (v2023.09.5)"
