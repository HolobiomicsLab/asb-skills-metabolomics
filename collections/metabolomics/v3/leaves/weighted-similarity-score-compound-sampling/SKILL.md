---
name: weighted-similarity-score-compound-sampling
description: Use when expanding a compound database with Pickaxe rules and unfiltered
  or similarity-cutoff expansion grows exponentially (e.g. YMDB one-generation expansion
  gives 575,262 compounds), but you need a fixed-size, target-biased network.
when_to_use_negative:
- When the goal is an exhaustive network of all compounds above a similarity threshold
  — use the similarity cutoff filter (0.4, 0.5, 0.8) or no filter instead, accepting
  exponential growth
- When no target compound set exists, since the filter weights a distribution of similarity
  scores to targets
- When filtering is needed on other criteria (m/z peaks, thermodynamics >50 kJ/mol,
  carbon count <12) — those require the metabolomics, thermodynamics, or atomic composition
  filters
tools:
- name: Pickaxe / MINE-Database (minedatabase python package)
  role: Runs the rule-based expansion and applies the similarity sampling filter to
    draw the fixed-size compound set each generation
  repo: https://github.com/tyo-nu/MINE-Database
- name: MongoDB
  role: Stores the sampled networks and expanded compounds/reactions generated in
    fig4_YMDB_cutoff
  repo: https://www.mongodb.com/try/download/community
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:2dda0eb9f91439e2903512ab708ab3e0ac7ca9761a7d6dbf4ce6ee479a83beb2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyo-nu/MINE-Database
  tools:
  - MongoDB
  - Pickaxe
  - Pickaxe / MINE-Database (minedatabase python package)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# weighted-similarity-score-compound-sampling

## Summary

A Pickaxe filter that bounds reaction-network growth by sampling a fixed number of compounds per expansion (e.g. 2000) from a similarity-score distribution raised to a power (zeroth, third, or sixth) against a target set, rather than keeping all compounds above a similarity cutoff.

## When to use

Use when expanding a compound database with Pickaxe rules and unfiltered or similarity-cutoff expansion grows exponentially (e.g. YMDB one-generation expansion gives 575,262 compounds), but you need a fixed-size, target-biased network.

## When NOT to use

- When the goal is an exhaustive network of all compounds above a similarity threshold — use the similarity cutoff filter (0.4, 0.5, 0.8) or no filter instead, accepting exponential growth
- When no target compound set exists, since the filter weights a distribution of similarity scores to targets
- When filtering is needed on other criteria (m/z peaks, thermodynamics >50 kJ/mol, carbon count <12) — those require the metabolomics, thermodynamics, or atomic composition filters

## Inputs

- Filtered compound database (e.g. YMDB 2.0 or KEGG Release 99.0, compounds <600 g/mol) as Pickaxe starting compounds
- Set of target compounds (250 KEGG compounds with 4-9 carbons not present in YMDB)
- JN1224min reaction ruleset (top 500 rules, 94% MetaCyc mapped) with coreactants
- Similarity score distribution of generated compounds against the target set
- Power exponent for distribution weighting (zeroth, third, or sixth)

## Outputs

- Reaction network of exactly 2000 sampled compounds per generation, stored in a MongoDB database
- Target and network compound counts per weighting, written to fig4_target_network_counts.csv

## How to apply

Expand the filtered starting compounds (e.g. YMDB 2.0 compounds <600 g/mol) for one generation with the top 500 JN1224min rules (94% MetaCyc mapped) via the minedatabase package, storing results in MongoDB. Compute each generated compound's maximum similarity score to the target set (e.g. 250 KEGG compounds with 4-9 carbons absent from YMDB), build a distribution of those scores, and scale it to a chosen power (zeroth = uniform, third, or sixth) so higher powers bias sampling toward high-similarity compounds. Specify the sampling filter to draw a fixed number of compounds (2000 in the Figure 4 workflow) for expansion in the next generation; unlike the cutoff filter (tested at 0.4, 0.5, 0.8), network size does not grow exponentially because only the sampled compounds react. Compare weightings by tabulating target and network compound counts with fig4_gen/analysis/tani_both_analysis.ipynb and writing fig4_target_network_counts.csv.

## Related tools

- **Pickaxe / MINE-Database (minedatabase python package)** (Runs the rule-based expansion and applies the similarity sampling filter to draw the fixed-size compound set each generation) — https://github.com/tyo-nu/MINE-Database
- **MongoDB** (Stores the sampled networks and expanded compounds/reactions generated in fig4_YMDB_cutoff) — https://www.mongodb.com/try/download/community

## Examples

```
python -m minedatabase.pickaxe -C ./data/metacyc_generalized_rules.tsv -r ./data/metacyc_coreactants.tsv -g 1 -c ../example_data/iML1515_ecoli_GEM.csv
```

## Evaluation signals

- Each sampled network contains exactly the specified number of compounds (2000) per expansion generation
- Network size remains bounded across generations (no exponential growth seen with the cutoff filter or no filter)
- Higher-power weightings (third, sixth) yield more target compounds among the 2000 sampled and a distribution shifted toward higher similarity to targets
- Unfiltered one-generation YMDB reference run reproduces 575,262 compounds and 892,343 reactions from 1,035 starting compounds
- fig4_target_network_counts.csv counts per cutoff and weighting match the trends and values reported in Figure 4

## Limitations

- Sampling is random within the weighted distribution, so exact compound membership is not deterministic across runs
- The sampling filter fixes network size but does not enforce a minimum similarity threshold — low-similarity compounds can still be sampled under a zeroth (uniform) power
- Full paper-accurate reproduction depends on the pickaxe_paper repo layout (fig4_YMDB_cutoff generation code, results in a mongo database) and matching database versions (YMDB 2.0, KEGG Release 99.0)

## Evidence

- [results] sampling filter samples fixed number per generation: "The similarity sampling filter utilizes a weighted distribution of the similarity scores calculated with a set of target compounds to sample a specified number of random compounds to be expanded."
- [results] three weightings, 2000 compounds: "Three networks were generated by specifying the sampling filter to select 2000 compounds from distributions generated from scaling the similarity score distribution to the zeroth, third, and sixth"
- [results] YMDB unfiltered reference counts: [evidence span withheld — anchor status: not_found]
- [readme] Figure 4 code locations and storage: "Generation code is found in `fig4_YMDB_cutoff/`
- Results are stored in a mongo database.
- Analysis code is found in `fig4_gen/analysis/tani_both_analysis.ipynb`"
