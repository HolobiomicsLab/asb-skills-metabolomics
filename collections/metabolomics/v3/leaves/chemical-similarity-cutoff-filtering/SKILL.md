---
name: chemical-similarity-cutoff-filtering
description: Use when running a Pickaxe/MINE-Database reaction-rule expansion that
  explodes in compound counts and you want to bias the network toward chemical space
  near defined target structures via a hard Tanimoto cutoff.
when_to_use_negative:
- The goal is to hold network size constant per generation (e.g., exactly 2000 compounds);
  use the similarity sampling filter instead, which samples from a weighted distribution
  of similarity scores.
- The filtering criterion is metabolomics peaks, thermodynamic Gibbs energy, or reaction
  feasibility rather than structural similarity to targets.
- No target compounds are defined, so no maximum-similarity score can be computed
  for pruning.
edam_topics:
- http://edamontology.org/topic_3331
tools:
- name: Pickaxe
  role: Generates the novel compound/reaction network via reaction rules and applies
    the similarity cutoff filter on-the-fly between generations
  repo: https://github.com/tyo-nu/MINE-Database
- name: MINE-Database (minedatabase python package)
  role: Python package through which Pickaxe is invoked and through which filters
    and database storage are configured
  repo: https://pypi.org/project/minedatabase/2.0.0/
- name: MongoDB
  role: Stores the expansion results against which the cutoff filter is applied per
    generation
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:5ace19069fbd7e8a3b194f1b090abf79b717fb413f3beee567e7f4422b7c894e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyo-nu/MINE-Database
  tools:
  - MINE-Database (minedatabase python package)
  - MongoDB
  - Pickaxe
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3331
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# chemical-similarity-cutoff-filtering

## Summary

A Pickaxe on-the-fly network-pruning method that computes each candidate compound's maximum Tanimoto similarity to a set of target compounds and removes any compound whose maximum score falls below a specified cutoff before the next expansion generation, preventing exponential network growth.

## When to use

Use when running a Pickaxe/MINE-Database reaction-rule expansion that explodes in compound counts and you want to bias the network toward chemical space near defined target structures via a hard Tanimoto cutoff.

## When NOT to use

- The goal is to hold network size constant per generation (e.g., exactly 2000 compounds); use the similarity sampling filter instead, which samples from a weighted distribution of similarity scores.
- The filtering criterion is metabolomics peaks, thermodynamic Gibbs energy, or reaction feasibility rather than structural similarity to targets.
- No target compounds are defined, so no maximum-similarity score can be computed for pruning.

## Inputs

- Target compound structures with computable molecular fingerprints (e.g., 250 KEGG compounds with 4-9 carbons absent from YMDB)
- Pickaxe expansion network compounds stored in a MongoDB database (e.g., YMDB <600 g/mol compounds expanded one generation with top 500 JN1224min rules)
- Similarity cutoff threshold (e.g., 0.4, 0.5, or 0.8)

## Outputs

- filtered_compounds.csv with a per-compound maximum-similarity-score column and the filter parameters used
- A pruned Pickaxe network in MongoDB retaining only compounds with max Tanimoto similarity above the cutoff
- Per-cutoff target and network compound counts (e.g., fig4_target_network_counts.csv)

## How to apply

First load the target compound structures and compute molecular fingerprints for each target (e.g., 250 targets randomly chosen from KEGG Release 99.0 with 4-9 carbons not present in YMDB 2.0). For every candidate compound in the expansion set, compute the Tanimoto similarity against all targets and record the maximum score per compound. Then implement the similarity cutoff filter so any compound whose maximum similarity score is below the specified threshold (the paper demonstrates cutoffs of 0.4, 0.5, and 0.8) is removed from the network before each expansion generation. Surviving compounds are written to filtered_compounds.csv with a per-compound maximum-similarity-score column and the filter parameters used. Unlike the sampling filter, the cutoff filter prunes by score rather than by count, so network size may still grow across generations but only with compounds near the targets.

## Related tools

- **Pickaxe** (Generates the novel compound/reaction network via reaction rules and applies the similarity cutoff filter on-the-fly between generations) — https://github.com/tyo-nu/MINE-Database
- **MINE-Database (minedatabase python package)** (Python package through which Pickaxe is invoked and through which filters and database storage are configured) — https://pypi.org/project/minedatabase/2.0.0/
- **MongoDB** (Stores the expansion results against which the cutoff filter is applied per generation)

## Evaluation signals

- filtered_compounds.csv exists and contains a maximum-similarity score for every retained compound.
- No retained compound has a maximum similarity score below the specified threshold (e.g., all scores >= 0.4 for a 0.4 cutoff).
- The filter is applied before each expansion generation, so the network contains no compound below the cutoff at any generation.
- Filtered network compound counts are substantially lower than the unfiltered expansion (YMDB unfiltered one-generation expansion yields 575,262 compounds and 892,343 reactions).
- Stricter cutoffs (0.4 -> 0.5 -> 0.8) yield monotonically smaller networks biased toward targets, matching the Figure 4 trends.

## Limitations

- The cutoff filter does not bound network size per generation; only the sampling filter reacts a fixed number of compounds per generation so network size does not grow exponentially.
- Thermodynamics results caveats do not apply here, but the general filter workflow depends on MongoDB storage and the pickaxe_paper generation code (fig4_YMDB_cutoff/) for reproducibility.
- Cutoff choice is empirical; the paper only demonstrates thresholds 0.4, 0.5, and 0.8 and does not prescribe an optimal value.

## Evidence

- [results] Similarity cutoff filter: remove compounds whose maximum similarity score to targets is below a threshold before each expansion: [evidence span withheld — anchor status: not_found]
- [abstract] Filter family includes chemical similarity to target molecules: "The filters include chemical similarity to target molecules, metabolomics, thermodynamics, and reaction feasibility filters."
- [results] Target selection protocol for the similarity filters: "A set of 250 target compounds was selected by randomly choosing compounds from the KEGG database containing between four and nine carbons that are not also within YMDB."
- [results] Starting compounds must be pre-filtered before expansion: "Each database was filtered to only contain compounds whose molecular weights are less than 600 g/mol."
- [readme] Figure 4 cutoff-filter generation code location: "- Generation code is found in `fig4_YMDB_cutoff/`"
