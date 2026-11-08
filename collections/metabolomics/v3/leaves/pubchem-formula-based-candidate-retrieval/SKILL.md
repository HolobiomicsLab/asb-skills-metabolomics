---
name: pubchem-formula-based-candidate-retrieval
description: Use when you have a measured precursor molecular formula plus an observed
  LC retention time and need to enumerate and prune isomeric candidate structures
  to reduce metabolite-annotation ambiguity.
when_to_use_negative:
- The unknown has no reliable observed retention time under the chromatographic method
  the RT model was trained/fine-tuned for — RT pruning is then uninformative.
- Formula-matched isomers are indistinguishable by RT-Transformer (near-identical
  predicted RTs), e.g. positional isomers with similar chromatographic behavior.
- You need structural ranking (MS/MS similarity) rather than chromatographic plausibility
  — this skill only eliminates formula-matched candidates by RT.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: RT-Transformer
  role: Graph-attention + 1D-Transformer model used to predict retention times of
    every formula-matched candidate under the target chromatographic method, enabling
    RT-based candidate elimination
  repo: https://github.com/01dadada/RT-Transformer
- name: PubChem PUG REST
  role: Exact molecular formula query service that retrieves all isomeric candidate
    structures for each test molecule
- name: RDKit
  role: Converts candidate isomeric SMILES into molecular-graph features compatible
    with RT-Transformer's graph input (listed in README requirements)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae084
    title: 'RT-Transformer: retention time prediction for metabolite annotation to
      assist in metabolite identification'
  source_content_hash: sha256:0b887860724918e7a454fe3c15514b819d0f2166ba9165e5e432c40d52183ed7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/01dadada/RT-Transformer
  tools:
  - PubChem PUG REST
  - RDKit
  - RT-Transformer
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# PubChem formula-based candidate retrieval

## Summary

Retrieves all isomeric candidate structures from PubChem (PUG REST) by exact molecular formula for each unknown metabolite, then uses the RT-Transformer retention-time predictor to eliminate formula-matched candidates whose predicted RT is inconsistent with the observed chromatographic RT.

## When to use

Use when you have a measured precursor molecular formula plus an observed LC retention time and need to enumerate and prune isomeric candidate structures to reduce metabolite-annotation ambiguity.

## When NOT to use

- The unknown has no reliable observed retention time under the chromatographic method the RT model was trained/fine-tuned for — RT pruning is then uninformative.
- Formula-matched isomers are indistinguishable by RT-Transformer (near-identical predicted RTs), e.g. positional isomers with similar chromatographic behavior.
- You need structural ranking (MS/MS similarity) rather than chromatographic plausibility — this skill only eliminates formula-matched candidates by RT.

## Inputs

- Molecular formula of the unknown metabolite (e.g., from formula annotation of an MS peak)
- Observed liquid-chromatography retention time of the unknown peak under the target method
- PubChem PUG REST access for exact-formula compound search returning isomeric SMILES
- Pre-trained RT-Transformer weights (best_state_dict.pth or best_state_download_dict.pth)

## Outputs

- Filtered list of isomeric candidate SMILES consistent with the observed RT
- Per-molecule candidate counts before and after RT filtering
- Per-molecule and mean filter rates (fraction of formula-matched candidates removed), e.g. candidate_filter_results.csv

## How to apply

For each molecule, convert its formula to a PUG REST exact-formula query and retrieve all isomeric candidate structures with their isomeric SMILES. Convert each candidate SMILES to molecular-graph features compatible with RT-Transformer's graph attention network input (RDKit per README requirements, following QuickStart.ipynb), and run the pre-trained model (best_state_dict.pth trained on full data or best_state_download_dict.pth trained on retained data) to predict RT under the target chromatographic method. Discard candidates whose predicted RT is inconsistent with the observed RT, recording per-molecule candidate counts before/after filtering. Because RT-Transformer reaches an average absolute error of 27.30 (retained molecules removed) to 33.41 (no samples removed) on SMRT, the consistency window must tolerate this error rather than demanding exact matches; the SMRT benchmark showed a mean 59.35% candidate elimination rate over 100 test molecules.

## Related tools

- **RT-Transformer** (Graph-attention + 1D-Transformer model used to predict retention times of every formula-matched candidate under the target chromatographic method, enabling RT-based candidate elimination) — https://github.com/01dadada/RT-Transformer
- **PubChem PUG REST** (Exact molecular formula query service that retrieves all isomeric candidate structures for each test molecule)
- **RDKit** (Converts candidate isomeric SMILES into molecular-graph features compatible with RT-Transformer's graph input (listed in README requirements))

## Examples

```
curl "https://pubchem.ncbi.nlm.nih.gov/rest/pug/compound/fastformula/C7H10O5/property/IsomericSMILES/TXT"; then score candidates with the RT-Transformer QuickStart.ipynb pipeline
```

## Evaluation signals

- Mean filter rate over 100 SMRT test molecules reproduces the reported 59.35% candidate-elimination rate.
- Every formula query returns candidate counts > 0 before filtering; per-molecule candidate counts and filter rates are logged (candidate_filter_results.csv).
- Predicted RTs of retained candidates fall within a consistency window compatible with RT-Transformer's reported average absolute error (27.30 retained / 33.41 full data).
- RT-Transformer model runs reproduce README validation via `python ./test.py` before being applied to candidates.
- Candidates passed by the filter remain exact-formula isomers of the unknown — no formula mismatches introduced by the retrieval step.

## Limitations

- RT-Transformer's average absolute error is 27.30 (not-retained molecules removed) to 33.41 (no samples removed) on SMRT, so RT-based elimination is inherently approximate and can discard or retain wrong candidates near this error scale.
- Retention times are chromatographic-method dependent; the pre-trained model must be transferred/fine-tuned (transfer.py with a data.csv having 'InChI' and 'RT' columns) for methods outside the SMRT/PredRet training conditions.
- The pre-trained model weights differ by training set (best_state_dict.pth full data vs best_state_download_dict.pth retained data), affecting error characteristics of the filter.

## Evidence

- [abstract] we present RT-Transformer, a novel deep neural network model coupled with graph attention network and 1D-Transformer, which can predict retention times under any chromatographic methods: "we present RT-Transformer, a novel deep neural network model coupled with graph attention network and 1D-Transformer, which can predict retention times under any chromatographic methods"
- [abstract] the average absolute error reached 27.30 after removing not retained molecules. Still, it reached 33.41 when no samples were removed: "the average absolute error reached 27.30 after removing not retained molecules. Still, it reached 33.41 when no samples were removed"
- [readme] The RT-Tranformer combine the fingerprint and the molecular graph data and predict retention time as the output.: [evidence span withheld — anchor status: external_doc]
- [readme] Prepare your dataset as a csv file which has "InChI" and "RT" columns.: [evidence span withheld — anchor status: external_doc]
- [readme] The SMRT dataset is collect from [this paper](https://doi.org/10.1038/s41467-019-13680-7): [evidence span withheld — anchor status: external_doc]
