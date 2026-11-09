---
name: mongo-database-result-storage
description: Use when Pickaxe expansion runs must be persisted as compounds/reactions
  in a database (rather than .tsv files) and later queried to tabulate per-database
  counts for validation against published results.
when_to_use_negative:
- When only small, easily-inspected result sets are needed — Pickaxe can write results
  directly to .tsv files without any database
- When the goal is the network expansion itself rather than persistent storage/retrieval
  of already-generated compounds and reactions
- When no MongoDB installation (remote or local) is available and installing one is
  not feasible for the environment
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MongoDB
  role: Database into which generated compounds and reactions from Pickaxe runs are
    inserted and queried for per-database counts
  repo: https://www.mongodb.com/try/download/community
- name: PyMongo
  role: Python library providing methods to interact with the mongo database via Python
    code
  repo: https://pymongo.readthedocs.io/en/stable/
- name: Pickaxe
  role: Generates the novel compounds and reactions that are stored in the mongo database
  repo: https://github.com/tyo-nu/MINE-Database
- name: minedatabase
  role: Python package (installed via pip) within which Pickaxe runs are executed
    before results are stored in the database
  repo: https://pypi.org/project/minedatabase/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:166d21a25948f626ca201e34e797937dff188169b38c51fa796cfda1e6bf439c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MongoDB
  - MongoDB (mongo database)
  - Pickaxe
  - PyMongo
  - minedatabase
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# mongo-database-result-storage

## Summary

Store Pickaxe-generated compounds and reactions in a MongoDB instance (accessed via PyMongo) and query it to count starting compounds, generated reactions, and generated compounds, as done for the KEGG/EcoCyc/YMDB expansion validation.

## When to use

Use when Pickaxe expansion runs must be persisted as compounds/reactions in a database (rather than .tsv files) and later queried to tabulate per-database counts for validation against published results.

## When NOT to use

- When only small, easily-inspected result sets are needed — Pickaxe can write results directly to .tsv files without any database
- When the goal is the network expansion itself rather than persistent storage/retrieval of already-generated compounds and reactions
- When no MongoDB installation (remote or local) is available and installing one is not feasible for the environment

## Inputs

- Pickaxe one-generation expansion results (generated compounds and reactions from KEGG 12,731 / EcoCyc 1,923 / YMDB 1,035 starting compounds <600 g/mol)
- MongoDB instance (local or remote) accessible from Python
- PyMongo connection from the minedatabase package environment

## Outputs

- Mongo database containing stored generated compounds and reactions per Pickaxe run
- Per-database counts (starting compounds, generated reactions, generated compounds) written to results_table.csv

## How to apply

Install MongoDB locally or remotely (per the README, MongoDB Community installer) and interact with it from Python via PyMongo, which the README names as the interface for database access. When running Pickaxe through a Python file (the recommended route, e.g. pickaxe_run_template.py), specify the database information and run options so that generated reactions and compounds are inserted into the database instead of only being written to .tsv files; the paper reports ~12 million new compounds inserted into a mongo database in about nine hours. After the run completes, query the mongo database to count starting compounds, generated reactions, and generated compounds per database (KEGG Release 99.0, EcoCyc 25.1, YMDB 2.0 expansions filtered to <600 g/mol, top 500 JN1224min rules, one generation), then write the per-database counts to results_table.csv for comparison with the paper's reported values.

## Related tools

- **MongoDB** (Database into which generated compounds and reactions from Pickaxe runs are inserted and queried for per-database counts) — https://www.mongodb.com/try/download/community
- **PyMongo** (Python library providing methods to interact with the mongo database via Python code) — https://pymongo.readthedocs.io/en/stable/
- **Pickaxe** (Generates the novel compounds and reactions that are stored in the mongo database) — https://github.com/tyo-nu/MINE-Database
- **minedatabase** (Python package (installed via pip) within which Pickaxe runs are executed before results are stored in the database) — https://pypi.org/project/minedatabase/

## Examples

```
python -m minedatabase.pickaxe -C ./data/metacyc_generalized_rules.tsv -r ./data/metacyc_coreactants.tsv -g 1 -c ../example_data/iML1515_ecoli_GEM.csv
```

## Evaluation signals

- KEGG expansion counts must match: 12,731 starting compounds, 13,759,651 generated reactions, 9,562,940 generated compounds
- EcoCyc expansion counts must match: 1,923 starting compounds, 1,388,653 reactions, 906,086 compounds
- YMDB expansion counts must match: 1,035 starting compounds, 892,343 reactions, 575,262 compounds
- results_table.csv contains one row per database (KEGG, EcoCyc, YMDB) with the three count columns matching the paper's results section

## Limitations

- Storing very large expansions is time-consuming: ~12 million new compounds were inserted into a mongo database in approximately nine hours
- The pickaxe_paper figure pipelines depend on a mongo database being available (local or remote), so database installation is a prerequisite
- No software version numbers for the packages are stated in the methods; only Python 3.7 or higher is specified

## Evidence

- [methods] Use Pickaxe (from the minedatabase package) to expand each filtered database for one generation, storing generated reactions and compounds in a mongo database via PyMongo: [evidence span withheld — anchor status: not_found]
- [results] 12 million new compounds being generated and inserted into a mongo database in approximately nine hours: "12 million new compounds being generated and inserted into a mongo database in approximately nine hours"
- [readme] PyMongo provides methods to interact with the database via python code.: "PyMongo provides methods to interact with the database via python code."
- [readme] After running the results are stored in a specified database or written to .tsv files.: [evidence span withheld — anchor status: external_doc]
- [readme] These examples rely on using a mongo database, either remote or local.: "These examples rely on using a mongo database, either remote or local."
