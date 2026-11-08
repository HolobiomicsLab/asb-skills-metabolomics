---
name: reaction-network-filtering
description: Use when a Pickaxe network expansion explodes (e.g. 13.7 M reactions
  from KEGG) and you must constrain compounds/reactions before the next generation,
  for metabolomics peak annotation, thermodynamic feasibility screening, or targeted
  compound discovery.
when_to_use_negative:
- You need a full, unpruned network expansion — filters remove compounds/reactions
  before the next generation, so the final network is incomplete by design (e.g. EcoCyc
  expansion pruned from 906,086 compounds to 2000 sampled ones).
- 'Your starting reaction rules or targets fall outside eQuilibrator''s scope: no
  ΔG is returned and the thermodynamics filter cannot judge those reactions, so pruning
  decisions are unsupported.'
- The goal is database-wide rule coverage assessment (e.g. JN1224min mapping percentages
  on KEGG/BRENDA) rather than constrained generation — no filter inputs (targets,
  peaks, ΔG) exist to drive pruning.
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: Pickaxe
  role: Runs the network generation and applies the four filters on-the-fly between
    expansion generations
  repo: https://github.com/tyo-nu/MINE-Database
- name: MINE-Database (minedatabase python package)
  role: Python package that bundles Pickaxe and provides the filter framework invoked
    via `pip install minedatabase`
  repo: https://pypi.org/project/minedatabase/2.0.0/
- name: eQuilibrator
  role: Computes physiological Gibbs free energies used by the thermodynamics filter's
    50 kJ/mol threshold
  repo: https://equilibrator.readthedocs.io/en/latest/
- name: DeepRFC
  role: Predicts reaction feasibility from substrate–product pairs; scores reactions
    so the feasibility filter can remove likely-infeasible ones
  repo: https://bitbucket.org/kaistsystemsbiology/deeprfc/src/master/
- name: MongoDB
  role: Stores run results; PyMongo reads reactions/compounds so filter inputs and
    outputs can be loaded and written
  repo: https://www.mongodb.com/try/download/community
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:c9550b13637d351fd1ac4d0f15fa835cd4b170cca9edf2bf1fa8530689f8279c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyo-nu/MINE-Database
  tools:
  - DeepRFC
  - MINE-Database (minedatabase python package)
  - MongoDB
  - Pickaxe
  - PyMongo
  - eQuilibrator
  - minedatabase
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# reaction-network-filtering

## Summary

On-the-fly pruning of Pickaxe-generated reaction networks using four composable filters — chemical similarity (cutoff or weighted sampling), metabolomics m/z matching, thermodynamics via eQuilibrator (ΔG > 50 kJ/mol removal), and DeepRFC reaction-feasibility scoring — so expansions stay small enough to interpret.

## When to use

Use when a Pickaxe network expansion explodes (e.g. 13.7 M reactions from KEGG) and you must constrain compounds/reactions before the next generation, for metabolomics peak annotation, thermodynamic feasibility screening, or targeted compound discovery.

## When NOT to use

- You need a full, unpruned network expansion — filters remove compounds/reactions before the next generation, so the final network is incomplete by design (e.g. EcoCyc expansion pruned from 906,086 compounds to 2000 sampled ones).
- Your starting reaction rules or targets fall outside eQuilibrator's scope: no ΔG is returned and the thermodynamics filter cannot judge those reactions, so pruning decisions are unsupported.
- The goal is database-wide rule coverage assessment (e.g. JN1224min mapping percentages on KEGG/BRENDA) rather than constrained generation — no filter inputs (targets, peaks, ΔG) exist to drive pruning.

## Inputs

- Pickaxe-generated set of novel reactions and compounds stored in a mongo database (via PyMongo)
- Starting compound list filtered to <600 g/mol (e.g. KEGG Release 99.0, EcoCyc 25.1, or YMDB 2.0 subsets)
- User-supplied target compounds for the similarity filter (e.g. 250 KEGG compounds with 4–9 carbons)
- List of unannotated experimental m/z peaks with adduct definitions and a mass tolerance for the metabolomics filter
- User-specified atom-count limits (e.g. fewer than twelve carbons in non-cofactor molecules) for the atomic composition filter

## Outputs

- Pruned compound/reaction network retained only for compounds passing the active filter(s)
- CSV table with one row per reaction containing computed physiological ΔG value (or a non-computable flag) and the DeepRFC feasibility label
- CSV table of metabolomics-matched compounds recording the matched experimental m/z peak and the adduct that produced the match
- Smaller network suitable for the next generation of expansion or for downstream analysis

## How to apply

