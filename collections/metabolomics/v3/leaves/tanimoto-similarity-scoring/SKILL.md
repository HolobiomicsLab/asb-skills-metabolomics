---
name: tanimoto-similarity-scoring
description: Use when a Pickaxe network expansion is generating too many compounds
  to expand further and you have a concrete set of target compounds (e.g. 250 KEGG
  targets with 4–9 carbons) to bias the network toward.
when_to_use_negative:
- The goal is to annotate unannotated metabolomics peaks — use the metabolomics filter,
  which keeps only compounds whose m/z value matches one of the peaks, rather than
  similarity-based pruning.
- Compounds should be retained or removed based on thermodynamic feasibility (physiological
  Gibbs free energy > 50 kJ/mol) rather than structural similarity to targets.
- No target compound set exists to define the desired chemical neighborhood of the
  network.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_0091
tools:
- name: Pickaxe
  role: Applies the similarity cutoff and similarity sampling filters on-the-fly during
    iterative rule-based network expansion
  repo: https://github.com/tyo-nu/MINE-Database
- name: MINE-Database (minedatabase python package)
  role: Provides the Python package (pip install minedatabase) through which Pickaxe
    and its filters are run and results are stored
  repo: https://pypi.org/project/minedatabase/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:2014854f5b1471989d4cae90412dba7240c09e18515bb129fadcf8946845e235
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyo-nu/MINE-Database
  tools:
  - MINE-Database (minedatabase python package)
  - Pickaxe
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# tanimoto-similarity-scoring

## Summary

Scores candidate compounds generated during Pickaxe network expansion by their maximum Tanimoto chemical similarity to a user-defined target compound set, then prunes or samples the network using either a similarity cutoff filter or a similarity sampling filter.

## When to use

Use when a Pickaxe network expansion is generating too many compounds to expand further and you have a concrete set of target compounds (e.g. 250 KEGG targets with 4–9 carbons) to bias the network toward.

## When NOT to use

- The goal is to annotate unannotated metabolomics peaks — use the metabolomics filter, which keeps only compounds whose m/z value matches one of the peaks, rather than similarity-based pruning.
- Compounds should be retained or removed based on thermodynamic feasibility (physiological Gibbs free energy > 50 kJ/mol) rather than structural similarity to targets.
- No target compound set exists to define the desired chemical neighborhood of the network.

## Inputs

- Target compound structures (e.g. 250 compounds selected from KEGG containing between four and nine carbons not in YMDB)
- Candidate compound set from a Pickaxe expansion generation
- Filter parameters (similarity cutoff threshold, or number of compounds to sample e.g. 2000 and distribution power scaling e.g. zeroth/third/sixth)

## Outputs

- filtered_compounds.csv with a per-compound maximum-similarity-score column and the filter parameters used
- Pruned or sampled compound set passed to the next expansion generation (optionally stored in a mongo database)

## How to apply

Load the target compound structures and compute molecular fingerprints; then for every candidate compound in the expansion set compute Tanimoto similarity against all targets and record the maximum score per compound. Apply one of two filter variants on-the-fly between expansion generations: the similarity cutoff filter removes any compound whose maximum similarity score falls below a specified threshold before the next generation, while the similarity sampling filter builds a weighted distribution over the maximum similarity scores and stochastically draws a specified number of compounds (e.g. 2000) to expand. For sampling, the similarity score distribution can be scaled to a power — the paper generated three networks using zeroth, third, and sixth power scalings, where higher powers weight the sample more heavily toward compounds most similar to the targets. Write surviving compounds to filtered_compounds.csv with a per-compound maximum-similarity-score column and the filter parameters used, so filter provenance is retained.

## Related tools

- **Pickaxe** (Applies the similarity cutoff and similarity sampling filters on-the-fly during iterative rule-based network expansion) — https://github.com/tyo-nu/MINE-Database
- **MINE-Database (minedatabase python package)** (Provides the Python package (pip install minedatabase) through which Pickaxe and its filters are run and results are stored) — https://pypi.org/project/minedatabase/

## Evaluation signals

- filtered_compounds.csv exists and contains a maximum-similarity score for every retained compound.
- The similarity sampling filter returns exactly the requested number of compounds (e.g. 2000).
- With the cutoff variant, no compound in the surviving set has a maximum similarity score below the specified threshold.
- Sampling runs at power 0 (uniform weighting over the distribution) are distinguishable from runs at third and sixth powers by their score distributions, with higher powers concentrating on the most target-similar compounds.
- Network size after filtering is substantially smaller than the unfiltered expansion (e.g. the paper's sampling runs selected 2000 compounds from full KEGG-scale expansions of ~9.5 million compounds).

## Limitations

- Similarity sampling is stochastic, so repeated runs with the same parameters draw different compound sets; the paper's Figure 5 runs are stored in mongo databases for reproducibility.
- The paper reports runtime performance benchmarks only as Additional file 1, whose content is not included, so scaling behavior of similarity computation against large expansions must be verified empirically.
- Only the maximum similarity score per compound (against the whole target set) is used; no per-target breakdown is retained in the filtering step.

## Evidence

- [results] Similarity cutoff filter removes low-similarity compounds: [evidence span withheld — anchor status: not_found]
- [results] Similarity sampling filter samples from a weighted distribution: "The similarity sampling filter utilizes a weighted distribution of the similarity scores calculated with a set of target compounds to sample a specified number of random compounds to be expanded."
- [results] Sampling runs use 2000 compounds and power-scaled distributions: "Three networks were generated by specifying the sampling filter to select 2000 compounds from distributions generated from scaling the similarity score distribution to the zeroth, third, and sixth"
- [abstract] Chemical similarity is one of four on-the-fly filter types: "The filters include chemical similarity to target molecules, metabolomics, thermodynamics, and reaction feasibility filters."
- [readme] Pickaxe applies rules to user-specified compounds (context for on-the-fly pruning): [evidence span withheld — anchor status: external_doc]
