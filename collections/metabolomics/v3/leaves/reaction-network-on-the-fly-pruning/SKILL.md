---
name: reaction-network-on-the-fly-pruning
description: Use when running a Pickaxe/MINE-Database network expansion and you need
  to prune the growing network on-the-fly with property, similarity, metabolomics,
  or thermodynamics filters before the next rule-application generation.
when_to_use_negative:
- The network is already fully generated/pre-computed and you only need post-hoc analysis
  rather than pruning during expansion
- You require thermodynamic feasibility verdicts for reactions outside eQuilibrator's
  scope — the paper notes eQuilibrator may not return a Gibbs free energy, and such
  reactions remain in the network
- Your goal is authoring custom reaction rules rather than controlling expansion size
  with filters
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: Pickaxe
  role: Generates the novel reaction network and applies the on-the-fly pruning filters
    during iterative expansion
  repo: https://github.com/tyo-nu/MINE-Database
- name: MINE-Database (minedatabase)
  role: Python package providing Pickaxe, filter interfaces, and database/TSV output
    for network runs
  repo: https://github.com/tyo-nu/MINE-Database
- name: eQuilibrator
  role: Computes physiological Gibbs free energies consumed by the thermodynamics
    filter
  repo: https://equilibrator.readthedocs.io/en/latest/
- name: DeepRFC
  role: Provides reaction feasibility classifications used by the reaction feasibility
    filter
  repo: https://bitbucket.org/kaistsystemsbiology/deeprfc/src/master/
- name: MongoDB
  role: Stores the filtered/expanded compounds and reactions; accessed via PyMongo
  repo: https://www.mongodb.com/try/download/community
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:46688201a0bc5c7a5bb715e9e503b8f51edef2255a86aed59edc9682f2b211f4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyo-nu/MINE-Database
  tools:
  - DeepRFC
  - MINE-Database (minedatabase)
  - MongoDB
  - Pickaxe
  - eQuilibrator
  - minedatabase
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# reaction-network-on-the-fly-pruning

## Summary

Applies Pickaxe's on-the-fly compound and reaction filters (chemical similarity cutoff/sampling, metabolomics m/z matching, thermodynamics via eQuilibrator, reaction feasibility via DeepRFC, and compound property filters such as MW < 600 g/mol and atomic composition < 12 carbons for non-cofactors) during iterative rule-based network expansion, so each generation only expands compounds that survive pruning.

## When to use

Use when running a Pickaxe/MINE-Database network expansion and you need to prune the growing network on-the-fly with property, similarity, metabolomics, or thermodynamics filters before the next rule-application generation.

## When NOT to use

- The network is already fully generated/pre-computed and you only need post-hoc analysis rather than pruning during expansion
- You require thermodynamic feasibility verdicts for reactions outside eQuilibrator's scope — the paper notes eQuilibrator may not return a Gibbs free energy, and such reactions remain in the network
- Your goal is authoring custom reaction rules rather than controlling expansion size with filters

## Inputs

- Starting-compound CSV with id and SMILES columns (e.g. from KEGG, EcoCyc, or YMDB, optionally pre-filtered to MW < 600 g/mol)
- Reaction rules TSV (e.g. JN1224min generalized rules from MetaCyc) and coreactants/cofactors TSV
- Target compound set for chemical similarity or sampling filters (e.g. 250 KEGG compounds with 4-9 carbons not in YMDB)
- Unannotated metabolomics peak list with m/z values (e.g. 200 peaks, m/z 150-300 Da) for the metabolomics filter
- Pickaxe run options: number of generations and filter thresholds

## Outputs

- Pruned compound/reaction network for each generation, stored in a mongo database or written as TSV files
- Per-compound kept/removed decision table per filter (e.g. filtered_compounds.csv)
- Expanded network of novel compounds and reactions surviving all filters

## How to apply

