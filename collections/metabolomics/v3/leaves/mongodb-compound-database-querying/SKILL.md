---
name: mongodb-compound-database-querying
description: Use when a Pickaxe network expansion (e.g. two-generation EcoCyc expansion
  with top 50 JN1224min rules) has stored its generated compounds and reactions in
  MongoDB and you must quantify per-filter annotation counts.
when_to_use_negative:
- The Pickaxe expansion has not been stored in a MongoDB database (e.g. Pickaxe was
  run writing only .tsv files) - there is no compound/reaction collection to query
- You need to generate novel compounds/reactions themselves rather than filter and
  count an already-expanded network stored in MongoDB
- The dataset is not an untargeted metabolomics peak list; the metabolomics filter
  queries only make sense against peak m/z values
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_0091
tools:
- name: MongoDB
  role: Stores the generated compounds and reactions from Pickaxe expansions; queried
    for per-filter annotation counts
  repo: https://www.mongodb.com/try/download/community
- name: PyMongo
  role: Python API used in the analysis notebook to query the Mongo database and count
    annotated peaks for each filter combination
  repo: https://pymongo.readthedocs.io/en/stable/
- name: Pickaxe
  role: Generates the network expansion whose compounds/reactions are persisted in
    MongoDB prior to querying
  repo: https://github.com/tyo-nu/MINE-Database
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:aafb2e95a00740cc4e17c1340e595a4a48e3bc6d8dc09250624bab3099dec9f6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepRFC
  - MongoDB
  - Pickaxe
  - PyMongo
  - eQuilibrator
  - minedatabase
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# mongodb-compound-database-querying

## Summary

Query a Pickaxe-generated compound/reaction network stored in MongoDB with PyMongo to count how many unannotated metabolomics peaks are matched under each filter combination (metabolomics, thermodynamics, reaction feasibility).

## When to use

Use when a Pickaxe network expansion (e.g. two-generation EcoCyc expansion with top 50 JN1224min rules) has stored its generated compounds and reactions in MongoDB and you must quantify per-filter annotation counts.

## When NOT to use

- The Pickaxe expansion has not been stored in a MongoDB database (e.g. Pickaxe was run writing only .tsv files) - there is no compound/reaction collection to query
- You need to generate novel compounds/reactions themselves rather than filter and count an already-expanded network stored in MongoDB
- The dataset is not an untargeted metabolomics peak list; the metabolomics filter queries only make sense against peak m/z values

## Inputs

- MongoDB database containing Pickaxe-generated compounds and reactions (results of a two-generation EcoCyc expansion with the top 50 JN1224min rules)
- Set of 200 unannotated peaks with m/z values between 150 and 300 Da from the E. coli untargeted metabolomics dataset
- Filter outputs: eQuilibrator physiological Gibbs free energies and DeepRFC reaction feasibility classifications

## Outputs

- Per-filter-combination counts of annotated (matched) peaks and generated compounds/compound peak pairs
- CSV count table (e.g. figure6_peak_annotations.csv)
- Rendered comparison plot (e.g. figure6_annotation_plot.png)

## How to apply

After the expansion run (compounds <600 Da from EcoCyc, two generations, top 50 JN1224min rules with 65% MetaCyc mapping) is stored in a local or remote MongoDB, connect via PyMongo and query the stored compounds and reactions per filter combination. For the metabolomics-only filter, keep compounds whose m/z matches one of the 200 unannotated peaks (m/z 150-300 Da); for the thermodynamics filter, additionally remove reactions with physiological Gibbs free energy >50 kJ/mol while retaining reactions without a computable Gibbs energy; for the reaction feasibility filter, use the DeepRFC feasibility results. Count annotated peaks for each combination in the analysis notebook (fig6_ecolyc_metabolomics/laptop/analysis/ecoli_metabolomics_analysis.ipynb style), write per-combination counts to a CSV table, and render the comparison plot. The rationale is that Pickaxe stores all generated compounds and reactions in the mongo database, so filtering and counting become database queries rather than re-running the expansion.

## Related tools

- **MongoDB** (Stores the generated compounds and reactions from Pickaxe expansions; queried for per-filter annotation counts) — https://www.mongodb.com/try/download/community
- **PyMongo** (Python API used in the analysis notebook to query the Mongo database and count annotated peaks for each filter combination) — https://pymongo.readthedocs.io/en/stable/
- **Pickaxe** (Generates the network expansion whose compounds/reactions are persisted in MongoDB prior to querying) — https://github.com/tyo-nu/MINE-Database

## Examples

```
from pymongo import MongoClient; db = MongoClient()['pickaxe_db']; annotated = [c for c in db.compounds.find({'generation': {'$lte': 2}}) if c['m/z'] in peak_mz_set]
```

## Evaluation signals

- The reproduced count table qualitatively matches the paper's finding: adding the reaction feasibility filter to the metabolomics filter reduces both compounds and compound peak pairs by roughly an order of magnitude while matching a similar number of unannotated peaks
- The thermodynamics-plus-metabolomics combination yields a similar annotation count to metabolomics alone (reactions without a computable Gibbs energy are retained)
- All queried compound documents fall within the expansion parameters: EcoCyc-derived, <600 Da, generation <= 2, rules from the top 50 JN1224min set
- Matched compounds correspond to the 200 input peaks with m/z 150-300 Da; no peak is counted more than once per combination

## Limitations

- eQuilibrator may not return a Gibbs free energy for reactions outside its prediction scope; the thermodynamics filter cannot comment on their feasibility, so they remain in the network
- Feasibility-filter computation is expensive enough that the paper repo ships speed-up scripts (feasibility_first_gen.py, feasibility_second_gen_gen2.py, get_physiological_dgs.py); naive queries over feasibility results can be slow
- No software version numbers for the queried packages are stated in the paper, complicating exact reproduction

## Evidence

- [readme] Results are stored in a mongo database. - Analysis code is found in fig6_ecocyc_metabolomics/laptop/analysis/ecoli_metabolomics_analysis.ipynb: "Results are stored in a mongo database.
- Analysis code is found in `fig6_ecocyc_metabolomics/laptop/analysis/ecoli_metabolomics_analysis.ipynb`"
- [results] 12 million new compounds being generated and inserted into a mongo database in approximately nine hours: "12 million new compounds being generated and inserted into a mongo database in approximately nine hours"
- [readme] PyMongo provides methods to interact with the database via python code.: "[PyMongo](https://pymongo.readthedocs.io/en/stable/) provides methods to interact with the database via python code."
- [other] Run the analysis notebook to query the Mongo database with PyMongo and count annotated peaks for each filter combination: [evidence span withheld — anchor status: not_found]
- [results] The thermodynamics filter was set to remove reactions with a physiological Gibbs free energy greater than 50 kJ/mol.: "The thermodynamics filter was set to remove reactions with a physiological Gibbs free energy greater than 50 kJ/mol."
