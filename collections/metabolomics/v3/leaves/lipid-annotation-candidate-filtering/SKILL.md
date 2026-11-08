---
name: lipid-annotation-candidate-filtering
description: Use when you must narrow a LipidBlast-style candidate list from precursor
  m/z matches for lipidomics queries and want multidimensional (RT, CCS) re-ranking
  with recall metrics.
when_to_use_negative:
- Your molecules contain elements outside C, H, O, N, P, S, F, Cl, Br, I, Co, As,
  and Se — SigmaCCS cannot predict CCS for them
- Your ion adduct is not one of the three encoded types ([M+H]+, [M+Na]+, [M-H]-)
  — SigmaCCS only handles these adduct ion types
- You already have a fully resolved identification table with no candidate ambiguity
  — filtering would add no ranking benefit
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
tools:
- name: SigmaCCS
  role: Predicts CCS values for candidate SMILES via GNN on 3D-conformer graphs with
    adduct encoding, used in the third filtering stage
  repo: https://github.com/zmzhang/SigmaCCS
- name: GNN-RT
  role: Predicts LC retention times from molecular graphs, used to re-rank m/z-matched
    candidates in the second filtering stage
  repo: https://github.com/Qiong-Yang/GNN-RT
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42004-023-00939-w
    title: Highly accurate and large-scale collision cross sections prediction with
      graph neural networks
  source_content_hash: sha256:f6ad77e8b7d45ae692e99c442f02dabeb095cd9855f7227f250401b51bac3f88
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmzhang/SigmaCCS
  tools:
  - GNN-RT
  - SigmaCCS
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# lipid-annotation-candidate-filtering

## Summary

A staged candidate-filtering workflow for lipid identification that progressively combines precursor m/z matching, GNN-RT-predicted retention times, and SigmaCCS-predicted CCS values to re-rank candidate annotations and improve Recall@1/Recall@30.

## When to use

Use when you must narrow a LipidBlast-style candidate list from precursor m/z matches for lipidomics queries and want multidimensional (RT, CCS) re-ranking with recall metrics.

## When NOT to use

- Your molecules contain elements outside C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se — SigmaCCS cannot predict CCS for them
- Your ion adduct is not one of the three encoded types ([M+H]+, [M+Na]+, [M-H]-) — SigmaCCS only handles these adduct ion types
- You already have a fully resolved identification table with no candidate ambiguity — filtering would add no ranking benefit

## Inputs

- Query lipid list with measured precursor m/z and retention times (e.g., 761 negative-mode mouse lung entries, loaded via pandas)
- LipidBlast candidate annotation database with per-candidate SMILES
- Trained GNN-RT model for retention time prediction
- Trained SigmaCCS model (model.h5) with parameter.pkl and adduct encoding

## Outputs

- Ranked candidate lists per query after each of the three filtering stages (m/z; m/z+RT; m/z+RT+CCS)
- Per-stage Recall@1 and Recall@30 metrics (expected ~15.2%/24.6%/28.9% and ~47.6%/78.4%/91.2%)
- recall_results.csv with the per-stage recall metrics

## How to apply

Load the query lipids (e.g., 761 negative-mode mouse lung queries) with measured m/z and retention times using pandas. Stage 1 filters LipidBlast candidate annotations by precursor m/z matching only. Stage 2 re-ranks those candidates by additionally applying retention times predicted with the GNN-RT graph neural network. Stage 3 further re-ranks by SigmaCCS-predicted CCS values, which requires converting candidate SMILES to 3D-conformer graphs via RDKit (ETKDGv3 embedding with numConfs=1, then MMFF optimization) and feeding the graph plus the [M-H]- adduct encoding into the trained SigmaCCS model. After each stage, compute Recall@1 and Recall@30 across all queries and write per-stage metrics to recall_results.csv.

## Related tools

- **SigmaCCS** (Predicts CCS values for candidate SMILES via GNN on 3D-conformer graphs with adduct encoding, used in the third filtering stage) — https://github.com/zmzhang/SigmaCCS
- **GNN-RT** (Predicts LC retention times from molecular graphs, used to re-rank m/z-matched candidates in the second filtering stage) — https://github.com/Qiong-Yang/GNN-RT

## Examples

```
jupyter notebook others/Filtering.ipynb  # plus: re = AllChem.EmbedMultipleConfs(mol, numConfs = 1, params = ps); re = AllChem.MMFFOptimizeMoleculeConfs(mol, numThreads = 0)
```

## Evaluation signals

- Stage-wise Recall@1 should increase from ~15.2% (m/z) to ~24.6% (m/z+RT) to ~28.9% (m/z+RT+CCS) on the 761 mouse lung queries
- Stage-wise Recall@30 should reach ~47.6%, ~78.4%, and ~91.2%, matching Supplementary Table 12 within a small tolerance
- ETKDG conformer generation must succeed with numConfs=1 followed by MMFF optimization; candidates failing these steps are excluded as SigmaCCS inputs
- Only candidates whose adduct matches the query adduct (e.g., [M-H]-) are passed to the SigmaCCS model
- Output file recall_results.csv must contain per-stage metrics for all three filtering stages

## Limitations

- SigmaCCS cannot predict molecules containing elements other than C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se
- SigmaCCS covers only three adduct ion types ([M+H]+, [M+Na]+, [M-H]-); other adducts are not encoded
- ETKDG could not generate conformers for a small fraction of molecules (33/94,161,896 PubChem, 5/5,645 CCSbase), and MMFF94 minimization failed for 662 conformers, so some candidates will be unfilterable by CCS

## Evidence

- [methods] Filtering of target unknown molecules based on the CCS and mz of the molecules: "Filtering of target unknown molecules based on the CCS and *mz* of the molecules"
- [abstract] based on graph neural networks using 3D conformers as inputs: "we have developed the Structure included graph merging with adduct method for CCS prediction (SigmaCCS) based on graph neural networks using 3D conformers as inputs"
- [discussion] thus SigmaCCS focuses on the three adduct ion types: "thus SigmaCCS focuses on the three adduct ion types"
- [readme] SigmaCCS is a model for predicting CCS based on graph neural networks, so we need to convert SMILES strings to Graph: "SigmaCCS is a model for predicting CCS based on graph neural networks, so we need to convert SMILES strings to Graph."
- [methods] re = AllChem.EmbedMultipleConfs(mol, numConfs = 1, params = ps): "re = AllChem.EmbedMultipleConfs(mol, numConfs = 1, params = ps)"
