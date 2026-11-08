---
name: weighted-distribution-compound-sampling
description: Use when an unfiltered network expansion would generate too many compounds
  to expand further, and you want stochastic, similarity-biased pruning toward a set
  of target compounds rather than a hard cutoff.
when_to_use_negative:
- You need deterministic, threshold-based pruning of all low-similarity compounds
  — use the similarity cutoff filter instead of sampling
- The goal is metabolomics peak annotation rather than target-biased exploration —
  the metabolomics filter (m/z matching) is the appropriate filter
- The network is small enough that no on-the-fly pruning is needed before the next
  generation
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: Pickaxe
  role: Implements the similarity sampling filter within its on-the-fly network pruning
    during rule-based expansion
  repo: https://github.com/tyo-nu/MINE-Database
- name: MINE-Database (minedatabase python package)
  role: Installable Python package containing Pickaxe; used to run expansions with
    the sampling filter and write results to .tsv files or a mongo database
  repo: https://pypi.org/project/minedatabase/
- name: MongoDB
  role: Stores the results of sampled network expansions (e.g. Figure 5 sampling runs)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:8f24640a1d77238d05292d2c4c5f878094b5c30ddefdfe29d8a57166e7fd8046
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyo-nu/MINE-Database
  tools:
  - MINE-Database (minedatabase python package)
  - MongoDB
  - Pickaxe
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# weighted-distribution-compound-sampling

## Summary

Implements Pickaxe's similarity sampling filter, which builds a weighted distribution over each candidate compound's maximum Tanimoto similarity score to a set of target compounds and stochastically samples a fixed number of compounds (e.g. 2000) to carry into the next expansion generation.

## When to use

Use when an unfiltered network expansion would generate too many compounds to expand further, and you want stochastic, similarity-biased pruning toward a set of target compounds rather than a hard cutoff.

## When NOT to use

- You need deterministic, threshold-based pruning of all low-similarity compounds — use the similarity cutoff filter instead of sampling
- The goal is metabolomics peak annotation rather than target-biased exploration — the metabolomics filter (m/z matching) is the appropriate filter
- The network is small enough that no on-the-fly pruning is needed before the next generation

## Inputs

- Target compound structures (e.g. 250 target compounds selected from KEGG with 4-9 carbons not in YMDB)
- Candidate compounds from the current network expansion generation
- Sampling filter parameters: number of compounds to sample (e.g. 2000) and similarity-score power scaling (zeroth, third, or sixth)

## Outputs

- Filtered compound set (e.g. 2000 sampled compounds) passed to the next expansion generation
- filtered_compounds.csv with a per-compound maximum-similarity-score column and the filter parameters used
- Optionally a mongo database of sampled compounds and reactions (as in the paper's Figure 5 runs)

## How to apply

Compute molecular fingerprints for each target compound, then for every candidate compound in the expansion set compute the Tanimoto similarity against all targets and record the maximum score per compound. Build a weighted distribution from these maximum similarity scores, optionally scaling the scores by a power (Pickaxe supports zeroth, third, and sixth powers, where higher powers bias sampling more strongly toward target-like compounds). Stochastically draw the specified number of compounds from this distribution — e.g. 2000 in the paper's YMDB/KEGG target example — to be expanded in the next generation. Unlike the similarity cutoff filter, which deterministically removes compounds below a threshold, sampling preserves some diversity and lets the exponent control exploration vs. exploitation. Write surviving compounds with their per-compound maximum-similarity score and filter parameters for provenance.

## Related tools

- **Pickaxe** (Implements the similarity sampling filter within its on-the-fly network pruning during rule-based expansion) — https://github.com/tyo-nu/MINE-Database
- **MINE-Database (minedatabase python package)** (Installable Python package containing Pickaxe; used to run expansions with the sampling filter and write results to .tsv files or a mongo database) — https://pypi.org/project/minedatabase/
- **MongoDB** (Stores the results of sampled network expansions (e.g. Figure 5 sampling runs))

## Examples

```
python -m minedatabase.pickaxe -C ./data/metacyc_generalized_rules.tsv -r ./data/metacyc_coreactants.tsv -g 1 -c ../example_data/iML1515_ecoli_GEM.csv
```

## Evaluation signals

- The sampling filter returns exactly the requested number of compounds (e.g. 2000)
- Every retained compound has a recorded maximum-similarity score in the output CSV
- Varying the power scaling (zeroth vs. third vs. sixth) produces the expected monotonic shift in the similarity distribution of sampled compounds toward targets
- The output honors the specified filter parameters and per-compound score column schema
- Sampled compounds are a subset of the pre-filter expansion generation (no fabricated compounds)

## Limitations

- The exponent choice (zeroth, third, sixth power) strongly controls the exploration/exploitation tradeoff; there is no single correct setting and results are stochastic
- Sampling removes compounds probabilistically, so low-similarity but potentially interesting compounds can be discarded; cutoff filtering is deterministic but even more aggressive
- Requires a defined set of target compounds; sampling is only meaningful relative to chosen targets

## Evidence

- [results] Similarity sampling filter: sample a specified number of compounds from a weighted distribution of similarity scores: "The similarity sampling filter utilizes a weighted distribution of the similarity scores calculated with a set of target compounds to sample a specified number of random compounds to be expanded."
- [results] Three networks were generated by specifying the sampling filter to select 2000 compounds from distributions generated from scaling the similarity score distribution to the zeroth, third, and sixth: "Three networks were generated by specifying the sampling filter to select 2000 compounds from distributions generated from scaling the similarity score distribution to the zeroth, third, and sixth"
- [results] Similarity cutoff filter: remove compounds whose maximum similarity score to targets is below a threshold before each expansion: [evidence span withheld — anchor status: not_found]
- [abstract] The filters include chemical similarity to target molecules, metabolomics, thermodynamics, and reaction feasibility filters.: "The filters include chemical similarity to target molecules, metabolomics, thermodynamics, and reaction feasibility filters."
- [readme] Analysis code for the sampling example is found in the pickaxe_paper repo: "Analysis code is found in `fig5_sample/C4C8_2k_Conly/analysis/run_production_sampling2k.ipynb`"
