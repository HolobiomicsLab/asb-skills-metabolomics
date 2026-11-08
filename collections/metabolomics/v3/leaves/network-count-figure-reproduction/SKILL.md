---
name: network-count-figure-reproduction
description: Use when validating a rule-based reaction network expansion against published
  counts — e.g. reproducing Pickaxe Figure 4 trends for the YMDB (Version 2.0) expansion
  filtered by similarity cutoffs (0.4/0.5/0.8) or by similarity sampling of 2000 compounds.
when_to_use_negative:
- The goal is thermodynamic or metabolomics filtering of an expansion (Figures 5–6
  use eQuilibrator thermodynamics or E. coli metabolomics peaks with m/z 150–300 Da,
  not the similarity cutoff/sampling design of Figure 4)
- Input is already a generated network and only downstream annotation or peak assignment
  is needed rather than regeneration and count validation
- MongoDB storage is unavailable and .tsv outputs are desired instead — the Figure
  4 workflow specifically stores results in a mongo database
tools:
- name: Pickaxe
  role: Applies the top 500 JN1224min reaction rules iteratively to filtered YMDB
    compounds for one generation to produce the network to be filtered and counted
  repo: https://github.com/tyo-nu/MINE-Database
- name: MINE-Database (minedatabase python package)
  role: Python package providing Pickaxe and the similarity cutoff/sampling filters
    used in the fig4_YMDB_cutoff generation code
  repo: https://github.com/tyo-nu/MINE-Database
- name: MongoDB (mongo database)
  role: Stores the expanded compounds and reactions from each Figure 4 generation
    run for later count analysis
- name: pickaxe_paper repository
  role: Contains fig4_YMDB_cutoff generation code and fig4_gen/analysis/tani_both_analysis.ipynb
    used to tabulate and validate the counts
  repo: https://github.com/tyo-nu/pickaxe_paper
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:404ebd1706075bf544982ece98f412c9b0246a3245786af017b7f7f2049a6aba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyo-nu/MINE-Database
  tools:
  - MINE-Database (minedatabase python package)
  - MongoDB (mongo database)
  - Pickaxe
  - pickaxe_paper repository
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# network-count-figure-reproduction

## Summary

Reproduces the target and network compound counts reported in Pickaxe's Figure 4 by regenerating the YMDB reaction network under similarity cutoff and similarity sampling filters, then comparing per-cutoff and per-weighting counts against the paper's reported values.

## When to use

Use when validating a rule-based reaction network expansion against published counts — e.g. reproducing Pickaxe Figure 4 trends for the YMDB (Version 2.0) expansion filtered by similarity cutoffs (0.4/0.5/0.8) or by similarity sampling of 2000 compounds.

## When NOT to use

- The goal is thermodynamic or metabolomics filtering of an expansion (Figures 5–6 use eQuilibrator thermodynamics or E. coli metabolomics peaks with m/z 150–300 Da, not the similarity cutoff/sampling design of Figure 4)
- Input is already a generated network and only downstream annotation or peak assignment is needed rather than regeneration and count validation
- MongoDB storage is unavailable and .tsv outputs are desired instead — the Figure 4 workflow specifically stores results in a mongo database

## Inputs

- YMDB (Version 2.0) compound database filtered to molecular weight < 600 g/mol
- KEGG (Release 99.0) compound database filtered to molecular weight < 600 g/mol
- Set of 250 KEGG target compounds with 4–9 carbons not present in YMDB
- Top 500 JN1224min ruleset (94% MetaCyc mapped)
- MongoDB instance (local or remote) for storing expansion results

## Outputs

- Networks of novel compounds and reactions per similarity cutoff (0.4, 0.5, 0.8) and per weighting (powers 0, 3, 6) stored in MongoDB
- fig4_target_network_counts.csv — table of target and network compound counts per cutoff and per weighting
- Comparison against Figure 4 reference values, e.g. unfiltered YMDB expansion of 575,262 compounds and 892,343 reactions

## How to apply

