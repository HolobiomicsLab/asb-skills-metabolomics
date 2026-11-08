---
name: precursor-mass-candidate-screening
description: Use when a set of LC-MS precursor queries (measured m/z, optional RT)
  needs candidate structures from a database narrowed down, and orthogonal predicted
  properties (RT, CCS) are available to re-rank candidates.
when_to_use_negative:
- Molecules contain elements outside C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se,
  since SigmaCCS cannot predict them
- The adduct is not one of the three encoded ion types ([M+H]+, [M+Na]+, [M-H]-),
  because SigmaCCS handles only these three adduct ion types
- Input is already a single confident identification per query with no candidate ambiguity,
  so multi-stage candidate re-ranking adds no value
edam_operation: http://edamontology.org/operation_3741
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: SigmaCCS
  role: Predicts CCS values from 3D-conformer molecular graphs for candidate re-ranking
    in the m/z+RT+CCS stage
  repo: https://github.com/zmzhang/SigmaCCS
- name: GNN-RT
  role: Predicts LC retention times used to re-rank m/z-matched candidates in the
    m/z+RT stage
  repo: https://github.com/Qiong-Yang/GNN-RT
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42004-023-00939-w
    title: Highly accurate and large-scale collision cross sections prediction with
      graph neural networks
  source_content_hash: sha256:2326ab1412e37b058cafa69716aa4622e21e4f721d36b6cb3ebaec719789c79c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmzhang/SigmaCCS
  tools:
  - GNN-RT
  - SigmaCCS
  edam_operation: http://edamontology.org/operation_3741
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# precursor-mass-candidate-screening

## Summary

A candidate-filtering workflow that screens lipid/small-molecule identification candidates by precursor m/z matching and then progressively re-ranks them with predicted retention time (GNN-RT) and predicted CCS (SigmaCCS), improving correct-identification recall in LC-MS lipidomics.

## When to use

Use when a set of LC-MS precursor queries (measured m/z, optional RT) needs candidate structures from a database narrowed down, and orthogonal predicted properties (RT, CCS) are available to re-rank candidates.

## When NOT to use

- Molecules contain elements outside C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se, since SigmaCCS cannot predict them
- The adduct is not one of the three encoded ion types ([M+H]+, [M+Na]+, [M-H]-), because SigmaCCS handles only these three adduct ion types
- Input is already a single confident identification per query with no candidate ambiguity, so multi-stage candidate re-ranking adds no value

## Inputs

- Query table of LC-MS precursor lipids with measured m/z and retention times (e.g. 761 negative-mode mouse lung lipid queries)
- Candidate annotation database (e.g. LipidBlast) with SMILES, precursor m/z, and formula per entry
- Trained GNN-RT model for retention-time prediction
- Trained SigmaCCS model (model.h5) and parameters (parameter.pkl) for CCS prediction

## Outputs

- Per-query ranked candidate lists after each filtering stage (m/z, m/z+RT, m/z+RT+CCS)
- Recall@1 and Recall@30 metrics per stage, written to recall_results.csv

## How to apply

Load the query table (e.g. 761 negative-mode mouse lung lipid queries with measured m/z and retention times) with pandas. Stage 1 screens database candidates (e.g. LipidBlast annotations) by precursor m/z matching to produce per-query ranked candidate lists. Stage 2 re-ranks the m/z-matched candidates by applying retention times predicted with the GNN-RT model. Stage 3 further re-ranks by applying SigmaCCS-predicted CCS values for the [M-H]- adduct: convert each candidate SMILES to a 3D-conformer graph (ETKDGv3 embedding with numConfs=1, MMFF optimization), run convertToGraph to build the node feature, adjacency, and edge feature matrices, and feed graph plus adduct into the trained SigmaCCS model. After each stage compute Recall@1 and Recall@30 over all queries (reported values: Recall@1 rising 15.2% → 24.6% → 28.9%; Recall@30 rising 47.6% → 78.4% → 91.2%) and write per-stage metrics to recall_results.csv.

## Related tools

- **SigmaCCS** (Predicts CCS values from 3D-conformer molecular graphs for candidate re-ranking in the m/z+RT+CCS stage) — https://github.com/zmzhang/SigmaCCS
- **GNN-RT** (Predicts LC retention times used to re-rank m/z-matched candidates in the m/z+RT stage) — https://github.com/Qiong-Yang/GNN-RT

## Examples

```
re = AllChem.EmbedMultipleConfs(mol, numConfs=1, params=ps); AllChem.MMFFOptimizeMoleculeConfs(mol, numThreads=0); adj, features, edge_features = convertToGraph(smiles, Coordinate, All_Atoms)
```

## Evaluation signals

- Recall@1 over the 761 queries should increase monotonically across stages (~15.2% → ~24.6% → ~28.9%)
- Recall@30 should increase monotonically across stages (~47.6% → ~78.4% → ~91.2%)
- Reported values should match Supplementary Table 12 within a small tolerance
- Every query should retain a non-empty candidate list after m/z screening; stage 2 and 3 must only re-rank, not discard, beyond the article's filtering rules
- CCS predictions must succeed for each candidate SMILES; failures traceable to ETKDG conformer generation or MMFF94 minimization should be logged (662 molecules in the paper's data could not be MMFF-minimized)

## Limitations

- SigmaCCS cannot predict molecules containing elements other than C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se
- SigmaCCS handles only three adduct ion types ([M+H]+, [M+Na]+, [M-H]-); more adducts require retraining or future work
- ETKDG could not generate conformers for 33/94,161,896 PubChem molecules and 5/5,645 CCSbase entries, and 662 ETKDG conformers could not be minimized by MMFF94 — such candidates must be dropped or flagged
- CCS values are predictions; filtering thresholds inherited from prediction error (median relative error ~1.1751% on the test set) should be validated against experimental CCS

## Evidence

- [abstract] we have developed the Structure included graph merging with adduct method for CCS prediction (SigmaCCS) based on graph neural networks using 3D conformers as inputs: "we have developed the Structure included graph merging with adduct method for CCS prediction (SigmaCCS) based on graph neural networks using 3D conformers as inputs"
- [methods] Filtering of target unknown molecules based on the CCS and mz of the molecules: "[Filtering.ipynb](others/Filtering.ipynb). Filtering of target unknown molecules based on the CCS and *mz* of the molecules"
- [readme] The CCS prediction of the molecule is obtained by inputting Graph and Adduct into the already trained SigmaCCS model.: "The CCS prediction of the molecule is obtained by inputting Graph and Adduct into the already trained SigmaCCS model."
- [methods] re = AllChem.EmbedMultipleConfs(mol, numConfs = 1, params = ps): "re = AllChem.EmbedMultipleConfs(mol, numConfs = 1, params = ps)"
- [discussion] thus SigmaCCS focuses on the three adduct ion types: "thus SigmaCCS focuses on the three adduct ion types"
