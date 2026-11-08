---
name: molecular-fingerprint-generation
description: Use when expanding a reaction network with Pickaxe and compounds must
  be pruned or sampled by chemical similarity to a set of target molecules before
  the next expansion generation.
when_to_use_negative:
- The goal is to prune compounds by metabolomic m/z matching rather than structural
  similarity (use the metabolomics filter instead)
- No target compound set is defined, so no similarity-to-targets criterion exists
- Reactions should be pruned by feasibility (Gibbs free energy > 50 kJ/mol) rather
  than compounds pruned by structural similarity
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3318
tools:
- name: Pickaxe
  role: Applies the fingerprint-based similarity cutoff and sampling filters on-the-fly
    during network expansion
  repo: https://github.com/tyo-nu/MINE-Database
- name: MINE-Database (minedatabase python package)
  role: Python package containing Pickaxe; installed via pip or conda to run the similarity-filtered
    expansions
  repo: https://github.com/tyo-nu/MINE-Database
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:6ad76c770820f2e0352ac758297e7f8dc28cef98f587057baa2e5b2fcd4ae9e6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyo-nu/MINE-Database
  tools:
  - MINE-Database (minedatabase python package)
  - Pickaxe
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# molecular-fingerprint-generation

## Summary

Generates molecular fingerprints for target and candidate compounds and computes maximum Tanimoto similarity scores to support Pickaxe's on-the-fly chemical similarity filters (cutoff and weighted sampling) during network expansion.

## When to use

Use when expanding a reaction network with Pickaxe and compounds must be pruned or sampled by chemical similarity to a set of target molecules before the next expansion generation.

## When NOT to use

- The goal is to prune compounds by metabolomic m/z matching rather than structural similarity (use the metabolomics filter instead)
- No target compound set is defined, so no similarity-to-targets criterion exists
- Reactions should be pruned by feasibility (Gibbs free energy > 50 kJ/mol) rather than compounds pruned by structural similarity

## Inputs

- Target compound structures (e.g., 250 target compounds selected from KEGG with 4-9 carbons not in YMDB)
- Candidate compounds from the current Pickaxe expansion generation
- Filter parameters: similarity threshold (cutoff filter), number of compounds to sample and similarity-score power scaling (sampling filter)

## Outputs

- filtered_compounds.csv with a per-compound maximum-similarity-score column and the filter parameters used
- A sampled compound set (e.g., 2000 compounds) passed to the next expansion generation

## How to apply

Load the target compound structures and compute a molecular fingerprint for each target. For every candidate compound in the Pickaxe expansion set, compute the Tanimoto similarity against all targets and record the maximum score per compound. Two filter variants consume these scores: the similarity cutoff filter removes any compound whose maximum similarity score falls below a specified threshold before the next expansion generation, and the similarity sampling filter builds a weighted distribution over the maximum similarity scores — supporting power-law scalings such as zeroth, third, and sixth powers — and stochastically draws a specified number of compounds (e.g., 2000) to be expanded. Write surviving compounds to filtered_compounds.csv with a per-compound maximum-similarity-score column and the filter parameters used. The rationale is that exponential network growth (e.g., 9,562,940 compounds from one generation of 500 JN1224min rules on KEGG) makes exhaustive expansion intractable, so similarity to targets constrains which compounds continue forward.

## Related tools

- **Pickaxe** (Applies the fingerprint-based similarity cutoff and sampling filters on-the-fly during network expansion) — https://github.com/tyo-nu/MINE-Database
- **MINE-Database (minedatabase python package)** (Python package containing Pickaxe; installed via pip or conda to run the similarity-filtered expansions) — https://github.com/tyo-nu/MINE-Database

## Examples

```
python -m minedatabase.pickaxe -C ./data/metacyc_generalized_rules.tsv -r ./data/metacyc_coreactants.tsv -g 1 -c ../example_data/iML1515_ecoli_GEM.csv
```

## Evaluation signals

- filtered_compounds.csv exists and contains a maximum-similarity score for every retained compound
- The similarity cutoff filter removes exactly those compounds whose maximum similarity score falls below the specified threshold before the next generation
- The similarity sampling filter returns exactly the requested number of compounds (e.g., 2000)
- Sampled networks reproduce the expected behavior of zeroth-, third-, and sixth-power scalings of the similarity-score distribution (zeroth power approximates uniform sampling)
- Compounds surviving the filter all have valid fingerprints comparable against the target set

## Limitations

- The sampling filter is stochastic: drawing a specified number (e.g., 2000) of compounds from the weighted distribution means retained compounds are random samples, not a deterministic best-similarity set
- The cutoff filter discards compounds below the threshold permanently at each generation, so an overly strict threshold can prune compounds that would have led to targets in later generations
- No Pickaxe software version number is stated in the paper (only Python 3.7 or higher), so exact filter behavior may vary across package versions

## Evidence

- [results] Similarity cutoff filter: remove compounds whose maximum similarity score to targets is below a threshold before each expansion: [evidence span withheld — anchor status: not_found]
- [results] Similarity sampling filter: sample a specified number of compounds from a weighted distribution of similarity scores: "The similarity sampling filter utilizes a weighted distribution of the similarity scores calculated with a set of target compounds to sample a specified number of random compounds to be expanded."
- [results] Sample 2000 compounds from similarity-score distributions scaled to the zeroth, third, and sixth powers: "Three networks were generated by specifying the sampling filter to select 2000 compounds from distributions generated from scaling the similarity score distribution to the zeroth, third, and sixth"
- [abstract] Chemical similarity to target molecules is one of Pickaxe's on-the-fly filters: "The filters include chemical similarity to target molecules, metabolomics, thermodynamics, and reaction feasibility filters."
