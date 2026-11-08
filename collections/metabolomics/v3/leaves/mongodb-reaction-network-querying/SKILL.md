---
name: mongodb-reaction-network-querying
description: Reference for practitioners whose Pickaxe network-expansion run has written
  results into a MongoDB collection and who need to load, query, or iterate those
  reactions/compounds in Python for downstream analysis or filtering.
when_to_use_negative:
- Run output was written to .tsv files only — parse those directly instead of standing
  up a MongoDB instance.
- You need only a single-generation, small network where an in-memory list of reactions
  from a Pickaxe script would be simpler than a database round-trip.
- The downstream task is rule construction or starting-compound selection, which occurs
  before any results exist in the database.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_0091
tools:
- name: Pickaxe
  role: generates the novel reaction network whose results are stored in the MongoDB
    database
  repo: https://github.com/tyo-nu/MINE-Database
- name: PyMongo
  role: Python client used to query the MongoDB collections holding the run's compounds
    and reactions
  repo: https://pymongo.readthedocs.io/en/stable/
- name: MongoDB
  role: persistent store into which Pickaxe inserts generated compounds and reactions
  repo: https://www.mongodb.com/try/download/community
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:9fd3ac6cc96dd67b800d2125fa35874af41d1102a22de34d3d3ba2e436bc7357
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyo-nu/MINE-Database
  tools:
  - DeepRFC
  - MongoDB
  - Pickaxe
  - PyMongo
  - eQuilibrator
  - minedatabase
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# mongodb-reaction-network-querying

## Summary

Store and query Pickaxe-generated novel compounds and reactions in a MongoDB instance via PyMongo, enabling downstream filtering and analysis workflows (e.g. metabolomics-peak annotation, target-compound expansion) that read generation results back from a persistent database.

## When to use

Reference for practitioners whose Pickaxe network-expansion run has written results into a MongoDB collection and who need to load, query, or iterate those reactions/compounds in Python for downstream analysis or filtering.

## When NOT to use

- Run output was written to .tsv files only — parse those directly instead of standing up a MongoDB instance.
- You need only a single-generation, small network where an in-memory list of reactions from a Pickaxe script would be simpler than a database round-trip.
- The downstream task is rule construction or starting-compound selection, which occurs before any results exist in the database.

## Inputs

- Pickaxe run results previously inserted into MongoDB (compounds and reactions collections)
- MongoDB connection details (local or remote database/collection names)
- Pickaxe run options specifying database storage instead of .tsv file output

## Outputs

- Python-side lists/cursors of reaction and compound documents (e.g. for ΔG computation or metabolomics m/z matching)
- Query results that feed downstream filter steps (thermodynamics, feasibility, similarity)

## How to apply

Configure Pickaxe (from the minedatabase package) with database information so a run's results are inserted into a local or remote MongoDB rather than only written to .tsv files; the README notes runs can be 'stored in a specified database or written to .tsv files'. Install MongoDB (community installer) and PyMongo (pip install pymongo) to obtain the python client for querying. After generation, open a PyMongo client pointed at the database and collection holding the run's compounds and reactions, and iterate the documents to extract the fields needed by downstream steps — e.g. reaction SMILES for eQuilibrator physiological ΔG computation, or compound m/z for metabolomics filtering. Because reaction networks are large (e.g. 12.7 million compounds / 13.8 million reactions from one KEGG expansion), prefer filtered queries over full-collection scans when selecting subsets for analysis. Validate retrieval by confirming the queried collections contain the expected number of documents relative to the run log and that each reaction document carries the fields required by the next pipeline step.

## Related tools

- **Pickaxe** (generates the novel reaction network whose results are stored in the MongoDB database) — https://github.com/tyo-nu/MINE-Database
- **PyMongo** (Python client used to query the MongoDB collections holding the run's compounds and reactions) — https://pymongo.readthedocs.io/en/stable/
- **MongoDB** (persistent store into which Pickaxe inserts generated compounds and reactions) — https://www.mongodb.com/try/download/community

## Examples

```
from pymongo import MongoClient; client = MongoClient('localhost', 27017); db = client['pickaxe_run']; reactions = db['reactions'].find({})
```

## Evaluation signals

- Queried reaction/compound collections contain the number of documents reported by the Pickaxe run (e.g. paper reports 12,731 starting compounds expanding to 9,562,940 compounds and 13,759,651 reactions for KEGG).
- Each retrieved reaction document carries the SMILES/substrate-product fields needed by downstream eQuilibrator or DeepRFC filter steps.
- Subset queries (e.g. filtering to reactions from a specific generation) return fewer documents than the unfiltered collection and remain consistent across repeated queries.
- The paper's figure-analysis notebooks (e.g. fig4_gen/analysis/tani_both_analysis.ipynb) explicitly note 'Results are stored in a mongo database', so reproduce that reading path when re-running them.

## Limitations

- Results stored in MongoDB require a running server — local or remote — which adds deployment overhead not needed for .tsv output.
- Full networks are very large (12 million new compounds inserted into a mongo database in approximately nine hours for one KEGG run), so unfiltered scans can be slow.
- Querying assumes the Pickaxe run itself already inserted the documents; it does not replace the generation step.

## Evidence

- [readme] After running the results are stored in a specified database or written to .tsv files.: [evidence span withheld — anchor status: external_doc]
- [readme] These examples rely on using a mongo database, either remote or local. To install this database, download the installer and follow the instructions.: "These examples rely on using a mongo database, either remote or local. To install this database, [download the installer](https://www.mongodb.com/try/download/community) and follow the instructions."
- [readme] [PyMongo](https://pymongo.readthedocs.io/en/stable/) provides methods to interact with the database via python code.: "[PyMongo](https://pymongo.readthedocs.io/en/stable/) provides methods to interact with the database via python code."
- [results] 12 million new compounds being generated and inserted into a mongo database in approximately nine hours: "12 million new compounds being generated and inserted into a mongo database in approximately nine hours"
- [readme] The Analysis code can be sped up using `feasibility_first_gen.py`...: "The Analysis code can be sped up using `feasibility_first_gen.py`, `feasibility_second_gen_gen2.py`, and `get_physiological_dgs.py`"