Configure filters alongside the starting compounds, reaction rules (e.g. the prepackaged JN1224min ruleset from MetaCyc), and cofactor list in a Pickaxe run, then apply them between expansion generations. Compound property filters operate by range specification: the molecular weight filter retains compounds whose MW falls under a range threshold (the paper uses MW < 600 g/mol), while the atomic composition filter specifies per-element ranges (the paper limits carbon atoms in non-cofactor molecules to fewer than twelve). The similarity filter removes any compound whose maximum similarity score to the target set is below a specified threshold, or alternatively samples a fixed number of compounds (e.g. 2000) from a similarity-score distribution scaled to the zeroth, third, or sixth power. The metabolomics filter keeps only compounds whose m/z matches peaks in the unannotated dataset (e.g. 200 peaks with m/z 150-300 Da), and the thermodynamics filter removes reactions with physiological Gibbs free energy greater than 50 kJ/mol while retaining reactions for which eQuilibrator returns no energy. Persist results to a mongo database or TSV files and record per-compound kept/removed decisions for every filter.

## Related tools

- **Pickaxe** (Generates the novel reaction network and applies the on-the-fly pruning filters during iterative expansion) — https://github.com/tyo-nu/MINE-Database
- **MINE-Database (minedatabase)** (Python package providing Pickaxe, filter interfaces, and database/TSV output for network runs) — https://github.com/tyo-nu/MINE-Database
- **eQuilibrator** (Computes physiological Gibbs free energies consumed by the thermodynamics filter) — https://equilibrator.readthedocs.io/en/latest/
- **DeepRFC** (Provides reaction feasibility classifications used by the reaction feasibility filter) — https://bitbucket.org/kaistsystemsbiology/deeprfc/src/master/
- **MongoDB** (Stores the filtered/expanded compounds and reactions; accessed via PyMongo) — https://www.mongodb.com/try/download/community

## Examples

```
python -m minedatabase.pickaxe -C ./data/metacyc_generalized_rules.tsv -r ./data/metacyc_coreactants.tsv -g 1 -c ../example_data/iML1515_ecoli_GEM.csv
```

## Evaluation signals

- Every input compound has a kept/removed decision under both compound property filters, and each removed compound violates a stated threshold (MW >= 600 g/mol, or >= 12 carbons for non-cofactors)
- Any compound whose maximum similarity score to the target set falls below the specified threshold is absent from the expanded network
- All retained reactions have physiological Gibbs free energy <= 50 kJ/mol, while reactions with no computable energy remain in the network
- Kept compounds under the metabolomics filter all have an m/z matching a peak in the unannotated dataset
- Sampling filters return exactly the requested number of compounds (e.g. 2000) drawn from the specified similarity-score power distribution

## Limitations

- eQuilibrator does not return a Gibbs free energy for some reactions (outside its prediction scope), so the thermodynamics filter cannot comment on their feasibility and they remain in the network
- The thermodynamics filter requires a separate eQuilibrator conda install and the feasibility filter requires DeepRFC, adding dependency complexity
- Network growth is exponential with generations, so filters trade prediction completeness for runtime tractability (the paper benchmarks exponential growth of runtimes)
- The similarity sampling filter uses random sampling from a weighted distribution, so results are not deterministic unless seeds are fixed

## Evidence

- [abstract] The filters include chemical similarity to target molecules, metabolomics, thermodynamics, and reaction feasibility filters.: "The filters include chemical similarity to target molecules, metabolomics, thermodynamics, and reaction feasibility filters."
- [results] if the compound's maximum similarity score is below a specified threshold it is removed from the network: [evidence span withheld — anchor status: not_found]
- [results] an atomic composition filter was used to limit the number of carbon atoms contained in non-cofactor molecules to fewer than twelve: "an atomic composition filter was used to limit the number of carbon atoms contained in non-cofactor molecules to fewer than twelve"
- [results] The thermodynamics filter was set to remove reactions with a physiological Gibbs free energy greater than 50 kJ/mol.: "The thermodynamics filter was set to remove reactions with a physiological Gibbs free energy greater than 50 kJ/mol."
- [results] These reactions were left to remain in the network, as the thermodynamics filter cannot comment on their feasibility.: "These reactions were left to remain in the network, as the thermodynamics filter cannot comment on their feasibility."
- [readme] Pickaxe applies reaction rules, representing reaction transformation patterns, to a list of user-specified compounds in order to predict reactions.: [evidence span withheld — anchor status: external_doc]
- [results] it is possible that eQuilibrator does not return a Gibbs free energy for a reaction, as the reaction falls outside the scope of its predictions: "it is possible that eQuilibrator does not return a Gibbs free energy for a reaction, as the reaction falls outside the scope of its"