After each generation of rule application (e.g. top 500 JN1224min rules applied to 12,731 KEGG compounds <600 g/mol), apply filters before expanding further. The chemical similarity filter computes Tanimoto-like similarity to user targets and either removes compounds below a cutoff threshold or samples a fixed number (e.g. 2000) from a weighted distribution of similarity scores scaled to the zeroth, third, or sixth power. The metabolomics filter computes possible m/z values for each predicted compound from user-supplied adducts and retains only compounds whose adducted m/z matches an experimental peak (e.g. 200 peaks in the 150–300 Da range) within a user-defined tolerance. The thermodynamics filter computes physiological Gibbs free energies with eQuilibrator-api and removes reactions with ΔG greater than 50 kJ/mol, while reactions whose ΔG falls outside eQuilibrator's scope are retained because the filter cannot comment on their feasibility. The feasibility filter scores remaining reactions with DeepRFC and removes those predicted infeasible. All four filters run on-the-fly during generation so pruned networks never grow to the unfiltered size.

## Related tools

- **Pickaxe** (Runs the network generation and applies the four filters on-the-fly between expansion generations) — https://github.com/tyo-nu/MINE-Database
- **MINE-Database (minedatabase python package)** (Python package that bundles Pickaxe and provides the filter framework invoked via `pip install minedatabase`) — https://pypi.org/project/minedatabase/2.0.0/
- **eQuilibrator** (Computes physiological Gibbs free energies used by the thermodynamics filter's 50 kJ/mol threshold) — https://equilibrator.readthedocs.io/en/latest/
- **DeepRFC** (Predicts reaction feasibility from substrate–product pairs; scores reactions so the feasibility filter can remove likely-infeasible ones) — https://bitbucket.org/kaistsystemsbiology/deeprfc/src/master/
- **MongoDB** (Stores run results; PyMongo reads reactions/compounds so filter inputs and outputs can be loaded and written) — https://www.mongodb.com/try/download/community

## Examples

```
python -m minedatabase.pickaxe -C ./data/metacyc_generalized_rules.tsv -r ./data/metacyc_coreactants.tsv -g 1 -c ../example_data/iML1515_ecoli_GEM.csv
```

## Evaluation signals

- Every metabolomics-retained compound has at least one computed adduct m/z within the user-defined tolerance of an input peak, and each output row records the matched peak and adduct.
- The thermodynamics/feasibility output CSV covers every input reaction, with a ΔG value or non-computable flag plus a feasibility label per row.
- No retained reaction has a computable physiological ΔG greater than 50 kJ/mol; reactions flagged non-computable by eQuilibrator remain in the network.
- Similarity-sampling runs yield exactly the specified number of compounds (e.g. 2000) drawn from the weighted score distribution used to scale the similarity distribution.
- Filtered network size is bounded by the filter parameters (e.g. sampling to 2000 compounds instead of the unfiltered 906,086 EcoCyc-derived compounds).

## Limitations

- The paper describes but never evaluates the thermodynamics and feasibility filters in isolation, so no quantitative accuracy criterion is available for them.
- Reactions for which eQuilibrator returns no ΔG fall outside its prediction scope and are left in the network, so the thermodynamics filter cannot comment on their feasibility.
- The metabolomics filter requires user-supplied adduct definitions and a tolerance; without them, no m/z matching can be performed.
- Pickaxe is run through a Python file or as a module from the MINE-Database directory due to relative imports — the README marks command-line usage as 'not recommended'.
- No publication metadata, version number, or changelog was found for the paper's code; Python 3.7+ is the only stated requirement.

## Evidence

- [abstract] The filters include chemical similarity to target molecules, metabolomics, thermodynamics, and reaction feasibility filters.: "The filters include chemical similarity to target molecules, metabolomics, thermodynamics, and reaction feasibility filters."
- [results] The thermodynamics filter was set to remove reactions with a physiological Gibbs free energy greater than 50 kJ/mol.: "The thermodynamics filter was set to remove reactions with a physiological Gibbs free energy greater than 50 kJ/mol."
- [results] These reactions were left to remain in the network, as the thermodynamics filter cannot comment on their feasibility.: "These reactions were left to remain in the network, as the thermodynamics filter cannot comment on their feasibility."
- [results] The similarity sampling filter utilizes a weighted distribution of the similarity scores calculated with a set of target compounds to sample a specified number of random compounds to be expanded.: "The similarity sampling filter utilizes a weighted distribution of the similarity scores calculated with a set of target compounds to sample a specified number of random compounds to be expanded."
- [results] The metabolomics filter computes possible m/z values for each predicted compound from user-supplied adducts and retains compounds whose adducted m/z matches an input experimental peak within a user-defined tolerance.: [evidence span withheld — anchor status: not_found]
- [readme] The thermodynamics filters require the use of eQuilibrator. This can be installed via conda: "The thermodynamics filters require the use of [eQuilibrator](https://equilibrator.readthedocs.io/en/latest/). This can be installed via conda"
- [readme] The feasibility filters require the use of DeepRFC, which can be installed by following the instructions found on their website.: "The feasibility filters require the use of DeepRFC, which can be installed by [following the instructions found on their website.](https://bitbucket.org/kaistsystemsbiology/deeprfc/src/master/)"
