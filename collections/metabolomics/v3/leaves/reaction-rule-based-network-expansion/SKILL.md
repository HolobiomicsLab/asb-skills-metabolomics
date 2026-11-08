---
name: reaction-rule-based-network-expansion
description: Use when you need to enumerate predicted promiscuous-enzyme reactions
  and novel compounds from a compound database (KEGG, EcoCyc, YMDB) or metabolite
  list, e.g. for pathway design or metabolomics peak annotation.
when_to_use_negative:
- Input is already a pre-computed reaction network or you only need to browse existing
  MINE databases rather than generate novel reactions
- You require web-interface-only tools or cannot install MongoDB/minedatabase locally
  (Pickaxe requires Python 3.7+, MongoDB, and pip install minedatabase)
- Your rules fall outside MetaCyc-derived enzymatic transformations and you cannot
  define custom rules or cofactor coreactant sets
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3314
tools:
- name: Pickaxe
  role: Core tool that iteratively applies reaction rules to metabolites to generate
    novel compounds and reactions for the network
  repo: https://github.com/tyo-nu/MINE-Database
- name: MINE-Database (minedatabase)
  role: Python package (2.0.0) containing Pickaxe; provides pickaxe.py CLI and pickaxe_run_template.py
    for configuring runs
  repo: https://github.com/tyo-nu/MINE-Database
- name: MongoDB / PyMongo
  role: Stores generated compounds and reactions from expansion runs; queried with
    PyMongo to count results
- name: JN1224min ruleset
  role: Prepackaged MetaCyc-derived reaction rules (top 500 = 94% MetaCyc mapped;
    top 50 = 65% mapped) applied to starting compounds
  repo: https://github.com/tyo-nu/MINE-Database
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_007
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:f047e0a498f17455b805224153a1173de64e08f44d038ba947549fc8bd826f57
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyo-nu/MINE-Database
  tools:
  - DeepRFC
  - JN1224min ruleset
  - MINE-Database (minedatabase)
  - MongoDB / PyMongo
  - Pickaxe
  - PyMongo
  - eQuilibrator
  - minedatabase
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# reaction-rule-based-network-expansion

## Summary

Iteratively apply reaction rules (e.g. the prepackaged JN1224min ruleset derived from MetaCyc) to a set of metabolites with Pickaxe to generate a network of novel compounds and reactions, storing results in a MongoDB database.

## When to use

Use when you need to enumerate predicted promiscuous-enzyme reactions and novel compounds from a compound database (KEGG, EcoCyc, YMDB) or metabolite list, e.g. for pathway design or metabolomics peak annotation.

## When NOT to use

- Input is already a pre-computed reaction network or you only need to browse existing MINE databases rather than generate novel reactions
- You require web-interface-only tools or cannot install MongoDB/minedatabase locally (Pickaxe requires Python 3.7+, MongoDB, and pip install minedatabase)
- Your rules fall outside MetaCyc-derived enzymatic transformations and you cannot define custom rules or cofactor coreactant sets

## Inputs

- Compound database set (KEGG Release 99.0, EcoCyc Version 25.1, or YMDB Version 2.0) filtered to compounds <600 g/mol molecular weight
- Reaction rules from the prepackaged JN1224min ruleset (top 500 or top 50 rules) or user-defined custom rules
- Optional target compounds for similarity filters (e.g. 250 KEGG compounds with 4-9 carbons absent from YMDB)
- Optional unannotated metabolomics peaks (e.g. 200 peaks with m/z 150-300 Da from an E. coli dataset) for the metabolomics filter

## Outputs

- Mongo database of generated compounds and reactions (queried via PyMongo)
- Per-database counts of starting compounds, generated compounds, and generated reactions (e.g. results_table.csv)
- Expanded reaction network usable for pathway design or metabolomics peak annotation

## How to apply

Specify starting compounds (e.g. KEGG Release 99.0, EcoCyc 25.1, or YMDB 2.0 compounds filtered to <600 g/mol) and reaction rules, selecting the top 500 (94% MetaCyc mapped) or top 50 (65% MetaCyc mapped) JN1224min rules. Run Pickaxe from the minedatabase package for one or more generations, storing generated compounds and reactions in a mongo database via PyMongo, optionally applying on-the-fly filters (similarity cutoff/sampling, atomic composition, metabolomics, thermodynamics, feasibility) between generations. Count starting compounds, generated compounds, and generated reactions per database by querying the mongo database, e.g. one generation on KEGG yields 12,731 starting compounds, 9,562,940 compounds, and 13,759,651 reactions. Use the pickaxe_run_template.py framework or the paper's per-figure generation scripts (fig4_YMDB_cutoff/, fig6_ecocyc_metabolomics/) to configure runs.

## Related tools

- **Pickaxe** (Core tool that iteratively applies reaction rules to metabolites to generate novel compounds and reactions for the network) — https://github.com/tyo-nu/MINE-Database
- **MINE-Database (minedatabase)** (Python package (2.0.0) containing Pickaxe; provides pickaxe.py CLI and pickaxe_run_template.py for configuring runs) — https://github.com/tyo-nu/MINE-Database
- **MongoDB / PyMongo** (Stores generated compounds and reactions from expansion runs; queried with PyMongo to count results)
- **JN1224min ruleset** (Prepackaged MetaCyc-derived reaction rules (top 500 = 94% MetaCyc mapped; top 50 = 65% mapped) applied to starting compounds) — https://github.com/tyo-nu/MINE-Database

## Examples

```
python -m minedatabase.pickaxe -C ./data/metacyc_generalized_rules.tsv -r ./data/metacyc_coreactants.tsv -g 1 -c ../example_data/iML1515_ecoli_GEM.csv
```

## Evaluation signals

- Per-database counts match the paper: KEGG 12,731 starting compounds → 9,562,940 compounds / 13,759,651 reactions; EcoCyc 1,923 → 906,086 / 1,388,653; YMDB 1,035 → 575,262 / 892,343
- Run configuration matches stated parameters: one generation, top 500 JN1224min rules (94% MetaCyc mapped), compounds <600 g/mol
- Generated compounds and reactions are successfully inserted into the mongo database (approx. 12 million compounds in ~9 hours per the paper's benchmark)
- Atomic composition constraint holds: non-cofactor generated molecules contain fewer than twelve carbon atoms when that filter is enabled
- results_table.csv (or equivalent per-run CSV) populated with starting/compound/reaction counts per database

## Limitations

- Network size grows exponentially with generations without filters — the similarity cutoff or sampling filters are needed to keep the network tractable (sampling filter reacts a fixed number, e.g. 2000, compounds per generation)
- eQuilibrator may not return a Gibbs free energy for reactions outside the scope of its predictions; such reactions must be retained in the network
- Rules are limited by the JN1224min coverage — only 94% of MetaCyc mapped in the top 500, 65% in the top 50 — so unmapped reactions are not predicted

## Evidence

- [abstract] This software iteratively applies reaction rules to a set of metabolites to generate novel reactions.: "This software iteratively applies reaction rules to a set of metabolites to generate novel reactions."
- [results] Database filter and rule selection: "Each database was filtered to only contain compounds whose molecular weights are less than 600 g/mol."
- [results] Rule selection fraction: "The top 500 rules (94% of MetaCyc mapped) from the JN1224min ruleset were applied to the starting compounds for one generation."
- [readme] Storage requirement: "These examples rely on using a mongo database, either remote or local."
- [readme] Run framework: [evidence span withheld — anchor status: external_doc]
- [discussion] Network generation prerequisites: "A network expansion requires the specification of starting compounds and reaction rules"