Load the YMDB (Version 2.0) and KEGG (Release 99.0) compound databases and filter each to compounds with molecular weight below 600 g/mol; randomly select 250 target compounds from KEGG containing four to nine carbons that are not in YMDB. Expand the filtered YMDB starting compounds (1,035 compounds) for one generation with the top 500 JN1224min rules (94% MetaCyc mapped) using Pickaxe via the minedatabase package, storing results in MongoDB as done in fig4_YMDB_cutoff/. For cutoff networks, remove compounds whose maximum similarity score to the 250 targets is below each threshold (0.4, 0.5, 0.8) before each expansion; for sampling networks, select 2000 compounds per generation from distributions created by scaling the similarity score distribution to the zeroth, third, and sixth powers. Run fig4_gen/analysis/tani_both_analysis.ipynb to tabulate target and network compound counts per cutoff and per weighting and write fig4_target_network_counts.csv. Rationale: the cutoff filter grows exponentially with threshold, while the sampling filter reacts a fixed number of compounds per generation, preventing exponential network growth; higher-power weighting biases the sampled compounds toward higher similarity to targets and yields the most target compounds.

## Related tools

- **Pickaxe** (Applies the top 500 JN1224min reaction rules iteratively to filtered YMDB compounds for one generation to produce the network to be filtered and counted) — https://github.com/tyo-nu/MINE-Database
- **MINE-Database (minedatabase python package)** (Python package providing Pickaxe and the similarity cutoff/sampling filters used in the fig4_YMDB_cutoff generation code) — https://github.com/tyo-nu/MINE-Database
- **MongoDB (mongo database)** (Stores the expanded compounds and reactions from each Figure 4 generation run for later count analysis)
- **pickaxe_paper repository** (Contains fig4_YMDB_cutoff generation code and fig4_gen/analysis/tani_both_analysis.ipynb used to tabulate and validate the counts) — https://github.com/tyo-nu/pickaxe_paper

## Examples

```
jupyter nbconvert --to notebook --execute fig4_gen/analysis/tani_both_analysis.ipynb
```

## Evaluation signals

- The unfiltered YMDB expansion must reproduce 575,262 compounds and 892,343 reactions from 1,035 starting compounds after one generation with the top 500 JN1224min rules
- Cutoff-filtered network sizes must decrease as the similarity threshold rises (0.4 → 0.5 → 0.8), reflecting removal of compounds below each threshold before each expansion
- Each similarity-sampling network must contain a fixed ~2000 sampled compounds per generation regardless of weighting, confirming no exponential growth
- Higher-power weightings (sixth vs third vs zeroth) must yield the most target compounds and a sampled set biased toward higher similarity to the 250 targets
- fig4_target_network_counts.csv must contain one count row per cutoff and per weighting matching the trends and values reported in Figure 4

## Limitations

- Unlike the cutoff filter or no filter, the sampling filter reacts only a fixed number of compounds per generation, so sampling networks cannot be compared to cutoff networks on raw size alone
- Expansion results must be stored in a MongoDB database; without a local or remote mongo instance the fig4_YMDB_cutoff generation code cannot run as specified
- No software version numbers were stated for the minedatabase package or Pickaxe in the reproduced materials, so exact reproducibility across versions is not guaranteed
- Network expansion is computationally heavy (the comparable KEGG expansion generated 12 million new compounds in approximately nine hours)

## Evidence

- [results] YMDB expansion (1,035 starting compounds) generated 575,262 compounds and 892,343 reactions: [evidence span withheld — anchor status: not_found]
- [results] Similarity sampling filter design: "Three networks were generated by specifying the sampling filter to select 2000 compounds from distributions generated from scaling the similarity score distribution to the zeroth, third, and sixth"
- [results] Target compound selection: "A set of 250 target compounds was selected by randomly choosing compounds from the KEGG database containing between four and nine carbons that are not also within YMDB."
- [results] Cutoff filter semantics: [evidence span withheld — anchor status: not_found]
- [results] Database filtering threshold: "Each database was filtered to only contain compounds whose molecular weights are less than 600 g/mol."
- [readme] Figure 4 code locations: "Generation code is found in `fig4_YMDB_cutoff/` - Results are stored in a mongo database. - Analysis code is found in `fig4_gen/analysis/tani_both_analysis.ipynb`"
