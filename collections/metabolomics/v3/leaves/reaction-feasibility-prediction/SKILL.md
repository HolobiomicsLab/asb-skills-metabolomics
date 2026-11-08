---
name: reaction-feasibility-prediction
description: Use when you have Pickaxe-generated novel reactions in a MongoDB database
  and need to exclude reactions with unfavorable physiology or unlikely catalytic
  transformation before downstream pathway design or metabolomics annotation.
when_to_use_negative:
- When ΔG values fall outside the scope of eQuilibrator predictions — the filter cannot
  comment on their feasibility, so reactions must be retained rather than pruned.
- When reactions are already experimentally validated rather than novel generated
  reactions; DeepRFC's substrate-product pair predictions target KEGG-derived transformations
  and may not generalize to curated pathways.
- When the analysis goal is compound-level filtering (e.g. molecular weight or m/z
  matching) rather than reaction-level feasibility.
tools:
- name: eQuilibrator
  role: Computes physiological Gibbs free energy (ΔG) for each Pickaxe-generated reaction
    to apply the thermodynamics filter
  repo: https://equilibrator.readthedocs.io/en/latest/
- name: DeepRFC
  role: Deep neural net scoring substrate-product pairs within KEGG reactions to predict
    reaction feasibility
  repo: https://bitbucket.org/kaistsystemsbiology/deeprfc/src/master/
- name: PyMongo
  role: Retrieves Pickaxe run results from the MongoDB database for filtering
  repo: https://pymongo.readthedocs.io/en/stable/
- name: Pickaxe
  role: Generates the novel reactions that are subsequently scored and filtered
  repo: https://github.com/tyo-nu/MINE-Database
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:82b4e58ac16b832a6c9074ec61f7ed83806dfff8d0cb9c63a4d1d09c53232aa1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepRFC
  - Pickaxe
  - PyMongo
  - eQuilibrator
  - minedatabase
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# reaction-feasibility-prediction

## Summary

Predict whether reactions generated during Pickaxe network expansion are thermodynamically and mechanistically feasible, using eQuilibrator physiological Gibbs free energies (ΔG > 50 kJ/mol cutoff) and the DeepRFC substrate–product feasibility neural net, and prune infeasible reactions from the expanding network.

## When to use

Use when you have Pickaxe-generated novel reactions in a MongoDB database and need to exclude reactions with unfavorable physiology or unlikely catalytic transformation before downstream pathway design or metabolomics annotation.

## When NOT to use

- When ΔG values fall outside the scope of eQuilibrator predictions — the filter cannot comment on their feasibility, so reactions must be retained rather than pruned.
- When reactions are already experimentally validated rather than novel generated reactions; DeepRFC's substrate-product pair predictions target KEGG-derived transformations and may not generalize to curated pathways.
- When the analysis goal is compound-level filtering (e.g. molecular weight or m/z matching) rather than reaction-level feasibility.

## Inputs

- Pickaxe reaction network stored in a MongoDB database (accessible via PyMongo)
- Reaction definitions with substrate-product pairs
- eQuilibrator-api installation (conda install -c condaforge equilibrator-api)
- DeepRFC installation (https://bitbucket.org/kaistsystemsbiology/deeprfc)

## Outputs

- CSV table with one row per reaction containing the computed physiological ΔG (kJ/mol) or a non-computable flag, plus the DeepRFC feasibility label
- Filtered reaction network retaining only feasible reactions

## How to apply

Load the Pickaxe run results from MongoDB via PyMongo and, for each reaction, compute the physiological Gibbs free energy with eQuilibrator-api, flagging reactions for which no ΔG can be computed as non-computable. Remove reactions with physiological ΔG greater than 50 kJ/mol while retaining reactions outside the scope of eQuilibrator predictions, since the filter cannot comment on their feasibility. Score remaining reactions with DeepRFC — a deep neural net trained on substrate-product pairs within KEGG reactions — and remove those predicted infeasible. In the paper's EcoCyc metabolomics workflow this is applied per generation, and the README notes feasibility can be accelerated with precomputed scripts (get_physiological_dgs.py, feasibility_first_gen.py). Write results to CSV with one row per reaction containing the ΔG value (or non-computable flag) and the feasibility label.

## Related tools

- **eQuilibrator** (Computes physiological Gibbs free energy (ΔG) for each Pickaxe-generated reaction to apply the thermodynamics filter) — https://equilibrator.readthedocs.io/en/latest/
- **DeepRFC** (Deep neural net scoring substrate-product pairs within KEGG reactions to predict reaction feasibility) — https://bitbucket.org/kaistsystemsbiology/deeprfc/src/master/
- **PyMongo** (Retrieves Pickaxe run results from the MongoDB database for filtering) — https://pymongo.readthedocs.io/en/stable/
- **Pickaxe** (Generates the novel reactions that are subsequently scored and filtered) — https://github.com/tyo-nu/MINE-Database

## Examples

```
conda install -c condaforge equilibrator-api; pip install minedatabase
```

## Evaluation signals

- Verify the output CSV covers every input reaction from the Pickaxe run, with ΔG and feasibility fields present for each row.
- Confirm removal decisions are consistent with the >50 kJ/mol physiological ΔG threshold.
- Check that reactions flagged non-computable by eQuilibrator are retained in the network, not removed.
- Confirm the physiological ΔG cutoff value and feasibility labels are written alongside reaction identifiers.
- The paper describes but never evaluates these filters in isolation, so no quantitative accuracy criterion is available — rely on schema and threshold consistency checks.

## Limitations

- Reactions outside the scope of eQuilibrator predictions are retained despite unknown feasibility.
- The feasibility and thermodynamics filters are described but not quantitatively evaluated in isolation in the paper.
- DeepRFC requires following external installation instructions and depends on KEGG-derived training data, limiting generalization to non-KEGG transformations.
- No software version number is stated for Pickaxe, eQuilibrator, or DeepRFC in the paper or README.

## Evidence

- [other] DeepRFC, a deep neural net developed by Kim et al. that uses substrate-product pairs within KEGG reactions to predict a reaction's feasibility: [evidence span withheld — anchor status: not_found]
- [results] The thermodynamics filter was set to remove reactions with a physiological Gibbs free energy greater than 50 kJ/mol.: "The thermodynamics filter was set to remove reactions with a physiological Gibbs free energy greater than 50 kJ/mol."
- [results] These reactions were left to remain in the network, as the thermodynamics filter cannot comment on their feasibility.: "These reactions were left to remain in the network, as the thermodynamics filter cannot comment on their feasibility."
- [readme] The thermodynamics filters require the use of [eQuilibrator](https://equilibrator.readthedocs.io/en/latest/). This can be installed via conda: "The thermodynamics filters require the use of [eQuilibrator](https://equilibrator.readthedocs.io/en/latest/). This can be installed via conda"
- [readme] The feasibility filters require the use of DeepRFC, which can be installed by following the instructions found on their website.: "The feasibility filters require the use of DeepRFC, which can be installed by following the instructions found on their website."
