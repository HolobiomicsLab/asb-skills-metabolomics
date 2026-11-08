---
name: mongodb-network-results-storage
description: Use when running Pickaxe network expansions (e.g. YMDB one-generation
  expansion with top 500 JN1224min rules) whose results must be persisted in a database
  for downstream analysis notebooks instead of flat files.
when_to_use_negative:
- Small exploratory runs where .tsv file output suffices — the pickaxe_run_template
  notes results can alternatively be 'written to .tsv files'
- When no MongoDB server is available and installing one is not feasible; the README
  states 'These examples rely on using a mongo database, either remote or local'
- When the tool is being run only via the standalone command line 'not recommended'
  path without database storage
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3365
tools:
- name: MINE-Database (minedatabase python package)
  role: Package whose Pickaxe module writes expansion results into a specified MongoDB
    database during generation
  repo: https://github.com/tyo-nu/MINE-Database
- name: MongoDB (mongo database)
  role: Persistent store for generated compounds and reactions; installed locally
    or remotely per the README
  repo: https://www.mongodb.com/try/download/community
- name: PyMongo
  role: Python library providing methods to interact with the stored MongoDB results
  repo: https://pymongo.readthedocs.io/en/stable/
- name: Pickaxe
  role: Generates the compound/reaction network whose results are inserted into MongoDB
  repo: https://github.com/tyo-nu/MINE-Database
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:7d003868dbf7c5dfac0f77fb269dcb6fc00cc8dcee95def4e0533e36aa986743
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyo-nu/MINE-Database
  tools:
  - MINE-Database (minedatabase python package)
  - MongoDB
  - MongoDB (mongo database)
  - Pickaxe
  - PyMongo
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3365
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# mongodb-network-results-storage

## Summary

Store Pickaxe-generated network expansion results (novel compounds and reactions) in a MongoDB database via PyMongo, as done for the YMDB cutoff/sampling-filter experiments in the Pickaxe paper, so that large expansions (hundreds of thousands of compounds/reactions) persist for later notebook analysis.

## When to use

Use when running Pickaxe network expansions (e.g. YMDB one-generation expansion with top 500 JN1224min rules) whose results must be persisted in a database for downstream analysis notebooks instead of flat files.

## When NOT to use

- Small exploratory runs where .tsv file output suffices — the pickaxe_run_template notes results can alternatively be 'written to .tsv files'
- When no MongoDB server is available and installing one is not feasible; the README states 'These examples rely on using a mongo database, either remote or local'
- When the tool is being run only via the standalone command line 'not recommended' path without database storage

## Inputs

- Pickaxe run configuration (starting compounds, rules, cofactors, run options) from a Python file such as pickaxe_run_template.py
- Pickaxe-generated compounds and reactions from network expansion (e.g. YMDB <600 g/mol compounds, top 500 JN1224min rules, one generation)
- MongoDB instance, local or remote

## Outputs

- MongoDB database containing generated compounds and reactions for the expansion run
- Queryable collection used by analysis notebooks (e.g. tani_both_analysis.ipynb) to produce count tables such as fig4_target_network_counts.csv

## How to apply

Install MongoDB (Community installer or remote instance) and PyMongo, then run Pickaxe through a Python driver file specifying starting compounds, rules, cofactors, and a target database so compounds and reactions are inserted as the expansion proceeds. This is how the Figure 4 YMDB cutoff/sampling experiments stored their generation results ('Results are stored in a mongo database'). MongoDB scales to the paper's demonstrated workload of 12 million compounds inserted in about nine hours, so it is appropriate for unfiltered one-generation expansions (e.g. YMDB producing 575,262 compounds and 892,343 reactions). After generation, analysis notebooks (e.g. fig4_gen/analysis/tani_both_analysis.ipynb) query the stored database to tabulate target and network compound counts. Correct application is judged by confirming the stored compound/reaction counts match the paper's reported values and that the analysis notebooks can retrieve the results.

## Related tools

- **MINE-Database (minedatabase python package)** (Package whose Pickaxe module writes expansion results into a specified MongoDB database during generation) — https://github.com/tyo-nu/MINE-Database
- **MongoDB (mongo database)** (Persistent store for generated compounds and reactions; installed locally or remotely per the README) — https://www.mongodb.com/try/download/community
- **PyMongo** (Python library providing methods to interact with the stored MongoDB results) — https://pymongo.readthedocs.io/en/stable/
- **Pickaxe** (Generates the compound/reaction network whose results are inserted into MongoDB) — https://github.com/tyo-nu/MINE-Database

## Examples

```
from pymongo import MongoClient; db = MongoClient()['ymdb_fig4']  # Pickaxe run inserts compounds/reactions into this database
```

## Evaluation signals

- Stored counts match the paper's reported expansion sizes, e.g. YMDB 1,035 starting compounds yielding 575,262 compounds and 892,343 reactions
- The analysis notebook (fig4_gen/analysis/tani_both_analysis.ipynb) can query the database and reproduce Figure 4 target/network counts per cutoff and weighting
- Database insertion completes without failure at scale — the article demonstrated '12 million new compounds being generated and inserted into a mongo database in approximately nine hours'
- PyMongo queries return compounds/reactions with the expected identifiers and structure produced by Pickaxe generation

## Limitations

- A running MongoDB server (local or remote) is required; the README directs users to download the installer and follow its instructions
- Pickaxe command-line use is flagged 'not recommended'; running through a Python file is the supported way to specify database storage
- No software version numbers for minedatabase or MongoDB are stated in the context; only 'Python 3.7 or higher' is given

## Evidence

- [results] MongoDB (mongo database): "12 million new compounds being generated and inserted into a mongo database in approximately nine hours"
- [readme] results stored in mongo: "Results are stored in a mongo database."
- [readme] after running, results stored in database: [evidence span withheld — anchor status: external_doc]
- [readme] PyMongo interaction: "PyMongo provides methods to interact with the database via python code."
- [results] YMDB expansion counts: [evidence span withheld — anchor status: not_found]
