---
name: ccs-database-joining-by-adduct
description: Use when you have a large table of predicted CCS values plus an independent
  reference CCS database and need adduct-stratified agreement statistics rather than
  a single pooled correlation.
when_to_use_negative:
- Your compounds contain elements outside C, H, O, N, P, S, F, Cl, Br, I, Co, As,
  Se — SigmaCCS cannot predict these molecules, so joins would silently drop them
- You need CCS values for adduct types other than the three encoded by SigmaCCS ([M+H]+,
  [M+Na]+, [M-H]-), e.g. [M+K]+ or [M+NH4]+
- Your input is already experimental CCS values that need no model prediction — join
  directly to the reference database instead of running the GNN pipeline
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: SigmaCCS
  role: Graph neural network model whose convertToGraph/MyDataset and Model_prediction
    functions produce the predicted CCS values joined to the reference database
  repo: https://github.com/zmzhang/SigmaCCS
- name: ISiCLE
  role: Source of the independently calculated reference CCS values (PNNL database
    v0.1.0, TM-method pipeline) used as the join target
- name: RDKit
  role: Generates and MMFF-optimizes 3D conformers (ETKDGv3) that feed the SigmaCCS
    graph construction
- name: TensorFlow
  role: Runtime for executing the pretrained SigmaCCS model.h5 to obtain CCS predictions
    before joining
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42004-023-00939-w
    title: Highly accurate and large-scale collision cross sections prediction with
      graph neural networks
  source_content_hash: sha256:5a3f5f33a6c728599d01d4842c30b33111a46beb5b58191ed4fd3f5bd58ef564
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmzhang/SigmaCCS
  tools:
  - ISiCLE
  - RDKit
  - SigmaCCS
  - TensorFlow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# ccs-database-joining-by-adduct

## Summary

Join SigmaCCS-predicted CCS values to reference collision cross section values (e.g., the 825,702-entry ISiCLE-calculated PNNL database, v0.1.0) on compound identifier and adduct type, then compute per-adduct Pearson correlation coefficients for [M+H]+, [M+Na]+, and [M-H]- to verify predictions at database scale.

## When to use

Use when you have a large table of predicted CCS values plus an independent reference CCS database and need adduct-stratified agreement statistics rather than a single pooled correlation.

## When NOT to use

- Your compounds contain elements outside C, H, O, N, P, S, F, Cl, Br, I, Co, As, Se — SigmaCCS cannot predict these molecules, so joins would silently drop them
- You need CCS values for adduct types other than the three encoded by SigmaCCS ([M+H]+, [M+Na]+, [M-H]-), e.g. [M+K]+ or [M+NH4]+
- Your input is already experimental CCS values that need no model prediction — join directly to the reference database instead of running the GNN pipeline

## Inputs

- Compound SMILES/adduct lists keyed by compound identifier
- Reference CCS database (825,702 ISiCLE-calculated entries, PNNL database v0.1.0) with compound identifier and adduct columns
- Pretrained SigmaCCS model files: model.h5 and parameter.pkl
- SigmaCCS train/test metadata (TrainData.csv, TestData.csv) if reusing the repo's split

## Outputs

- Per-adduct Pearson correlation coefficients for [M+H]+, [M+Na]+, and [M-H]-
- correlation_summary.csv recording the joined per-adduct agreement statistics

## How to apply

Load the compound SMILES/adduct lists and the reference CCS database (825,702 ISiCLE-calculated entries from PNNL v0.1.0), matching entries on compound identifier and adduct type rather than pooling adducts. For each compound, generate a 3D conformer with RDKit ETKDGv3 (numConfs=1, randomSeed=-1, maxAttempts=1, useRandomCoords=True), optimize with MMFFOptimizeMoleculeConfs (numThreads=0), and build the molecular graph via SigmaCCS convertToGraph/MyDataset (node features, adjacency, edge features). Run SigmaCCS Model_prediction with the pretrained model.h5 and parameter.pkl to obtain predicted CCS per adduct. Join predictions to reference CCS per adduct using pandas, then compute the Pearson r separately for [M+H]+, [M+Na]+, and [M-H]-; the paper's rationale is that adduct-specific ion sizes give different CCS distributions, so a single pooled coefficient would mask per-adduct bias. Accept the reproduction when each computed r matches the reported values — 0.9572 ([M+H]+), 0.9586 ([M+Na]+), 0.9546 ([M-H]-) — within ±0.005 in the correlation_summary.csv output.

## Related tools

- **SigmaCCS** (Graph neural network model whose convertToGraph/MyDataset and Model_prediction functions produce the predicted CCS values joined to the reference database) — https://github.com/zmzhang/SigmaCCS
- **ISiCLE** (Source of the independently calculated reference CCS values (PNNL database v0.1.0, TM-method pipeline) used as the join target)
- **RDKit** (Generates and MMFF-optimizes 3D conformers (ETKDGv3) that feed the SigmaCCS graph construction)
- **TensorFlow** (Runtime for executing the pretrained SigmaCCS model.h5 to obtain CCS predictions before joining)

## Examples

```
joined = pred.merge(ref, on=['compound_id','adduct']); joined.groupby('adduct').apply(lambda g: g.ccs_pred.corr(g.ccs_ref)).to_csv('correlation_summary.csv')
```

## Evaluation signals

- Pearson r per adduct matches reported values within ±0.005: 0.9572 for [M+H]+, 0.9586 for [M+Na]+, 0.9546 for [M-H]-
- Every joined row pairs identical compound identifiers and identical adduct type — no cross-adduct matches in the merge keys
- Join coverage reflects the 825,702-entry reference database; molecules whose ETKDG conformer generation or MMFF94 minimization failed are excluded and counted
- correlation_summary.csv is produced with one row per adduct ion type (exactly three) and finite correlation values

## Limitations

- SigmaCCS handles only three adduct ion types; encoding more adduct types is noted as future improvement
- ETKDG conformer generation failed for 33/94,161,896 PubChem molecules and 5/5,645 CCSbase entries, and MMFF94 minimization failed for 662 conformers, creating reproducibility gaps in the join coverage
- The ISiCLE reference pipeline has high computational-resource demands, limiting how large a reference set can be regenerated from scratch

## Evidence

- [methods] Adduct-encoded prediction is the join stratification unit: "The CCS prediction of the molecule is obtained by inputting Graph and Adduct into the already trained SigmaCCS model."
- [discussion] Only three adduct ion types are supported: "thus SigmaCCS focuses on the three adduct ion types"
- [methods] Scale of the predicted database that joins against reference values: "a dataset including CCS values of ~90,000,000 molecules from Pubchem was formed"
- [discussion] Some molecules cannot enter the join because conformer generation fails: "the conformers of some molecules (33 out 94,161,896) in PubChem and (5 out 5645 entries) in CCSbase cannot be generated by the ETKDG method"
