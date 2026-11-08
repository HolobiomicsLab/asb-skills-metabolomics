---
name: ccs-based-compound-identification
description: Use when a metabolomics/lipidomics query set has measured precursor m/z
  (and optionally RT) and you must narrow a large candidate library (e.g. LipidBlast)
  using CCS from a 3D-conformer GNN before identification.
when_to_use_negative:
- The candidate molecule contains elements outside C, H, O, N, P, S, F, Cl, Br, I,
  Co, As, Se — SigmaCCS cannot predict those structures
- The adduct type is not one of [M+H]+, [M+Na]+, [M-H]- — SigmaCCS only encodes these
  three ion types
- 3D conformer generation/minimization fails (ETKDG failed for a small number of PubChem/CCSbase
  molecules and MMFF94 minimization failed for 662 molecules), so no CCS prediction
  is possible
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: SigmaCCS
  role: Predicts CCS values from 3D-conformer molecular graphs via a graph neural
    network, used for the Stage 3 m/z+RT+CCS candidate filtering
  repo: https://github.com/zmzhang/SigmaCCS
- name: GNN-RT
  role: Predicts LC retention times from molecular graphs, used for the Stage 2 m/z+RT
    candidate re-ranking
  repo: https://github.com/Qiong-Yang/GNN-RT
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42004-023-00939-w
    title: Highly accurate and large-scale collision cross sections prediction with
      graph neural networks
  source_content_hash: sha256:8962e5e45850e2ab3f92595842d468495bbac1160cf4df2d178a939317e762d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmzhang/SigmaCCS
  tools:
  - GNN-RT
  - SigmaCCS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# ccs-based-compound-identification

## Summary

Improves LC-MS compound identification by using SigmaCCS-predicted collision cross sections (CCS), together with precursor m/z and GNN-RT-predicted retention time, as orthogonal filtering dimensions to re-rank candidate lipid annotations and raise Recall@1/Recall@30.

## When to use

Use when a metabolomics/lipidomics query set has measured precursor m/z (and optionally RT) and you must narrow a large candidate library (e.g. LipidBlast) using CCS from a 3D-conformer GNN before identification.

## When NOT to use

- The candidate molecule contains elements outside C, H, O, N, P, S, F, Cl, Br, I, Co, As, Se — SigmaCCS cannot predict those structures
- The adduct type is not one of [M+H]+, [M+Na]+, [M-H]- — SigmaCCS only encodes these three ion types
- 3D conformer generation/minimization fails (ETKDG failed for a small number of PubChem/CCSbase molecules and MMFF94 minimization failed for 662 molecules), so no CCS prediction is possible

## Inputs

- Query table of 761 negative-mode mouse lung lipid features with measured precursor m/z and retention times (pandas-readable CSV)
- LipidBlast candidate annotation library with SMILES for each candidate
- Trained SigmaCCS model (model.h5) with parameter.pkl and GraphData.py graph conversion utilities
- Trained GNN-RT retention time prediction model
- Adduct specification (e.g. [M-H]-; SigmaCCS encodes [M+H]+, [M+Na]+, [M-H]-)

## Outputs

- Per-query ranked candidate lists after each filtering stage (m/z, m/z+RT, m/z+RT+CCS)
- Recall@1 and Recall@30 metrics per stage (e.g. 15.2%/24.6%/28.9% and 47.6%/78.4%/91.2%)
- Per-stage recall metrics written to recall_results.csv
- SigmaCCS-predicted CCS values per candidate per adduct

## How to apply

Run the staged filtering chain on each query: Stage 1 screens candidate annotations by precursor m/z matching alone; Stage 2 re-ranks the m/z-matched candidates with predicted retention times from the trained GNN-RT model; Stage 3 re-ranks again using SigmaCCS-predicted CCS values computed for the specific adduct (e.g. [M-H]-). For SigmaCCS, convert each candidate SMILES to a 3D conformer via RDKit ETKDGv3 embedding (EmbedMultipleConfs with numConfs=1) and MMFF optimization, build the node/adjacency/edge feature matrices with convertToGraph, then input the Graph plus adduct into the trained model. Evaluate each stage by computing Recall@1 and Recall@30 against the known correct molecules and compare with reference metrics (in the mouse lung dataset of 761 negative-mode lipids, Recall@1 rose 15.2%→24.6%→28.9% and Recall@30 rose 47.6%→78.4%→91.2%). The rationale is that CCS is a stable, structure-informed physicochemical property that supplies information orthogonal to m/z and RT, so adding it monotonically improves correct-molecule ranking.

## Related tools

- **SigmaCCS** (Predicts CCS values from 3D-conformer molecular graphs via a graph neural network, used for the Stage 3 m/z+RT+CCS candidate filtering) — https://github.com/zmzhang/SigmaCCS
- **GNN-RT** (Predicts LC retention times from molecular graphs, used for the Stage 2 m/z+RT candidate re-ranking) — https://github.com/Qiong-Yang/GNN-RT

## Examples

```
adj, features, edge_features = convertToGraph(smiles, Coordinate, All_Atoms); ccs = sigma_predict(adj, features, edge_features, adduct='[M-H]-')
```

## Evaluation signals

- Recall@1 and Recall@30 after each stage match Supplementary Table 12 within a small tolerance (e.g. ~15.2%/24.6%/28.9% Recall@1; ~47.6%/78.4%/91.2% Recall@30)
- Recall@1 and Recall@30 are monotonically non-decreasing across the m/z → m/z+RT → m/z+RT+CCS stages
- Every candidate sent to SigmaCCS has a valid 3D conformer (ETKDGv3, numConfs=1) that was successfully MMFF-optimized
- recall_results.csv contains exactly three rows/columns of metrics, one per filtering stage, over all 761 lipids
- CCS predictions are requested for the correct adduct ([M-H]- for the negative-mode mouse lung dataset)

## Limitations

- SigmaCCS cannot predict molecules containing elements other than C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se
- SigmaCCS covers only three adduct ion types; encoding more adduct types is noted as future improvement
- ETKDG conformer generation failed for 33/94,161,896 PubChem molecules and 5/5,645 CCSbase entries, and MMFF94 minimization failed for 662 molecules
- The SigmaCCS model was trained/evaluated with >5,000 experimental CCS values (R^2 0.9945, median relative error 1.1751%), so accuracy may degrade outside the training chemical space

## Evidence

- [abstract] SigmaCCS GNN-based CCS prediction: "we have developed the Structure included graph merging with adduct method for CCS prediction (SigmaCCS) based on graph neural networks using 3D conformers as inputs"
- [methods] Filtering notebook for m/z+CCS candidate filtering: "Filtering of target unknown molecules based on the CCS and *mz* of the molecules"
- [readme] SigmaCCS prediction procedure: "The CCS prediction of the molecule is obtained by inputting Graph and Adduct into the already trained SigmaCCS model."
- [discussion] Adduct coverage limitation: "Further improvement can be made by encoding more adduct ion types in the one-hot vector."
- [methods] Graph construction from conformer: "Generate the three matrices used to construct the Graph"
