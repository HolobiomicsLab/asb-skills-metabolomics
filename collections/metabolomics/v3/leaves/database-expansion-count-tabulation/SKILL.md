---
name: database-expansion-count-tabulation
description: Use when the goal is to quantify how many novel compounds and reactions
  a reaction-rule network expansion (e.g., top 500 JN1224min rules, one generation,
  compounds <600 g/mol) produces from a compound database.
when_to_use_negative:
- The task targets filtered or sampled networks (e.g., similarity cutoff, similarity
  sampling, metabolomics, or thermodynamics filters) rather than a full unfiltered
  one-generation expansion whose counts should match the paper's results table
- The goal is multi-generation expansion or rule sets other than the top 500 JN1224min
  rules (e.g., the top 50 rules / two-generation EcoCyc run in the paper), since counts
  will not match the reported values
- Only pathway design or metabolomics peak annotation is needed — Pickaxe supports
  those applications, but this skill specifically produces count tabulation for expansion
  validation
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3361
tools:
- name: Pickaxe
  role: Applies reaction rules iteratively to the filtered compound sets for one generation
    to generate novel reactions and compounds
  repo: https://github.com/tyo-nu/MINE-Database
- name: minedatabase
  role: Python package (2.0.0) that contains Pickaxe and provides the run framework
    for expansion runs
  repo: https://pypi.org/project/minedatabase/2.0.0/
- name: MongoDB (mongo database)
  role: Stores the generated compounds and reactions; queried via PyMongo to produce
    per-database counts
- name: Python
  role: Programming language for the run scripts and count queries (requires Python
    3.7 or higher)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:8e105c1745f6549a3388fdda4e8a4b98a5c9252a19c250a0d3b34b7e0ae73943
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyo-nu/MINE-Database
  tools:
  - MongoDB (mongo database)
  - Pickaxe
  - Python
  - minedatabase
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# database-expansion-count-tabulation

## Summary

Reproduces one-generation rule-based expansion of biological compound databases (KEGG, EcoCyc, YMDB) with Pickaxe and tabulates per-database counts of starting compounds, generated compounds, and generated reactions into a results table for validation against published values.

## When to use

Use when the goal is to quantify how many novel compounds and reactions a reaction-rule network expansion (e.g., top 500 JN1224min rules, one generation, compounds <600 g/mol) produces from a compound database.

## When NOT to use

- The task targets filtered or sampled networks (e.g., similarity cutoff, similarity sampling, metabolomics, or thermodynamics filters) rather than a full unfiltered one-generation expansion whose counts should match the paper's results table
- The goal is multi-generation expansion or rule sets other than the top 500 JN1224min rules (e.g., the top 50 rules / two-generation EcoCyc run in the paper), since counts will not match the reported values
- Only pathway design or metabolomics peak annotation is needed — Pickaxe supports those applications, but this skill specifically produces count tabulation for expansion validation

## Inputs

- Compound sets from KEGG (Release 99.0), EcoCyc (Version 25.1), and YMDB (Version 2.0)
- Prepackaged JN1224min reaction ruleset (top 500 rules, 94% MetaCyc mapped)
- Cofactor list for Pickaxe runs
- MongoDB instance (local or remote) for storing expansion results

## Outputs

- Per-database counts of starting compounds, generated compounds, and generated reactions
- results_table.csv with one row per database (KEGG, EcoCyc, YMDB)
- Mongo database containing generated compounds and reactions

## How to apply

Download the compound sets for KEGG (Release 99.0), EcoCyc (Version 25.1), and YMDB (Version 2.0) and filter each to compounds with molecular weight under 600 g/mol. Select the top 500 rules (94% of MetaCyc mapped) from the prepackaged JN1224min ruleset and run Pickaxe (via the minedatabase package) for exactly one generation per database, storing generated reactions and compounds in a mongo database through PyMongo (a run of this scale inserts ~12 million compounds in roughly nine hours). Configure starting compounds, rules, and cofactors in a Python run file following the pickaxe_run_template.py framework rather than the command line, which the README marks as not recommended. Query each mongo database to count starting compounds, generated reactions, and generated compounds per database, then write the per-database counts to results_table.csv. Treat the published values as the acceptance criteria: KEGG 12,731 starting / 13,759,651 reactions / 9,562,940 compounds; EcoCyc 1,923 / 1,388,653 / 906,086; YMDB 1,035 / 892,343 / 575,262.

## Related tools

- **Pickaxe** (Applies reaction rules iteratively to the filtered compound sets for one generation to generate novel reactions and compounds) — https://github.com/tyo-nu/MINE-Database
- **minedatabase** (Python package (2.0.0) that contains Pickaxe and provides the run framework for expansion runs) — https://pypi.org/project/minedatabase/2.0.0/
- **MongoDB (mongo database)** (Stores the generated compounds and reactions; queried via PyMongo to produce per-database counts)
- **Python** (Programming language for the run scripts and count queries (requires Python 3.7 or higher))

## Examples

```
python -m minedatabase.pickaxe -C ./data/metacyc_generalized_rules.tsv -r ./data/metacyc_coreactants.tsv -g 1 -c ../example_data/iML1515_ecoli_GEM.csv
```

## Evaluation signals

- Per-database starting-compound counts must equal 12,731 (KEGG), 1,923 (EcoCyc), and 1,035 (YMDB) after <600 g/mol filtering
- Generated compound counts must equal 9,562,940 (KEGG), 906,086 (EcoCyc), and 575,262 (YMDB)
- Generated reaction counts must equal 13,759,651 (KEGG), 1,388,653 (EcoCyc), and 892,343 (YMDB); reactions should exceed compounds in each database
- results_table.csv must contain exactly one row per database with all three counts matching the paper's results section
- The mongo database should hold the full expansion (~12 million compounds for the KEGG-scale run), confirming storage completed before counting

## Limitations

- A run of this scale takes substantial time — about nine hours to generate and insert 12 million compounds into a mongo database
- No Pickaxe software version number is stated in the paper (only the Python 3.7+ requirement and minedatabase 2.0.0), so exact version reproduction may be ambiguous
- No version or accession is specified for the MetaCyc source of the reaction rules
- The command-line interface is explicitly not recommended; the README directs users to the pickaxe_run_template.py framework instead

## Evidence

- [results] Each database was filtered to only contain compounds whose molecular weights are less than 600 g/mol.: "Each database was filtered to only contain compounds whose molecular weights are less than 600 g/mol."
- [results] Top 500 rules applied for one generation: "The top 500 rules (94% of MetaCyc mapped) from the JN1224min ruleset were applied to the starting compounds for one generation."
- [results] Compounds inserted into mongo database: "12 million new compounds being generated and inserted into a mongo database in approximately nine hours"
- [readme] README on run framework: [evidence span withheld — anchor status: external_doc]
- [readme] README on results storage: "Results are stored in a mongo database."
