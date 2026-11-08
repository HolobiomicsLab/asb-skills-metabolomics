---
name: compound-filter-table-export
description: Use when a Pickaxe expansion generation has applied a similarity cutoff
  or similarity sampling filter and you must persist the retained compounds with per-compound
  maximum-similarity scores and filter settings as a reusable table.
when_to_use_negative:
- The goal is to inspect or re-expand compounds already stored in a Pickaxe mongo
  database run (Figure 4/5 style) rather than produce a standalone filter table
- The filter being applied is the metabolomics, thermodynamics (>50 kJ/mol physiological
  Gibbs free energy), or reaction feasibility filter — the exported table schema here
  is specific to similarity-score-based pruning
- No target compound structures are available, so maximum similarity scores cannot
  be computed and a cutoff/sampling filter cannot be parameterized
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: Pickaxe
  role: Generates the reaction network and applies the similarity cutoff/sampling
    filters whose surviving compounds are exported to the filter table
  repo: https://github.com/tyo-nu/MINE-Database
- name: MINE-Database (minedatabase python package)
  role: Provides the Pickaxe implementation, pickaxe_run_template.py, and output handling;
    results can be written to .tsv files or stored in a specified database
  repo: https://github.com/tyo-nu/MINE-Database
- name: Python
  role: Language in which the fingerprint computation, similarity scoring, and CSV
    export are implemented (Python 3.7 or higher required)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:f4dcc905add0534004d8ca7723c61b195ecbfd66872f04bd18030afeabb934f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyo-nu/MINE-Database
  tools:
  - MINE-Database (minedatabase python package)
  - Pickaxe
  - Python
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# compound-filter-table-export

## Summary

Write the compounds surviving Pickaxe's similarity-based network pruning to a flat table (e.g. filtered_compounds.csv) that carries each compound's maximum Tanimoto similarity score against the target set plus the filter parameters used, so downstream generations and audits can trace exactly which compounds were retained and why.

## When to use

Use when a Pickaxe expansion generation has applied a similarity cutoff or similarity sampling filter and you must persist the retained compounds with per-compound maximum-similarity scores and filter settings as a reusable table.

## When NOT to use

- The goal is to inspect or re-expand compounds already stored in a Pickaxe mongo database run (Figure 4/5 style) rather than produce a standalone filter table
- The filter being applied is the metabolomics, thermodynamics (>50 kJ/mol physiological Gibbs free energy), or reaction feasibility filter — the exported table schema here is specific to similarity-score-based pruning
- No target compound structures are available, so maximum similarity scores cannot be computed and a cutoff/sampling filter cannot be parameterized

## Inputs

- Expansion-set candidate compounds (e.g. from a Pickaxe generation using the JN1224min ruleset)
- Target compound structures with computed molecular fingerprints (e.g. 250 KEGG compounds with 4-9 carbons not in YMDB)
- Per-compound maximum Tanimoto similarity scores against the target set
- Filter parameters: similarity cutoff threshold, or sampling count (e.g. 2000) and power-law scaling exponent (zeroth, third, or sixth)

## Outputs

- filtered_compounds.csv with one row per surviving compound, a maximum-similarity-score column, and the filter parameters used
- Optionally, the same retained compounds inserted into a mongo database (as in the paper's Figure 4 and Figure 5 runs)

## How to apply

After computing the Tanimoto similarity of every candidate compound against all target compounds (e.g. the 250 KEGG targets with 4-9 carbons) and recording the maximum score per compound, apply the chosen pruning variant: the cutoff filter removes compounds whose maximum similarity falls below the specified threshold before the next expansion generation, while the similarity sampling filter builds a weighted distribution over the maximum similarity scores (supporting power-law scalings such as zeroth, third, and sixth powers) and stochastically draws a specified number of compounds, e.g. 2000. Then write the surviving compounds to filtered_compounds.csv with a per-compound maximum-similarity-score column and the filter parameters used (threshold or sampling count and scaling exponent). The paper's Figure 4 workflow uses the cutoff variant against a YMDB-derived target set and stores results in a mongo database, while the Figure 5 sampling workflow samples 2000 compounds; the flat CSV export is the lightweight alternative when no MongoDB is configured. Keep the score column aligned one-to-one with retained compounds so the table can be diffed against the pre-filter expansion set.

## Related tools

- **Pickaxe** (Generates the reaction network and applies the similarity cutoff/sampling filters whose surviving compounds are exported to the filter table) — https://github.com/tyo-nu/MINE-Database
- **MINE-Database (minedatabase python package)** (Provides the Pickaxe implementation, pickaxe_run_template.py, and output handling; results can be written to .tsv files or stored in a specified database) — https://github.com/tyo-nu/MINE-Database
- **Python** (Language in which the fingerprint computation, similarity scoring, and CSV export are implemented (Python 3.7 or higher required))

## Examples

```
python pickaxe.py -C ./data/metacyc_generalized_rules.tsv -r ./data/metacyc_coreactants.tsv -g 1 -c ../example_data/iML1515_ecoli_GEM.csv
```

## Evaluation signals

- filtered_compounds.csv exists after the run and contains a maximum-similarity score for every retained compound
- The sampling filter returns exactly the requested number of compounds (e.g. 2000, as in the paper's zeroth/third/sixth power runs)
- Cutoff-mode invariant: no compound in the table has a maximum similarity score below the specified threshold
- Row count of the table equals pre-filter compound count minus cutoff removals (cutoff mode) or the sampled draw size (sampling mode)
- The recorded filter parameters in the table reproduce the run when re-applied to the same expansion set

## Limitations

- The similarity sampling filter is stochastic, so repeated runs with the same parameters will draw different compound subsets unless seeded
- Compounds' similarity depends entirely on the chosen target set (e.g. the paper's 250 randomly selected KEGG compounds with 4-9 carbons not in YMDB); a different target set changes which compounds survive
- The paper stores large-scale results (Figures 4-6) in mongo databases; a flat CSV export is only described for the filtered compound list and may not scale to multi-million-compound expansions (KEGG one-generation expansion produced 9,562,940 compounds)
- Pickaxe software version number is not stated in the paper (only Python 3.7+ requirement), so exact output schema may vary between package versions

## Evidence

- [results] Similarity cutoff filter: remove compounds whose maximum similarity score to targets is below a threshold before each expansion: [evidence span withheld — anchor status: not_found]
- [results] Similarity sampling filter samples a specified number of compounds from a weighted distribution of similarity scores: "The similarity sampling filter utilizes a weighted distribution of the similarity scores calculated with a set of target compounds to sample a specified number of random compounds to be expanded."
- [results] Three networks sampled 2000 compounds from similarity-score distributions scaled to the zeroth, third, and sixth powers: "Three networks were generated by specifying the sampling filter to select 2000 compounds from distributions generated from scaling the similarity score distribution to the zeroth, third, and sixth"
- [results] Target compounds were 250 randomly chosen KEGG compounds with 4-9 carbons absent from YMDB: "A set of 250 target compounds was selected by randomly choosing compounds from the KEGG database containing between four and nine carbons that are not also within YMDB."
- [readme] Pickaxe run results can be stored in a database or written to flat files: [evidence span withheld — anchor status: external_doc]
