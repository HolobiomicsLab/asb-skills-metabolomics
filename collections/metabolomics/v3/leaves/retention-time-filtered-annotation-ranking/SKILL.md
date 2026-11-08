---
name: retention-time-filtered-annotation-ranking
description: Use when you have m/z-matched candidate annotation lists (e.g. from LipidBlast
  or similar) plus measured retention times, and you need to improve Recall@1/Recall@k
  of the correct identification by adding orthogonal RT or CCS evidence.
when_to_use_negative:
- Candidates contain elements outside C, H, O, N, P, S, F, Cl, Br, I, Co, As, and
  Se, for which SigmaCCS cannot generate conformer predictions
- The query adduct is not one of the three encoded ion types ([M+H]+, [M+Na]+, [M-H]-);
  SigmaCCS does not cover other adducts
- No measured retention times are available for the queries, so the RT stage cannot
  be validated or applied as a filter
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2258
tools:
- name: GNN-RT
  role: Predicts LC retention times from molecular graphs, used as the second-stage
    filter to re-rank m/z-matched candidates
  repo: https://github.com/Qiong-Yang/GNN-RT
- name: SigmaCCS
  role: Predicts CCS values from 3D-conformer molecular graphs and adduct ([M-H]-)
    for the third-stage CCS filter (Filtering.ipynb)
  repo: https://github.com/zmzhang/SigmaCCS
- name: RDKit
  role: Generates 3D conformers of candidate SMILES (EmbedMultipleConfs numConfs=1,
    MMFFOptimizeMoleculeConfs) for graph construction
  repo: https://rdkit.org/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42004-023-00939-w
    title: Highly accurate and large-scale collision cross sections prediction with
      graph neural networks
  source_content_hash: sha256:ec7e0f62446ab417aff872f0ec5936bf252c05387e5c1e167c8dc3debe78deb4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Qiong-Yang/GNN-RT
  tools:
  - GNN-RT
  - RDKit
  - SigmaCCS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# retention-time-filtered-annotation-ranking

## Summary

Re-ranks m/z-matched lipid/compound candidate annotations by adding GNN-RT-predicted retention times (and optionally SigmaCCS-predicted CCS values) as additional filters, improving the ranking of correct identifications in LC-MS lipidomics datasets.

## When to use

Use when you have m/z-matched candidate annotation lists (e.g. from LipidBlast or similar) plus measured retention times, and you need to improve Recall@1/Recall@k of the correct identification by adding orthogonal RT or CCS evidence.

## When NOT to use

- Candidates contain elements outside C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se, for which SigmaCCS cannot generate conformer predictions
- The query adduct is not one of the three encoded ion types ([M+H]+, [M+Na]+, [M-H]-); SigmaCCS does not cover other adducts
- No measured retention times are available for the queries, so the RT stage cannot be validated or applied as a filter

## Inputs

- Lipid/compound query table with measured precursor m/z and retention times (e.g. 761 negative-mode mouse lung lipids, loaded via pandas)
- Candidate annotation lists from a spectral/structure database such as LipidBlast
- Candidate SMILES strings (for RT and CCS prediction)
- Trained GNN-RT model and trained SigmaCCS model (model.h5, parameter.pkl)

## Outputs

- Per-query re-ranked candidate lists after each filtering stage (m/z; m/z+RT; m/z+RT+CCS)
- Recall@1 and Recall@30 metrics per stage (e.g. recall_results.csv)
- SigmaCCS-predicted CCS values for the [M-H]- adduct of each candidate

## How to apply

Load the query list (e.g. 761 negative-mode mouse lung lipid queries with measured m/z and RT) with pandas. Stage 1: screen candidate annotations by precursor m/z matching and record the ranked candidate lists. Stage 2: re-rank or filter those candidates by applying retention times predicted with the GNN-RT model, which takes the molecular graph as input and the predicted RT as output. Stage 3 (optional, CCS chain): further re-rank by applying SigmaCCS-predicted CCS for the [M-H]- adduct, converting candidate SMILES to 3D-conformer graphs via ETKDG embedding (numConfs=1) with MMFF optimization, then feeding graph and adduct into the trained SigmaCCS model. Compute Recall@1 and Recall@30 after each stage; the reported chain raised Recall@1 from 15.2% to 24.6% to 28.9% and Recall@30 from 47.6% to 78.4% to 91.2%.

## Related tools

- **GNN-RT** (Predicts LC retention times from molecular graphs, used as the second-stage filter to re-rank m/z-matched candidates) — https://github.com/Qiong-Yang/GNN-RT
- **SigmaCCS** (Predicts CCS values from 3D-conformer molecular graphs and adduct ([M-H]-) for the third-stage CCS filter (Filtering.ipynb)) — https://github.com/zmzhang/SigmaCCS
- **RDKit** (Generates 3D conformers of candidate SMILES (EmbedMultipleConfs numConfs=1, MMFFOptimizeMoleculeConfs) for graph construction) — https://rdkit.org/

## Examples

```
re = AllChem.EmbedMultipleConfs(mol, numConfs=1, params=ps); AllChem.MMFFOptimizeMoleculeConfs(mol, numThreads=0)
```

## Evaluation signals

- Recall@1 and Recall@30 must increase monotonically across the three stages; expected values ~15.2%/24.6%/28.9% (Recall@1) and ~47.6%/78.4%/91.2% (Recall@30) matching Supplementary Table 12 within tolerance
- Per-stage metrics are written to recall_results.csv with one row per stage over all 761 query lipids
- ETKDGv3 embedding uses numConfs=1 followed by MMFF optimization for every candidate SMILES before graph conversion
- SigmaCCS predictions are made with the correct adduct encoding ([M-H]- for the negative-mode mouse lung dataset)

## Limitations

- SigmaCCS cannot predict molecules containing elements other than C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se
- SigmaCCS encodes only three adduct ion types ([M+H]+, [M+Na]+, [M-H]-); encoding more adduct types is noted as future work
- ETKDG conformer generation failed for 33/94,161,896 PubChem molecules and 5/5,645 CCSbase entries, and 662 ETKDG conformers could not be minimized by MMFF94

## Evidence

- [abstract] SigmaCCS prediction from 3D conformers: "we have developed the Structure included graph merging with adduct method for CCS prediction (SigmaCCS) based on graph neural networks using 3D conformers as inputs"
- [methods] CCS+mz filtering workflow: "Filtering of target unknown molecules based on the CCS and *mz* of the molecules"
- [results] Recall improvement across stages: [evidence span withheld — anchor status: not_found]
- [readme] GNN-RT input/output: [evidence span withheld — anchor status: external_doc]
- [methods] Conformer generation step: "re = AllChem.EmbedMultipleConfs(mol, numConfs = 1, params = ps)"
