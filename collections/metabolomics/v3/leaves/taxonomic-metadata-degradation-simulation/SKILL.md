---
name: taxonomic-metadata-degradation-simulation
description: Use when you need to validate or re-tune taxonomically informed re-ranking
  scores (e.g., taxo_scorer weights) under degraded or incomplete taxonomic metadata,
  testing robustness of the scoring strategy.
when_to_use_negative:
- Your annotations already have complete, reliably resolved taxonomy and you only
  need to apply pre-computed scores (0.81/1.62/2.55) rather than re-optimize them
- Your metabolites are mostly ubiquitous rather than specialized, since taxonomically
  informed scoring mostly benefits specialized metabolites
- Your candidate structures come from a tool other than ISDB-DNP without re-validating
  that the optimized weights transfer to that tool's output distributions
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0621
tools:
- name: taxo_scorer
  role: Computes the taxonomically informed re-ranking scores (2_taxo_scorer.Rmd)
    whose family/genus/species weights are being optimized
  repo: https://github.com/oolonek/taxo_scorer
- name: ParBayesianOptimization
  role: Parallelizable Bayesian optimization package used to tune the three taxo_scorer
    parameters in parallel mode over each degraded block
  repo: https://github.com/AnotherSamWilson/ParBayesianOptimization
- name: gnfinder
  role: Extracts taxonomic information from free-text biological source fields prior
    to scoring, providing the taxa levels subject to degradation
  repo: https://github.com/gnames/gnfinder
- name: ISDB-DNP
  role: Annotation tool whose outputs, backed by the Dictionary of Natural Products,
    form the candidate annotations whose taxonomy is degraded and re-scored
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:97d6fb450f7ded0f6a7549ef3531249a7707074ebda2c8e5379e31c070a5825a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oolonek/taxo_scorer
  tools:
  - ISDB-DNP
  - MS-Finder
  - ParBayesianOptimization
  - gnfinder
  - taxo_scorer
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# taxonomic-metadata-degradation-simulation

## Summary

Simulates incomplete taxonomic knowledge by randomly degrading the taxonomic metadata of ISDB-DNP annotations into four independent blocks, then uses Bayesian optimization to recover the optimal taxonomically informed scoring weights at family, genus, and species taxa levels.

## When to use

Use when you need to validate or re-tune taxonomically informed re-ranking scores (e.g., taxo_scorer weights) under degraded or incomplete taxonomic metadata, testing robustness of the scoring strategy.

## When NOT to use

- Your annotations already have complete, reliably resolved taxonomy and you only need to apply pre-computed scores (0.81/1.62/2.55) rather than re-optimize them
- Your metabolites are mostly ubiquitous rather than specialized, since taxonomically informed scoring mostly benefits specialized metabolites
- Your candidate structures come from a tool other than ISDB-DNP without re-validating that the optimized weights transfer to that tool's output distributions

## Inputs

- ISDB-DNP annotation outputs (from OSF deposit osf.io/bvs6x)
- 2,107-entry benchmarking dataset of identified molecules
- Taxonomic metadata of annotations (species/genus/family levels, resolved against Catalogue of Life)

## Outputs

- Four degradation blocks of annotations with partially removed taxonomic metadata
- Block-wise optimal scoring parameter combinations from Bayesian optimization
- Averaged optimal scores written to optimal_scores.csv (expected: 0.81, 1.62, 2.55 for family/genus/species)

## How to apply

Retrieve the ISDB-DNP annotation outputs and the 2,107-entry benchmarking dataset from the OSF deposit (osf.io/bvs6x). Randomly degrade the taxonomic metadata of the annotations into four independent blocks to simulate incomplete taxonomic knowledge. For each block, compute taxonomically informed re-ranking scores with the taxo_scorer workflow (2_taxo_scorer.Rmd), weighting similarity contributions at the family, genus, and species taxa levels, and tune the three score parameters with the ParBayesianOptimization package in parallel mode. Average the block-wise optimal parameter combinations across the four degradation blocks and compare them against the reported optima of 0.81 (family), 1.62 (genus), and 2.55 (species). The averaging over blocks guards against overfitting the optimization to one particular pattern of taxonomic incompleteness.

## Related tools

- **taxo_scorer** (Computes the taxonomically informed re-ranking scores (2_taxo_scorer.Rmd) whose family/genus/species weights are being optimized) — https://github.com/oolonek/taxo_scorer
- **ParBayesianOptimization** (Parallelizable Bayesian optimization package used to tune the three taxo_scorer parameters in parallel mode over each degraded block) — https://github.com/AnotherSamWilson/ParBayesianOptimization
- **gnfinder** (Extracts taxonomic information from free-text biological source fields prior to scoring, providing the taxa levels subject to degradation) — https://github.com/gnames/gnfinder
- **ISDB-DNP** (Annotation tool whose outputs, backed by the Dictionary of Natural Products, form the candidate annotations whose taxonomy is degraded and re-scored)

## Examples

```
Rscript -e "library(ParBayesianOptimization); opt <- bayesOpt(scoringFunction, bounds = list(w_family=c(0,2), w_genus=c(0,4), w_species=c(0,4)), initPoints=10, iters.n=5)"
```

## Evaluation signals

- Averaged optimized weights across the four degradation blocks reproduce the reported optima of 0.81 (family), 1.62 (genus), and 2.55 (species)
- Optimized scores are inversely proportional to taxa hierarchical position (family < genus < species)
- Each degradation block's Bayesian optimization converges (ParBayesianOptimization stopping criteria met) and its optimum is recorded before averaging
- Resulting optimal_scores.csv contains exactly three parameters matching the family/genus/species order
- Re-ranking with the recovered scores improves recall and precision relative to unweighted ISDB-DNP rankings on the benchmarking dataset

## Limitations

- The optimized score combination is heavily dependent on the training sets and should not be taken as absolute optimal values
- Taxonomically informed scoring mostly benefits specialized metabolites, not ubiquitous molecules
- The ISDB-DNP tool is favored in benchmarking because all compounds of the benchmarking dataset are present in the DNP

## Evidence

- [discussion] Optimal taxo-informed scores for ISDB-DNP were 0.81 (family), 1.62 (genus) and 2.55 (species): "the optimal combination of scores was 0.81, 1.62 and 2.55 (family, genus and species taxa level, respectively)"
- [discussion] Bayesian optimization package: "A parallelizable Bayesian optimization algorithm (https://github.com/AnotherSamWilson/ParBayesianOptimization) was then used"
- [discussion] Taxonomically informed scoring and re-ranking: "2) taxonomically informed scoring and re-ranking (2_taxo_scorer.Rmd)"
- [discussion] Caution on optimized values: "Such results should be taken with caution, and not as absolute optimal values, as such optimization process are heavily dependent on the training sets"
- [readme] Bayesian optimization is a hyperparameter tuning process guided by prior evaluations: "This is precisely the idea
behind Bayesian Optimization, in which our prior response distribution
is iteratively updated based on our best guess of where the best
parameters are."
