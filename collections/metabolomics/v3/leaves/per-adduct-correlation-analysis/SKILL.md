---
name: per-adduct-correlation-analysis
description: Use when you have SigmaCCS-predicted CCS values for [M+H]+, [M+Na]+,
  or [M-H]- adducts and an independently calculated CCS database (e.g., ISiCLE/PNNL)
  and need to quantify per-adduct agreement.
when_to_use_negative:
- Molecules contain elements outside C, H, O, N, P, S, F, Cl, Br, I, Co, As, Se —
  SigmaCCS cannot predict them
- Adduct types other than [M+H]+, [M+Na]+, [M-H]- are needed — SigmaCCS encodes only
  these three ion types
- You need experimental agreement rather than theoretical-to-predicted correlation
  — use the CCSbase experimental test set evaluation instead
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0091
tools:
- name: SigmaCCS
  role: GNN model whose convertToGraph/MyDataset and Model_prediction (model.h5, parameter.pkl)
    produce the CCS predictions being correlated
  repo: https://github.com/zmzhang/SigmaCCS
- name: ISiCLE
  role: Provides the independently calculated (TM-method) reference CCS values (PNNL
    database v0.1.0) used as the correlation target
- name: RDKit
  role: Generates and MMFF-optimizes the 3D conformers (ETKDGv3, numConfs=1) that
    SigmaCCS converts to molecular graphs
- name: TensorFlow
  role: Runtime backend (2.4.0 per README) for loading and running the pretrained
    SigmaCCS model
  repo: https://github.com/zmzhang/SigmaCCS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42004-023-00939-w
    title: Highly accurate and large-scale collision cross sections prediction with
      graph neural networks
  source_content_hash: sha256:cfd9b595c8deb6371f61129b9a984ae86e2b6b9c390b8238cec07077692ca9c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmzhang/SigmaCCS
  tools:
  - ISiCLE
  - RDKit
  - SigmaCCS
  - TensorFlow
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# per-adduct-correlation-analysis

## Summary

Validates SigmaCCS graph-neural-network CCS predictions by joining them per adduct to ISiCLE-calculated (PNNL/TM-method) reference CCS values and computing per-adduct Pearson correlation coefficients.

## When to use

Use when you have SigmaCCS-predicted CCS values for [M+H]+, [M+Na]+, or [M-H]- adducts and an independently calculated CCS database (e.g., ISiCLE/PNNL) and need to quantify per-adduct agreement.

## When NOT to use

- Molecules contain elements outside C, H, O, N, P, S, F, Cl, Br, I, Co, As, Se — SigmaCCS cannot predict them
- Adduct types other than [M+H]+, [M+Na]+, [M-H]- are needed — SigmaCCS encodes only these three ion types
- You need experimental agreement rather than theoretical-to-predicted correlation — use the CCSbase experimental test set evaluation instead

## Inputs

- Compound SMILES list with adduct annotations
- ISiCLE-calculated (TM-method) PNNL CCS database, v0.1.0, 825,702 entries
- Pretrained SigmaCCS model.h5 and parameter.pkl
- SigmaCCS TrainData.csv / TestData.csv format reference tables

## Outputs

- Per-adduct Pearson correlation coefficients (r) for [M+H]+, [M+Na]+, and [M-H]-
- correlation_summary.csv joining predicted and reference CCS values per adduct

## How to apply

Load the compound SMILES/adduct lists and the ISiCLE-calculated CCS database (PNNL database, v0.1.0, 825,702 entries), matching entries on compound identifier and adduct type. For each SMILES, generate a 3D conformer with RDKit ETKDGv3 (numConfs=1, randomSeed=-1, maxAttempts=1, useRandomCoords=True) and optimize with MMFFOptimizeMoleculeConfs (numThreads=0). Convert each conformer to a molecular graph via SigmaCCS convertToGraph/MyDataset (node feature matrix, adjacency matrix, edge feature matrix), then run SigmaCCS Model_prediction with the pretrained model.h5 and parameter.pkl to obtain predicted CCS values per adduct. Join predictions to the PNNL reference values per adduct with pandas/numpy and compute the Pearson correlation coefficient separately for [M+H]+, [M+Na]+, and [M-H]-, because agreement can differ by ionization adduct. Accept the reproduction when per-adduct r values match the reported 0.9572 ([M+H]+), 0.9586 ([M+Na]+), and 0.9546 ([M-H]-) within ±0.005 in correlation_summary.csv.

## Related tools

- **SigmaCCS** (GNN model whose convertToGraph/MyDataset and Model_prediction (model.h5, parameter.pkl) produce the CCS predictions being correlated) — https://github.com/zmzhang/SigmaCCS
- **ISiCLE** (Provides the independently calculated (TM-method) reference CCS values (PNNL database v0.1.0) used as the correlation target)
- **RDKit** (Generates and MMFF-optimizes the 3D conformers (ETKDGv3, numConfs=1) that SigmaCCS converts to molecular graphs)
- **TensorFlow** (Runtime backend (2.4.0 per README) for loading and running the pretrained SigmaCCS model) — https://github.com/zmzhang/SigmaCCS

## Examples

```
re = AllChem.EmbedMultipleConfs(mol, numConfs = 1, params = ps); re = AllChem.MMFFOptimizeMoleculeConfs(mol, numThreads = 0); adj, features, edge_features = convertToGraph(smiles, Coordinate, All_Atoms)
```

## Evaluation signals

- Pearson r for [M+H]+ within 0.9572 ± 0.005
- Pearson r for [M+Na]+ within 0.9586 ± 0.005
- Pearson r for [M-H]- within 0.9546 ± 0.005
- correlation_summary.csv produced with joined predicted and reference CCS rows keyed by compound identifier and adduct type
- All input SMILES yield conformers via ETKDGv3 and MMFF minimization; count and exclude conformer-generation/minimization failures

## Limitations

- SigmaCCS cannot predict molecules containing elements other than C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se
- SigmaCCS handles only three adduct ion types; other adducts are out of scope
- ISiCLE requires complex computational steps with high demand on computational resources
- ETKDG conformer generation failed for a small fraction of molecules (33/94,161,896 PubChem; 5/5,645 CCSbase), and MMFF94 minimization failed for 662 conformers

## Evidence

- [methods] Model prediction with pretrained model and per-adduct Pearson r thresholds: [evidence span withheld — anchor status: not_found]
- [methods] Conformer generation parameters: [evidence span withheld — anchor status: not_found]
- [results] Reported correlation values: [evidence span withheld — anchor status: not_found]
- [readme] Graph construction step: "SigmaCCS is a model for predicting CCS based on graph neural networks, so we need to convert SMILES strings to Graph."
- [discussion] Element coverage limitation: "Although SigmaCCS cannot predict molecules of elements other than C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se"
- [discussion] Adduct coverage limitation: "Further improvement can be made by encoding more adduct ion types in the one-hot vector."
