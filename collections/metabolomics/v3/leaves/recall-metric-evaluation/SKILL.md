---
name: recall-metric-evaluation
description: Use when you have precursor m/z-matched candidate lists plus orthogonal
  predictions (GNN-RT retention time, SigmaCCS CCS) and need to measure whether adding
  dimensions improves correct-molecule ranking via Recall@k.
when_to_use_negative:
- Queries lack measured retention times or candidates lack SMILES, so neither GNN-RT
  nor SigmaCCS scores can be computed for the later stages
- The adduct of interest is not one of the three SigmaCCS ion types ([M+H]+, [M+Na]+,
  [M-H]-)
- Each query already has a single identification (no candidate ranking), making Recall@k
  undefined
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: SigmaCCS
  role: Predicts CCS values for candidate SMILES via 3D-conformer graphs and the [M-H]-
    adduct, powering the third filtering stage whose recall gain is being measured
  repo: https://github.com/zmzhang/SigmaCCS
- name: GNN-RT
  role: Predicts LC retention times used to re-rank m/z-matched candidates in the
    second filtering stage
  repo: https://github.com/Qiong-Yang/GNN-RT
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42004-023-00939-w
    title: Highly accurate and large-scale collision cross sections prediction with
      graph neural networks
  source_content_hash: sha256:296267dc21fe112598a7dd017e6ad7c26bd52a849ea72d137f1690733358015b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmzhang/SigmaCCS
  tools:
  - GNN-RT
  - SigmaCCS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# recall-metric-evaluation

## Summary

Quantifies how correctly ranked true lipid identifications are after each stage of a multidimensional candidate-filtering chain (m/z → m/z+RT → m/z+RT+CCS), by computing Recall@1 and Recall@30 over 761 negative-mode mouse lung lipid queries.

## When to use

Use when you have precursor m/z-matched candidate lists plus orthogonal predictions (GNN-RT retention time, SigmaCCS CCS) and need to measure whether adding dimensions improves correct-molecule ranking via Recall@k.

## When NOT to use

- Queries lack measured retention times or candidates lack SMILES, so neither GNN-RT nor SigmaCCS scores can be computed for the later stages
- The adduct of interest is not one of the three SigmaCCS ion types ([M+H]+, [M+Na]+, [M-H]-)
- Each query already has a single identification (no candidate ranking), making Recall@k undefined

## Inputs

- Query lipid table with 761 negative-mode mouse lung lipids including measured m/z and retention times
- LipidBlast candidate annotation lists per query with precursor m/z and SMILES
- Trained GNN-RT retention-time prediction model
- Trained SigmaCCS model (model.h5, parameter.pkl) for the [M-H]- adduct

## Outputs

- Per-stage ranked candidate lists for each of the 761 queries (m/z, m/z+RT, m/z+RT+CCS)
- recall_results.csv containing Recall@1 and Recall@30 for each of the three filtering stages

## How to apply

Load the 761 negative-mode mouse lung lipid queries (measured m/z and retention times) with pandas. Stage 1 screens LipidBlast candidate annotations by precursor m/z matching; Stage 2 re-ranks the m/z-matched candidates with predicted retention times from the trained GNN-RT model; Stage 3 further re-ranks using SigmaCCS-predicted CCS values, obtained by converting candidate SMILES to 3D-conformer graphs (ETKDGv3 embedding with numConfs=1 followed by MMFF optimization) and running the trained model on the [M-H]- adduct graph. The rationale is that CCS and RT are stable, readily obtained physicochemical properties orthogonal to m/z that improve compound identification. Compute Recall@1 and Recall@30 after each of the three stages, write per-stage metrics to recall_results.csv, and compare against Supplementary Table 12 within a small tolerance.

## Related tools

- **SigmaCCS** (Predicts CCS values for candidate SMILES via 3D-conformer graphs and the [M-H]- adduct, powering the third filtering stage whose recall gain is being measured) — https://github.com/zmzhang/SigmaCCS
- **GNN-RT** (Predicts LC retention times used to re-rank m/z-matched candidates in the second filtering stage) — https://github.com/Qiong-Yang/GNN-RT

## Examples

```
re = AllChem.EmbedMultipleConfs(mol, numConfs = 1, params = ps); AllChem.MMFFOptimizeMoleculeConfs(mol, numThreads = 0)
```

## Evaluation signals

- Recall@1 across stages should approximate 15.2% (m/z), 24.6% (m/z+RT), and 28.9% (m/z+RT+CCS)
- Recall@30 across stages should approximate 47.6% (m/z), 78.4% (m/z+RT), and 91.2% (m/z+RT+CCS)
- Recall@1 and Recall@30 must increase monotonically as RT and CCS dimensions are added
- recall_results.csv values should match Supplementary Table 12 within a small tolerance
- Recall@k is computed over the full set of 761 negative-mode lipids for each stage

## Limitations

- SigmaCCS cannot predict molecules containing elements other than C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se, so such candidates drop out of stage 3
- SigmaCCS handles only three adduct ion types ([M+H]+, [M+Na]+, [M-H]-); other adducts are unsupported
- ETKDG conformer generation failed for 33/94,161,896 PubChem molecules and 5/5,645 CCSbase entries, and 662 ETKDG conformers could not be minimized by MMFF94, producing missing CCS predictions

## Evidence

- [methods] Filtering of target unknown molecules based on the CCS and *mz* of the molecules: "Filtering of target unknown molecules based on the CCS and *mz* of the molecules"
- [abstract] We developed a method named Structure included graph merging with adduct method for CCS prediction (SigmaCCS) based on graph neural networks using 3D conformers as inputs: "we have developed the Structure included graph merging with adduct method for CCS prediction (SigmaCCS) based on graph neural networks using 3D conformers as inputs"
- [supplementary] the reported Recall@1 values (~15.2%/24.6%/28.9%) and Recall@30 values (~47.6%/78.4%/91.2%) should match Supplementary Table 12 within a small tolerance: [evidence span withheld — anchor status: not_found]
- [readme] SigmaCCS is a model for predicting CCS based on graph neural networks, so we need to convert SMILES strings to Graph: "SigmaCCS is a model for predicting CCS based on graph neural networks, so we need to convert SMILES strings to Graph"
- [methods] The CCS prediction of the molecule is obtained by inputting Graph and Adduct into the already trained SigmaCCS model: "The CCS prediction of the molecule is obtained by inputting Graph and Adduct into the already trained SigmaCCS model."
