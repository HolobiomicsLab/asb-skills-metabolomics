---
name: target-compound-selection-by-carbon-count
description: Use when a Pickaxe-style network expansion needs an external target-compound
  set, and targets must be bounded in size (carbon count) and disjoint from the starting
  metabolome to make similarity-based filtering meaningful.
when_to_use_negative:
- The targets are already defined by the application (e.g. a user-specified product
  list) — random carbon-count-based selection is for benchmarking/filtering studies,
  not directed pathway design
- The starting database and reference database overlap is desired (e.g. annotating
  known metabolites) — the exclusion of YMDB-present compounds would remove exactly
  those candidates
- Target chemistry falls outside the MW < 600 g/mol range or the carbon window is
  inappropriate for the rule set being tested
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: Pickaxe
  role: Network-expansion engine whose similarity cutoff and similarity sampling filters
    consume the selected target-compound set
  repo: https://github.com/tyo-nu/MINE-Database
- name: MINE-Database (minedatabase python package)
  role: Python package providing Pickaxe and filter implementations used to run the
    YMDB expansion against the selected targets
  repo: https://github.com/tyo-nu/MINE-Database
- name: MongoDB
  role: Stores the expanded network compounds/reactions generated from the selected
    target set
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:1298e1797f3d6d76e4b93e359ef3f7c3c6275b7601ac8c043c54596ad5530983
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyo-nu/MINE-Database
  tools:
  - MINE-Database (minedatabase python package)
  - MongoDB
  - Pickaxe
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# target-compound-selection-by-carbon-count

## Summary

Defines a chemically meaningful target set for network-expansion studies by randomly sampling compounds from a reference database (e.g. KEGG Release 99.0) restricted to a carbon-number window (4–9 carbons) and excluding compounds already present in the starting database (e.g. YMDB Version 2.0).

## When to use

Use when a Pickaxe-style network expansion needs an external target-compound set, and targets must be bounded in size (carbon count) and disjoint from the starting metabolome to make similarity-based filtering meaningful.

## When NOT to use

- The targets are already defined by the application (e.g. a user-specified product list) — random carbon-count-based selection is for benchmarking/filtering studies, not directed pathway design
- The starting database and reference database overlap is desired (e.g. annotating known metabolites) — the exclusion of YMDB-present compounds would remove exactly those candidates
- Target chemistry falls outside the MW < 600 g/mol range or the carbon window is inappropriate for the rule set being tested

## Inputs

- KEGG compound database (Release 99.0), filtered to MW < 600 g/mol
- YMDB compound database (Version 2.0), filtered to MW < 600 g/mol
- Carbon-count bounds for target selection (4–9 carbons)
- Target set size (250 randomly selected compounds)

## Outputs

- List of 250 target compounds (4–9 carbons, present in KEGG but absent from YMDB)
- Target list consumed by similarity cutoff (0.4/0.5/0.8) and similarity sampling (2000 compounds, powers 0/3/6) filters during Pickaxe expansion

## How to apply

Load two compound databases — here KEGG (Release 99.0) and YMDB (Version 2.0), both filtered to compounds with molecular weight below 600 g/mol. From the reference database, randomly select compounds containing between four and nine carbons that are not also present in the starting database, yielding a fixed target set (250 compounds in the Figure 4 workflow). This carbon-count window keeps targets in a size range reachable by the chosen reaction rules (top 500 JN1224min rules, 94% MetaCyc mapped), and excluding YMDB members guarantees targets are novel relative to the starting network. The resulting target list then feeds the similarity cutoff filter (thresholds 0.4, 0.5, 0.8) and the similarity sampling filter, which samples 2000 compounds per generation from similarity-score distributions scaled to the zeroth, third, and sixth powers.

## Related tools

- **Pickaxe** (Network-expansion engine whose similarity cutoff and similarity sampling filters consume the selected target-compound set) — https://github.com/tyo-nu/MINE-Database
- **MINE-Database (minedatabase python package)** (Python package providing Pickaxe and filter implementations used to run the YMDB expansion against the selected targets) — https://github.com/tyo-nu/MINE-Database
- **MongoDB** (Stores the expanded network compounds/reactions generated from the selected target set)

## Examples

```
from minedatabase.pickaxe import Pickaxe; px = Pickaxe(coreactants='./data/metacyc_coreactants.tsv', rules='./data/metacyc_generalized_rules.tsv', targets=target_compound_list)
```

## Evaluation signals

- Target set size matches the specified count (250 compounds for the Figure 4 reproduction)
- Every selected target contains between four and nine carbons
- No selected target is present in the YMDB starting-compound set
- Both source databases were filtered to MW < 600 g/mol before selection
- Downstream Figure 4 trends reproduce: unfiltered YMDB expansion yields 575,262 compounds / 892,343 reactions, and higher-power similarity weightings (3rd, 6th) sample compounds closer to targets and yield more target hits than the zeroth power

## Limitations

- Random selection means the target set is stochastic; reproducibility requires fixing the random seed used when drawing the 250 compounds from KEGG
- The 4–9 carbon window and MW < 600 g/mol bounds were chosen for the Pickaxe benchmark; other rule sets or organisms may need different windows
- The carbon-count criterion alone does not control chemical-class diversity of targets beyond molecular size
- Molecular weight and carbon counts depend on the database records; version drift (YMDB 2.0, KEGG 99.0) changes the eligible pool

## Evidence

- [results] Target selection by carbon count and database exclusion: "A set of 250 target compounds was selected by randomly choosing compounds from the KEGG database containing between four and nine carbons that are not also within YMDB."
- [results] Database pre-filtering by molecular weight: "Each database was filtered to only contain compounds whose molecular weights are less than 600 g/mol."
- [results] Rule set context for target-driven expansion: "The top 500 rules (94% of MetaCyc mapped) from the JN1224min ruleset were applied to the starting compounds for one generation."
- [results] Similarity filters consuming the target set: "Three networks were generated by specifying the sampling filter to select 2000 compounds from distributions generated from scaling the similarity score distribution to the zeroth, third, and sixth"
- [readme] Analysis of target hits per weighting: "Generation code is found in `fig4_YMDB_cutoff/`
- Results are stored in a mongo database.
- Analysis code is found in `fig4_gen/analysis/tani_both_analysis.ipynb`"
